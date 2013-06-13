_for Eucalyptus 3.1, see notes for 3.2 features in progress_

**There have been some changes to how volume attachment works in Eucalyptus 3.3.0 see:** [Volume Attach/Detach Changes in Eucalyptus 3.3.0](Volume-Attach-Detach-in-Eucalyptus-3.3.0)

This page describes how the Storage Controller works and its various backend storage providers for providing EBS support in Eucalyptus. For installation/configuration and admin/install documentation, see the [Eucalyptus documentation](http://www.eucalyptus.com/docs).

The Storage Controller (SC) is the component of Eucalyptus that manages EBS volumes. Eucalyptus uses the iSCSI protocol to connect EBS volumes to instances (NCs actually), and uses standard linux commands for configuring and exporting the volumes to instances. Volumes are represented as files on the filesystem of the machine hosting the SC. The SC manages creating, deleting, snapshotting, and exporting volumes in response to both user commands (example: `euca-create-volume` and `euca-create-snapshot`) and system operations (example: `euca-run-instances` of a boot-from-ebs instance).

There are three broad SC configurations: filesystem-backed SC, direct disk-backed SC, and SAN-backed SC.

For filesystem-backed SCs, the SC machine is both on the control and data path for the volumes and snapshots. This means the machine running the SC component will host the volume and snapshot data, manage the iSCSI connections to the volumes, and service iSCSI block requests as NCs (and vm instances) read and write to the volume. In this configuration it is important to understand and plan-for the I/O requirements your specific deployment will have and ensure that the machine hosting the SC is up to the task. We recommend lots of memory and as many high-speed disks as possible in a reliable and performant configuration (RAID 10,01,5,6 but not just 0 or 1). SSDs would also make an excellent addition to an SC machine due to their very high IOPS capability. For more recommendations on hardware configurations and best-practices, see our Storage:Best Practices section.

### Backing Store (where the volumes and data are)
**Default Backing Store: Posix Filesystem**
The default configuration for the SC uses a standard Posix filesystem as the backing storage for EBS volumes. Each EBS volume is a single file and each snapshot is also a single file. The files for snapshots and volumes are not connected in any way in the filesystem (i.e. they are not symlinks to each other etc). 

For more information on the Overlay Manager see:
* Eucalyptus 3.1.x: [Overlay Storage (3.1.x)](Overlay-Storage-3.1)
* Eucalyptus 3.2.x: [Overlay Storage (3.2.x)](Overlay-Storage-3.2)

* **Which filesystem should you use?** The only requirement is that the filesystem is Posix compliant. We test the SC with ext4 most extensively, but ext3 and xfs work as well (others like btrfs and reiserfs will probably work but have not been tested). We recommend configuring the filesystem with full journaling of both data and metadata rather than just metadata journaling. Full journaling will increase the robustness of the filesystem and minimize chances of data-loss due to machine failure. Eucalyptus does not currently support a shared filesystem for hosting the volumes because the SC has the expectation that the filesystem subtree it uses is under the sole control of the SC local and updates to that subtree made by another SC could cause file conflicts that the SC does not expect. Doing so can cause name conflicts, particularly for snapshots because snapshot names (i.e. snap-abc123) are globally unique in Eucalyptus.

* **Where are the volumes stored on the filesystem?** 
The SC operates on the filesystem at `$EUCALYPTUS/var/lib/eucalyptus/volumes`, where `$EUCALYPTUS` is the root of the Eucalyptus installation on the SC machine (the default is '/' if installed by packages). This point in the filesystem can be provided by a separate LVM volume and for best performance that LVM volume should be on its own high-performance disks (the more spindles the better for random I/O performance).

* **What is the directory structure the SC uses?**
All volumes and snapshots on a Storage Controller are stored in a single flat directory structure at `$EUCALYPTUS/var/lib/eucalyptus/volumes`. 

**DAS Storage (direct attached as block device), open-sourced in Eucalyptus 3.2**
Basically the same operations as Overlay (filesystem), but without files on the filesystem and without loopback devices. Uses LVM operations on a block device/volume group directly. It is carved into pieces for user volumes using LVM logical volumes. Each volume/snapshot in EBS corresponds to an LVM logical volume on the SC.

For more information on how DAS Storage works see [DAS Manager (3.2)](DAS_Manager_3.2)

**SAN Storage (paid subscriptions only)**
The SC can also be configured to interact with a SAN device to host EBS volumes. In this configuration the SC manages metadata for volumes and snapshots and issues commands to the SAN to perform required operations, but the volumes and snapshots themselves are hosted by the SAN and the SAN manages the iSCSI connections to the NCs. In this configuration the SC is not in the data path for EBS volume block traffic and therefore this configuration typically yields much better performance.

For more information on our Netapp support see [Netapp Support (3.2)](Netapp-support-3.2)

For more information on our EMC VNX support see [EMC VNX Support (3.2)](EMC-VNX-Support-3.2)

For more information on our Equallogic support see [Equallogic Support (3.2)](Equallogic_Support_3.2)

### Export Volumes (how the volumes are accessible by instances)
**SAN-backed SCs**
For SAN-backed SCs the SAN exports the volumes directly to NCs using iSCSI. This is very device specific and we will not discuss it further in this section. Eventually we will probably put up an entire section dedicated to SAN-backed SCs as well as the devices we support and how they are used, but that feature is not part of the open-source Eucalyptus (we have to save some good bits for the paying customers right?).

**Filesystem-backed SCs**
SCs using a filesystem as the backend export volumes using standard linux iSCSI servers and tools. Specifically, the SC uses [the linux ISCSI Framework (TGT)](http://stgt.sourceforge.net/) to export the volumes as iSCSI stores to the NCs. The SC creates an LVM volume from the file using a loopback device (more details below) and exports that device as a store using tgtadm.

### Snapshotting
**Filesystem-backed SCs**
A snapshot operation is a simple file-copy in this case. However, there are some complexities due to the headers added to the file when it becomes an lvm volume. For that reason, we snapshot by copying (using dd) the source file into the destination file after the destination has been created and configured as an lvm volume independently of the source. The SC only copies the final lvm volume devices so that all lvm header and metadata information stays unique to each file.

### Startup and Shutdown 
The startup of the SC involves checking the DB and local/SAN resources.

### Lifecycle of a Request
**Create volume**

1.) User issues euca-create-volume -z cluster01 -s 1 

2.) Message sent from eucatools to the Cloud Controller (CLC) 

3.) Message processed by EC2 API components in CLC, parses the message 

4.) CLC creates its own metadata about the volume to be created and gives it a name: _vol-abc0123_ 

5.) CLC sets status of volume to 'creating' and sends a message to Storage Controller in the appropriate zone/cluster/partition to actually create the volume. 

6.) SC checks that permissions are okay, sets up some metadata of its own, and starts the asynchronous volume create operation. Then, immediately replies back to the CLC, which has been waiting for the reply. 

7.) SC creates the volume itself using these steps below for Filesystem- or SAN-backed

_Direct Disk and Filesystem-backed Storage Controller Volume Creation_

In this case the SC itself hosts the data and creates the volume locally (more info in DAS and Overlay sections)

_SAN-backed Storage Controller_

In this case the SC issues the proper commands to the SAN device directly to have the SAN create a LUN of the proper size and setup the proper credentials for connection later.

9.) Concurrently, the CLC is periodically checking the state of all volumes on every SC, so now that the volume is ready, the CLC sees it and sets the volume state to 'available' as viewable by the user via euca-describe-volumes commands.

***

**Attach volume**

1.) User runs euca-attach-volume -d /dev/sdf -i i-123abc vol-abc0123

2.) CLC gets message to attach the volume.

3.) CLC does some checks to make sure this is a valid operation (i.e. volume is not already attached)

4.) CLC finds the appropriate zone/cluster/partition where the volume is located.

5.) CLC sends message to SC telling it to prep the volume for attach and to get the connection information to send to the Node Controller.

6.) SC does attach stuff (see below) and returns the connection information necessary for clients to attach

For Filesystem-backed SC, this exports the volume via iSCSI from the SC host itself

For SAN-backed SC, the SC issues the proper commands to the SAN to configure credentials and expose the proper LUN to the proper NC.

7.) CLC gets response and sends request to Cluster Controller (CC) in proper zone/cluster/partition to do the attach of the volume to the instance.

8.) CC looks up the Node Controller (NC) hosting the instance requested

9.) CC sends the doAttach message to the proper NC including the connection info.

10.) NC uses the connection info to connect its local iSCSI initiator to the iscsi target on the SC

11.) NC has connection to the iSCSI target and has a local hypervisor block-device (i.e. /dev/sdd)

12.) NC exposes that block device to the instance via libvirt and does proper mapping of device to user-desired name (i.e. NC's /dev/sdd becomes the instance's /dev/sdf, as per the original user request).


**Snapshot**
Coming soon!

### Database
The SC uses a database (hosted on the CLC) to track metadata about volumes and snapshots such as their state, size, location, and backend-specific details.

## DesignDoc for the Developers
code is the ultimate truth here

*****

[[category.components]]