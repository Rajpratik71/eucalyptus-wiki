Node Controllers (NCs) are designed to communicate with Ceph via libvirt and Ceph-RBD driver for qemu. More information about this design decision can be found [[here|Use-RBD-kernel-module-or-Qemu-KVM-RBD-driver-only-for-Ceph-integration]]. Node Controller interaction with Ceph-RBD requires
* Ceph client
* Hypervisor with Ceph-RBD support.

The following instructions will walk you through steps for installing and configuring these two requirements. Repeat this process for every NC in the Eucalyptus cluster


## Steps for installing Ceph client
Refer to[[Ceph Client Installation For EBS|Ceph-Client-Installation-For-EBS]]


## Steps for installing hypervisor with Ceph-RBD support
Support for Ceph-RBD driver is available only in the RHEV packages of Qemu (qemu-kvm and qemu-img).




1. Verify if qemu-kvm and qemu-img are already installed


```
yum list qemu-kvm qemu-img
```
Proceed to step 4 if there are not installed


1. Verify qemu support for ceph-rbd driver


```
qemu-img --help
qemu-img version 0.12.1, Copyright (c) 2004-2008 Fabrice Bellard
...
Supported formats: raw cow qcow vdi vmdk cloop dmg bochs vpc vvfat qcow2 qed vhdx parallels nbd blkdebug host_cdrom 
host_floppy host_device file gluster gluster gluster gluster rbd
```
If 'rbd' is listed as one of the supported formats no further action is required

If 'rbd' is not listed as one of the supported formats, continue reading


1. Uninstall qemu-kvm and qemu-img packages. This will also uninstall libvirt if it was installed. Make a copy of the current /etc/libvirt/libvirtd.conf to be on the safe side. Yum uninstallation process will save the current copy to/etc/libvirt/libvirtd.conf.rpmsave

If eucalyptus-nc service is running terminate/stop all instances via Eucalyptus. After all instances are terminated, stop the eucalyptus-nc service


```
service eucalyptus-nc stop
```

```
yum remove qemu-kvm qemu-img
```

1. Prepare RHEV qemu packages (Addresses where to get the RHEV packages from question):
    * If this NC is a RHEL system and RHEV subscription to qemu packages is available, consult the RHEV package procedure to install qemu-kvm-rhev and qemu-img-rhev packages. Blacklist the RHEV packages in eucalyptus repo to ensure that packages from the RHEV repo are installed ( to review this for documentation). No further action is required and the remaining steps can be skipped
    * If this NC is a RHEL system and RHEV subscription to qemu packages is unavailable, eucalyptus built and maintained qemu-rhev packages may be used. These packages are available in the same yum repository as other eucalyptus packages. Note that using eucalyptus built RHEV packages voids the original RHEL support for the qemu packages (to review this for documentation). Proceed to next step
    * If this NC is a non RHEL (CentoOS) system, eucalyptus built and maintained qemu-rhev packages may be used. These packages are available in the same yum repository as other eucalyptus packages.Proceed to next step

    

    

    
1. Install eucalyptus built RHEV packages: qemu-kvm-rhev and qemu-img-rhev.


```
yum install qemu-kvm-rhev qemu-img-rhev
```

1. Install libvirt


```
yum install libvirt
```

1. If libvirt was uninstalled in step 3, copy the saved libvirtd.conf file to /etc/libvirt/libvirtd.conf.Otherwise modify the/etc/libvirt/libvirtd.conf file as per your requirements.
1. Start libvirtd service


```
service libvirtd start
```

1. Verify qemu support for ceph-rbd driver as indicated in step 2


1. If eucalyptus-nc service was stopped in step 3, start it


```
service eucalyptus-nc start
```



## Related articles
[[Ceph Client Installation For EBS|Ceph-Client-Installation-For-EBS]]







*****

[[category.storage]] 
[[category.confluence]] 
