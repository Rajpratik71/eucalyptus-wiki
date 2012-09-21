# EBS/Volumes
## DesignDoc for the Cloud Admin
The Storage Controller (SC) is the component of Eucalyptus that manages EBS volumes. Eucalyptus uses the iSCSI protocol to connect EBS volumes to instances (NCs actually), and uses standard linux commands for configuring and exporting the volumes to instances. Volumes are represented as files on the filesystem of the machine hosting the SC. The SC manages creating, deleting, snapshotting, and exporting volumes in response to both user commands (example: `euca-create-volume` and `euca-create-snapshot`) and system operations (example: `euca-run-instances` of a boot-from-ebs instance).

There are two broad SC configurations: filesystem-backed SC and SAN-backed SC.

For filesystem-backed SCs, the SC machine is both on the control and data path for the volumes and snapshots. This means the machine running the SC component will host the volume and snapshot data, manage the iSCSI connections to the volumes, and service iSCSI block requests as NCs (and vm instances) read and write to the volume. In this configuration it is important to understand and plan-for the I/O requirements your specific deployment will have and ensure that the machine hosting the SC is up to the task. We recommend lots of memory and as many high-speed disks as possible in a reliable and performant configuration (RAID 10,01,5,6 but not just 0 or 1). SSDs would also make an excellent addition to an SC machine due to their very high IOPS capability. For more recommendations on hardware configurations and best-practices, see our Storage:Best Practices section.

### Backing Store (where the volumes and data are)
**Default Backing Store: Posix Filesystem**
The default configuration for the SC uses a standard Posix filesystem as the backing storage for EBS volumes. Each EBS volume is a single file and each snapshot is also a single file. The files for snapshots and volumes are not connected in any way in the filesystem (i.e. they are not symlinks to each other etc). 

* **Which filesystem should you use?** The only requirement is that the filesystem is Posix compliant. We test the SC with ext4 most extensively, but ext3 and xfs work as well (others like btrfs and reiserfs will probably work but have not been tested). We recommend configuring the filesystem with full journaling of both data and metadata rather than just metadata journaling. Full journaling will increase the robustness of the filesystem and minimize chances of data-loss due to machine failure. Eucalyptus does not currently support a shared filesystem for hosting the volumes because the SC has the expectation that the filesystem subtree it uses is under the sole control of the SC local and updates to that subtree made by another SC could cause file conflicts that the SC does not expect. Doing so can cause name conflicts, particularly for snapshots because snapshot names (i.e. snap-abc123) are globally unique in Eucalyptus.
* **Where are the volumes stored on the filesystem?** The SC operates on the filesystem at `$EUCALYPTUS/var/lib/eucalyptus/volumes`, where `$EUCALYPTUS` is the root of the Eucalyptus installation on the SC machine (the default is '/' if installed by packages). This point in the filesystem can be provided by a separate LVM volume and for best performance that LVM volume should be on its own high-performance disks (the more spindles the better for random I/O performance).
* **What is the directory structure the SC uses?**All volumes and snapshots on a Storage Controller are stored in a single flat directory structure at `$EUCALYPTUS/var/lib/eucalyptus/volumes`. 

**SAN Storage (paid subscriptions only)**
The SC can also be configured to interact with a SAN device to host EBS volumes. In this configuration the SC manages metadata for volumes and snapshots and issues commands to the SAN to perform required operations, but the volumes and snapshots themselves are hosted by the SAN and the SAN manages the iSCSI connections to the NCs. In this configuration the SC is not in the data path for EBS volume block traffic and therefore this configuration typically yields much better performance.

### Export Volumes (how the volumes are accessible by instances)
**SAN-backed SCs**
For SAN-backed SCs the SAN exports the volumes directly to NCs using iSCSI. This is very device specific and we will not discuss it further in this section. Eventually we will probably put up an entire section dedicated to SAN-backed SCs as well as the devices we support and how they are used, but that feature is not part of the open-source Eucalyptus (we have to save some good bits for the paying customers right?).

**Filesystem-backed SCs**
SCs using a filesystem as the backend export volumes using standard linux iSCSI servers and tools. Specifically, the SC uses [the linux ISCSI Framework (TGT)](http://stgt.sourceforge.net/) to export the volumes as iSCSI stores to the NCs. The SC creates an LVM volume from the file using a loopback device (more details below) and exports that device as a store using tgtadm.

The basic operations done when creating a volume (example: vol-X, of size 1GB, and assuming /dev/loop0 is open and there are no other volumes, so target-id 1 is unused as well) are:

1. Create a file: $EUCALYPTUS/var/lib/eucalyptus/volumes/vol-X of size 1GB + lvm header size (~4MB): `dd -if /dev/zero -of $EUCALYPTUS/var/lib/eucalyptus/volumes/vol-X count=1 bs=1M`
2. Find unused loopback: `losetup -f`
2. Attach file to a loopback using losetup: `losetup /dev/loop0 $EUCALYPTUS/var/lib/eucalyptus/volumes`
3. Create physical volume from loopback: `pvcreate /dev/loop0`.
4. Create volume group: `vgcreate /dev/loop0 vg-xyza` (the volume group name is a 4 char random hash).
5. Create logical volume: `lvcreate -n lv-jklm -l 100%FREE vg-xyza`
6. Create new iSCSI target: `tgtadm --lld iscsi --op new --mode target --tid 1 -T <some_prefix><SC_NAME>:store0`
7. Create new lun in the target: `tgtadm --lld iscsi --op new --mode logicalunit --tid 1 --lun 1 -b /dev/vg-xyza/lv-jklm`
8. Add eucalyptus CHAP user to target: `tgtadm --lld iscsi --op bind --mode account --tid 1 --user eucalyptus`
9. Bind the target to all interfaces: `tgtadm --lld iscsi --op bind --mode target --tid 1 -I ALL`

### Snapshotting
**Filesystem-backed SCs**
A snapshot operation is a simple file-copy in this case. However, there are some complexities due to the headers added to the file when it becomes an lvm volume. For that reason, we snapshot by copying (using dd) the source file into the destination file after the destination has been created and configured as an lvm volume independently of the source. The SC only copies the final lvm volume devices so that all lvm header and metadata information stays unique to each file.

### Startup and Shutdown 
there is a lot of state in the StorageManager
### Lifecycle of a Request
Create volume
Attach volume
Snapshot
How Cloud Controller (CLC) is involved
### Database (same issue as walrus)

## DesignDoc for the Developers
code is the ultimate truth here