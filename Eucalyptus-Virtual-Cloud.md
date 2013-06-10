The **Eucalyptus Virtual Cloud** is basically a _CIABIAB_ (Cloud in a box in a box). When you use our CIAB installation from [Silvereye](https://github.com/eucalyptus/silvereye), you get a complete Eucalyptus cloud that can run on a single bare metal machine. With a **Eucalyptus Virtual Cloud** we go one step further. Now you can take that single machine _CIAB_ and install it into a _Virtual Machine_. For something like this to work, the [Hypervisor](https://en.wikipedia.org/wiki/Hypervisor) that is being used on the machine hosting the VM must support [nested virtualization](http://en.wikipedia.org/wiki/Nesting_%28computing%29).

The purpose of this document is to provide the necessary information to set up a **Eucalyptus Virtual Cloud**. It provides the installation and configuration procedures, as well as lists of supported _Hypervisors_ and tools that will be needed to make this work.

# Hypervisor Support

## Supports Nested Virtualization

+ [KVM](http://www.linux-kvm.org/page/Main_Page) supports nested virtualization on Fedora, but not CentOS/RHEL
  - <http://kashyapc.wordpress.com/2013/02/12/nested-virtualization-with-kvm-and-intel-on-fedora-18/>
+ VMWare Player supports nested virtualization, but there might be legal issues here
  - <http://communities.vmware.com/docs/DOC-8970>
  - <https://github.com/mitchellh/vagrant/issues/1774>
+ Xen supports nested virtualization
  - <http://www.slideshare.net/xen_com_mgr/nested-virtualization-update-from-intel>

## Does _Not_ Support Nested Virtualization

+ Virtualbox
  - <https://www.virtualbox.org/ticket/4032>
+ Parallels (maybe?)
  - <http://forum.parallels.com/showthread.php?264780-Missing-quot-Nested-VT-x-EPT-quot-VM-Optimization-Setting-Parallels-8-amp-Windows-8>

## Virtualization Platforms We Want to Use

1. KVM (since we know this works)
2. VMWare Player (since this could run on a Mac, though there may be legal issues with this)

These are our top two, but getting this working on other platforms would be an awesome bonus.

# Goals

1. Absolutely _NO_ configuration necessary to have a working demo cloud for the user to explore.
2. Simple bridge setup on the host. While, yes, we'd like there to be no configuration, a bridge is still required to provide network connectivity to the EVC. We should at least have a script that configures this for the user.

# Resources

* How to install CIAB into a VM - <http://agrimmsreality.blogspot.com/2013/04/cloud-in-box-in-vm-in-nutshell.html>

*****

[[category.install]]
[[category.tools]]