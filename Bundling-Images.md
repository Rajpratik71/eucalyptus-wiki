# NOTE: please consult our current image management guide for authoritative content on image bundling: https://www.eucalyptus.com/docs/eucalyptus/3.4/index.html#image-guide/img_task_intro.html

Eucalyptus Systems provides a handful of prebuilt images of various Linux distributions, but it's often useful to create your own. You can use the Euca2ools euca-bundle-vol command to bundle an existing system, either physical or virtual. Since each Linux distribution is different, there is no 100% generalized way to do this, but here are some common problems and solutions.

The euca-bundle-vol command is distributed as part of euca2ools. Make sure euca2ools are installed and working properly on the system you are trying to bundle. You will also need your Eucalyptus cloud credentials sourced into the environment. Please follow the euca2ools directions if you need help.

## Bundling Considerations

### Free space for the bundling operation

You need at least twice the 'used' disk space to bundle an image. Use an NFS mount, an EBS volume, or your instance's ephemeral space (if you're bundling an image running in Eucalyptus) to make sure you have enough room.

### Networking configuration

Eucalyptus images must be configured to obtain their network configuration from DHCP. On Red Hat-based Linux distributions, replace the contents of /etc/sysconfig/network-scripts/ifcfg-eth0 with the following three lines only:

```
DEVICE=eth0
BOOTPROTO=dhcp
ONBOOT=yes
```

Make sure the hardware MAC address is not specified--Eucalyptus provides its own. If you are in a static networking configuration, you will also need to edit /etc/sysconfig/network to remove the hostname and gateway settings.

You must also exclude the DHCP leases directory, if your system is using DHCP already. See below.

Finally, there sometimes exists a udev "rules" file that causes problems with the auto-generated MAC addresses that Eucalyptus VMs use. It is usually /etc/udev/rules.d/70-persistent-net.rules on most Linux systems. If this file exists, delete it before bundling. It will cause problems with the Ethernet device naming when the system is booted with a new MAC address assigned by Eucalyptus.

### Excluded directories:

You can specify directories to be excluded from the bundling operation. Most of the directories are already excluded, including /tmp, /mnt, /proc, etc. The only one you may want to additionally include is /root. You must also exclude /var/lib/dhclient/ (Red Hat-based systems) or /var/lib/dhcp3 (Debian/Ubuntu).

### Kernel, ramdisk, and modules

It's usually easiest to find a kernel and ramdisk combination from the Eucalyptus Systems pre-built images to boot your new image. Using your own kernel and ramdisk can be difficult because each ramdisk is configured specifically for your system's hardware setup, which may not be compatible with Eucalyptus. For Red Hat and CentOS bundled images, use the CentOS kernel/ramdisk, for Ubuntu use our Ubuntu kernel/ramdisk, etc.

The one thing you need to be aware of is that some systems require the kernel modules from the pre-built image to be implanted into your bundled image. This is because the kernel that you're booting with needs to match exactly the modules that are on the filesystem.

The best way to do this is to mount the pre-built root filesystem image, tar up the kernel modules, and then extract them to the image you're planning to bundle. For example:

```
mount -o loop centos.5-3.x86-64.img /mnt
cd /mnt
tar zcvf centos-xen-modules.tar.gz lib/modules/2.6.27.21-0.1-xen/
umount /mnt
```

Copy the resulting tar archive to the system you plan to bundle, and untar:

`tar zxvf centos-xen-modules.tar.gz -C /`

### Generating a correct /etc/fstab

It's almost always a good idea to use the --generate-fstab option, since most physical systems won't have the correct fstab for a Eucalyptus guest. If you're re-bundling an existing Eucalyptus instance, you don't need to specify this.

### Using --no-inherit

If your Eucalyptus environment is not running in Managed or Managed (No VLAN) networking modes, or if you're bundling an existing system either physical or virtual not running on Eucalyptus, use the --no-inherit option. The euca-bundle-vol command will attempt to contact the Eucalyptus metadata server to implant instance information into your bundled image, but this only works in Managed and Managed (No VLAN) modes.

## Using euca-bundle-vol:

If you're bundling a Eucalyptus instance, you can use a command like this:

    euca-bundle-vol -p my-new-img -d /mnt/ -e /var/lib/dhclient -s 4096

If you're bundling a virtual machine or a physical machine outside of Eucalyptus, use --no-inherit and --generate-fstab, and specify an appropriate kernel and ramdisk IDs (eki and eri) based on the kernel modules that you copied in during the previous step:

    euca-bundle-vol -p my-new-img -d /mnt/ -e /var/lib/dhclient -s 4096 \
     --generate-fstab --no-inherit --kernel eki-xxxxxx --ramdisk eri-xxxxxx

This will produce a bunch of image "chunks" and an XML manifest file. The next step is to upload the image to Eucalyptus:

    euca-upload-bundle -b bucket-name -m /tmp/my-new-img.manifest.xml

Finally, register it. This will produce an EMI ID.

    euca-register bucket-name/my-new-img.manifest.xml

## Common Problems

_My bundled image immediately goes from "pending" to "terminated" state._

Please make sure that you're using a compatible kernel and ramdisk for the Linux distribution you are bundling. Check the /var/log/eucalyptus/nc.log on the node controller for more details, as well as /var/lib/eucalyptus/instances/work/*/_instance-id_/console.log for bootup details.

_My system appears to boot, but doesn't get an IP address, or I can't connect to the IP address._

Please make sure you've excluded ALL networking configuration from the bundled image, including MAC address settings and DHCP leases, and make sure that the bundled VM is configured in DHCP mode.

*****
[[category.docs]] 
[[category.images]]