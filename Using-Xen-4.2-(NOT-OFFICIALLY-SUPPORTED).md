## THIS IS NOT OFFICIALLY SUPPORTED
## PLEASE USE AT YOUR OWN RISK

## Install Xen 4
http://wiki.centos.org/HowTos/Xen/Xen4QuickStart
```
yum install centos-release-xen
yum install xen
/usr/bin/grub-bootxen.sh
reboot
```
If libvirtd is already installed (ie you are converting an existing NC) ensure that you upgrade that package as well to the one from the XEN repo.

## Eucalyptus Config Changes
On the NC change the HYPERVISOR option to xen

## Cloud resource differences from KVM
BFEBS images were registered with /dev/vda as the root device which XEN didnt understand, reregistered with /dev/sda worked
Needed to upload and register XEN kernels and ramdisks for my images and reregister new emi's.

## Completed verifications
* BFEBS instances
* Volume attachment