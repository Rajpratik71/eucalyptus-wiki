An EBS backed EMI is registered with an EBS snapshot as the root device. The root device for the instance launched from this EMI is in an EBS volume created from the snapshot. You can create an EBS backed EMI from an existing bootable .img file or create a new one. The following section describes how to create a bootable .img file. If you already have one, skip to the next section

### Create a bootable .img file

One way to create your own bootable .img file is to use virt-install as described below.

>Use virt-install on a system with the same operating system version and hypervisor as your Node Controller. If you use an image created by virt-install under a different distribution or hypervisor combination, it is likely that it will not install the correct drivers into the ramdisk and the image will not boot on your Node Controller.

1. On the designated host, use the QEMU disk utility to create a disk image:  
```
qemu-img create -f raw bfebs.img 2G
```
2. Use parted to set the disk label:  
```
parted bfebs.img mklabel msdos  
```
3. Use virt-install to start a new virtual machine installation using the disk image you just created:
```
virt-install --name rhel6 --ram 1024 --os-type linux --os-variant rhel6 \
-c /tmp/media.iso --disk \
path=/tmp/bfebs.img,device=disk,bus=virtio --vnc
```
4. Once you have completed the installation, start the virtual machine using virt-manager or other libvirt tool of your choice.
5. Configure the virtual machine by connecting to it and making the following changes:  
   a. Comment out the HWADDR entry from the /etc/sysconfig/network-scripts/ifcfg-eth0 file: 
```
DEVICE="eth0"
BOOTPROTO="dhcp"
#HWADDR="B8:AC:6F:83:1C:45"
IPV6INIT="yes"
MTU="1500"
NM_CONTROLLED="yes"
ONBOOT="yes"
TYPE="Ethernet"
UUID="499c07cc-4a53-408c-87d2-ce0db991648e"
PERSISTENT_DHCLIENT=1
```
   b. Add the following option to the end of /boot/grub/menu.lst to get a serial console:
```
console=ttyS0
```
   c. Remove the quiet option from the kernel parameters and grub menu splash image in the /boot/grub/menu.lst file.  
   d. Add the following line to the /etc/sysconfig/network file to disable the zeroconf route, which can interfere with access to the metadata service:
```
NOZEROCONF=yes
```
   e. Edit /etc/udev/rules.d/70-persistent-net.rules and remove the entry for the existing NIC.  
   f. Copy the Eucalyptus rc.local file from [here](https://github.com/eucalyptus/Eucalyptus-Scripts/blob/master/rc.local).  
   g. Customize the image as necessary.

### Create the EBS backed EMI

Creating an EBS-backed EMI will require initial assistance from a helper instance. The helper instance can be either an instance store or EBS-backed instance and can be deleted when finished. It only exists to help prepare the initial volume that will be the source of the snapshot behind the EBS-backed EMI.

1. Launch a helper instance.
2. Create a volume big enough to hold the bootable .img file.
```
euca-create-volume -z <cluster_name> -s <size_in_GB>
```
3. Attach the volume to the helper instance.
```
euca-attach-volume <volume-id> -i <instance-id> -d <device>
```
4. Log in to the instance and copy the bootable image to the attached volume by performing one of the following steps:  
If the bootable image is saved in an http or ftp repository, use curl or wget to download the .img file to the attached volume:
```
curl <path_to_bootable_image> > <device>
```
If the bootable image is from a source other than an http or FTP repository, copy the bootable image from its source to the helper instance, and then copy it to the attached volume using the dd command. For example:
```
dd if=<path_to_bootable_image> of=<device> bs=1M
```
5. Detach the volume from the instance:
```
euca-detach-volume <volume-id>
```
6. Create a snapshot of the volume:
```
euca-create-snapshot <volume-id>
```
7. Register the snapshot:
```
euca-register --name <image-name> --snapshot <snapshot-id> --root-device-name <device>
```

Read about changes to this feature in Eucalyptus 3.3 [here](https://github.com/eucalyptus/eucalyptus/wiki/Boot-from-EBS-changes-in-Eucalyptus-3_3_0).

***
[[category.ebs]]  
[[category.storage]]