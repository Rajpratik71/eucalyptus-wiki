

| Complete | 
|  | 
| Qemu/KVM driver for NC and direct librbd access on SC.Requires: RHEV packages from RH or Eucalyptus (we will build them and provide them) on the NCs and librbd package on SCs.For Tech-Preview: use this solutionFor Ceph-Integration GA: use this solutionFor RHEL7 support: use this solution | 
|  | 
|  | 
|  --- | 
|  --- | 
|  --- | 
|  --- | 
|  --- | 
| Complete | 
|  | 
| Qemu/KVM driver for NC and direct librbd access on SC.Requires: RHEV packages from RH or Eucalyptus (we will build them and provide them) on the NCs and librbd package on SCs.For Tech-Preview: use this solutionFor Ceph-Integration GA: use this solutionFor RHEL7 support: use this solution | 
|  | 
|  | 


## Details on Decision
For Ceph support, NCs will use the Qemu/KVM RBD driver to access Ceph images directly from the hypervisor. The SC will leverage the librbd library via JNA bindings. The SC will require the installation of the librbd packages from Ceph/Inktank but nokernel or Qemu/KVM changes. **_Eucalyptus will build and distribute the RHEV packages (qemu-kvm-rhev and qemu-img-rhev)for non-RHEV subscribers to get RBD support in Qemu/KVM as well as a side benefit of added vm migration support._** 

NC Package Requirements for Ceph:


*  _qemu-kvm-rhev_  and  _qemu-img-rhev_ package from either RHEV or Eucalyptus's build of RHEV
* Ceph's  _librbd_  package and dependencies from Ceph

SC Package Requirements for Ceph:


* Cephs'  _librbd_  package and dependencies from Ceph

    

    

The Tech Preview release of Ceph RBD support in Eucalyptus 4.1.0 will use this strategy as will the GA-release of the feature in 4.1+.

This has two benefits:


1. By building our own RHEV packages we give customers VM migration support with non-shared storage, which RHEL/CentOS 6.5 removed from the stock Qemu/KVM packages and it is still absent in RHEL/CentOS 7. Thus, the only way for our non-RHEV -subscribing customers to get migration support in RHEL/CentOS 7 is for us to provide these packages.
1. Ceph RBD support in the RHEV qemu-img and qemu-kvm packages obviates the need for kernel modules

 **CentOS customers** : install the RHEV packages provided by Eucalyptus. Will require adding a repo.

 **RHEL customers:** 


1. If already a RHEV subscriber, then do nothing.
1. If the customer wants continued RH support for the Qemu/KVM services then they must purchase a RHEV subscription from RH for all NC hosts.
1. If the customer doesn't care about RH support for Qemu/KVM or doesn't want to pay more, then they can use the Eucalyptus-provided RHEV packages and RH will simply not support  **those specific packages** (basically the hypervisor) but the rest of the RHEL stack will continue to be supported by RH.


## Background
[EUCA-9655 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-9655)




1. 
### RHEL/CentOS7 considerations

1. VM migration without shared storage is still not present in v7 since its removal in v6.5. There are no current indications that such support will be re-added.
1. RHEV packages **DO**  contain that support
1. RHEV packages also contain RBD support




### Use kernel module only: SC and NC use RBD kernel modules + 'rbd map/unmap' to create block devices from RBD images.

1. Requires upgraded kernel for CentOS 6.5 on SC and NC - The LT kernels from elrepo work– v3.10.


```
yum --enablerepo=elrepo-kernel install kernel-lt
```

1. Kernel modules are not as fast for IO as the KVM driver


1. Will require NC/SC host restart




### Use Qemu/KVM + librbd only: SC uses JNA binding of librbd for data transfer to/from images

1. For NC - use qemu-kvm packages from Ceph/Inktank. Steps for installing ceph packages (subset of steps from [Ceph docs](http://ceph.com/docs/master/install/install-vm-cloud/#install-qemu))


    1. Add the ceph release key to the system's list of trusted keys to avoid a security warning


```
rpm --import 'https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc'
```

    1. Install yum-plugin-priorities.


```
yum install yum-plugin-priorities
```

    1. Ensure /etc/yum/pluginconf.d/priorities.conf exists.
    1. Ensure priorities.conf enables the plugin.


```
[main]
enabled = 1
```

    1. Create a /etc/yum.repos.d/ceph-extras.repo file with the following contents


```
[ceph-extras]
name=Ceph Extras
baseurl=http://ceph.com/packages/ceph-extras/rpm/centos6.3/$basearch
enabled=1
priority=2
gpgcheck=1
type=rpm-md
gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc
 
[ceph-qemu-source]
name=Ceph Extras Sources
baseurl=http://ceph.com/packages/ceph-extras/rpm/centos6.3/SRPMS
enabled=1
priority=2
gpgcheck=1
type=rpm-md
gpgkey=https://ceph.com/git/?p=ceph.git;a=blob_plain;f=keys/release.asc
```

    1. Ensure that non-priority versions are removed. Note that removing qemu-kvm and qemu-img uninstalls libvirt and saves /etc/libvirt/libvirtd.conf to /etc/libvirt/libvirtd.conf.rpmsave


```
yum remove qemu-kvm qemu-img
yum clean all
```

    1. Install QEMU for Ceph


```
yum install qemu-kvm qemu-img
```

    1. Install libvirt. If libvirt was uninstalled in step f, copy /etc/libvirt/libvirtd.conf.rpmsave to /etc/libvirt/libvirtd.conf to restore the configuration. Start libvirtd service


```
yum install libvirt
cp /etc/libvirt/libvirtd.conf.rpmsave /etc/libvirt/libvirtd.conf
service libvirtd start
```
NOTE: the Qemu/KVM driver is what both OpenStack and CloudStack use in their block implementations.



    
1. For NC - use RHEV qemu-kvm packages from RH for customers with RHEV subscriptions, and build our own RHEV packages for non-RHEV users. Steps for installing our own RHEV packages:
    1. Download the qemu-kvm and qemu-img packages on the NC (these are temporary URLs for demonstration purposes provided by Garrett. Final repos would be Euca repos.)


```
wget http://temp.devzero.com/l/ceph/qemu-img-rhev-0.12.1.2-0.2.415.el6_5.10.x86_64.rpm
wget http://temp.devzero.com/l/ceph/qemu-kvm-rhev-0.12.1.2-0.2.415.el6_5.10.x86_64.rpm
```

    1. Ensure that previously installed qemu-kvm and qemu-img packages are removed. Note that removing qemu-kvm and qemu-img uninstalls libvirt and saves /etc/libvirt/libvirtd.conf to /etc/libvirt/libvirtd.conf.rpmsave


```
yum remove qemu-kvm qemu-img
```

    1. Install the downloaded packages


```
yum localinstall qemu-img-rhev-0.12.1.2-0.2.415.el6_5.10.x86_64.rpm qemu-kvm-rhev-0.12.1.2-0.2.415.el6_5.10.x86_64.rpm
```

    1. Install libvirt. If libvirt was uninstalled in step b, copy /etc/libvirt/libvirtd.conf.rpmsave to /etc/libvirt/libvirtd.conf to restore the configuration. Start libvirtd service


```
yum install libvirt
cp /etc/libvirt/libvirtd.conf.rpmsave /etc/libvirt/libvirtd.conf
service libvirtd start
```


    
1. No restart required. If qemu-kvm and qemu-img packages that don't provide rbd support are already installed on the NC, they might need to be removed before installing rhev/ceph packages. This uninstalls libvirt package too and it has to be reinstalled post qemu package installation. The entire process is disruptive and all virtual machines running prior to change are lost.
1. For SC - use rados-java JNA bindings from[Ceph's git repo](https://github.com/ceph/rados-java). Repo provides source only. Either build the jar separately and drop it into eucalyptus dependencies or use the source code directly in eucalyptus. Found a missing operation in JNA bindings (flatten image), adding it to forked source code and rebuilding the jar fixed the issue.


## Action items

1.  to build RHEV qemu-kvm package internally for testing evaluation and make available to storage team
1.  to verify non-shared storage VM migration capabilities and ceph rbd support in RHEV packages built from source by us.
1.  to verifynon-shared storage VM migration capabilities and ceph rbd support in ceph packages.
1. Enumerate exactly, the steps a customer would be required to do for each option
    1. Kernel Module upgrade in CentOS/RHEL 6.5
    1. qemu-kvm package installation from RHEV
    1. qemu-kvm package installation from Ceph/Inktank
    1. qemu-kvm package installation from Eucalyptus

    



*****

[[category.storage]] 
[[category.confluence]] 
