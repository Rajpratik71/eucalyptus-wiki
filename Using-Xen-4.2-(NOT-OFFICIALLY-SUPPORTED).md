## THIS IS NOT OFFICIALLY SUPPORTED
## PLEASE USE AT YOUR OWN RISK

## Install Xen 4
1. http://wiki.centos.org/HowTos/Xen/Xen4QuickStart
```
yum install centos-release-xen
yum install xen
/usr/bin/grub-bootxen.sh
reboot
```
2. If libvirtd was already installed (ie you are converting an existing NC) ensure that you upgrade that package as well to the one from the XEN repo.
```
yum upgrade libvirt
```

## Config Changes
1) On the Eucalyptus NC change the HYPERVISOR option to "xen" in /etc/eucalyptus/eucalyptus.conf

2) Edit the /etc/xen/xend-config.sxp file to be:
```
(xend-http-server yes)
(xend-unix-server yes)
(xend-unix-path /var/lib/xend/xend-socket)
(xend-address localhost)
(network-script network-bridge)
(vif-script vif-bridge)
(dom0-min-mem 196)
(dom0-cpus 0)
(vncpasswd '')
```

3) Restart xend and libvirtd
```
service xend restart
service libvirtd restart
```

4) Ensure Domain-0 shows up in virsh list
```
[root@node-2 /root]# virsh list
 Id    Name                           State
----------------------------------------------------
 0     Domain-0                       running
```

5) Start the NC
```
service eucalyptus-nc start
```

## Cloud resource differences from KVM
* BFEBS images were previously registered with /dev/vda as the root device which XEN didnt understand, reregistered with /dev/sda worked
* Needed to upload and register XEN kernels and ramdisks for my images and reregister new emi's.

## Completed verifications
* BFEBS instances
* Volume attachment