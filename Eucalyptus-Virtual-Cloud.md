The **Eucalyptus Virtual Cloud** is basically a _CIABIAB_ (Cloud in a box in a box). When you use our CIAB installation from [Silvereye](https://github.com/eucalyptus/silvereye), you get a complete Eucalyptus cloud that can run on a single bare metal machine. With a **Eucalyptus Virtual Cloud** we go one step further. Now you can take that single machine _CIAB_ and install it into a _Virtual Machine_. For something like this to work, the [Hypervisor](https://en.wikipedia.org/wiki/Hypervisor) that is being used on the machine hosting the VM must support [nested virtualization](http://en.wikipedia.org/wiki/Nesting_%28computing%29).

The purpose of this document is to provide the necessary information to set up a **Eucalyptus Virtual Cloud**. It provides the installation and configuration procedures, as well as lists of supported _Hypervisors_ and tools that will be needed to make this work.

# Hypervisor Support

## Supports Nested Virtualization

+ [KVM](http://www.linux-kvm.org/page/Main_Page) supports nested virtualization on Fedora, but not CentOS/RHEL
  - <http://kashyapc.wordpress.com/2013/02/12/nested-virtualization-with-kvm-and-intel-on-fedora-18/>
  - <http://www.rdoxenham.com/?p=275>
+ VMware Workstation/Player/Fusion supports nested virtualization, but there might be legal issues here
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
2. VMware Workstation/Player/Fusion (since this could run on a Mac, though there may be legal issues with this)

These are our top two, but getting this working on other platforms would be an awesome bonus.

## Enabling Nested Virtualization in Parallels

It is possible to enable nested virtualization in Parallels, but it does not seem to work. In order to enable nested virtualization change into the directory of your virtual machine.

    $ cd ~/Documents/Parallels/CentOS\ Linux.pvm

Once inside, edit the file called `config.pvs` and edit the tag called _VirtualizedHV_ so that the value is `1` instead of `0`.

While this does enable nested virtualization and allows CIAB to install into the virtual machine, I cannot run any instances. The computer will completely seize up and must be rebooted. My last glimmer of hope, that the logs might contain something of use, was completely dashed; the logs ended with the following:

    NVMX:   The guest start using virtualization!

And that was all. So, it appears that nested virtualization does not work in Parallels.

## Check for Nested Virtualization in KVM

If you'd like to check your system to see if it supports nested virtualization in KVM, you can check the parameters exposed by the kernel module. If you're using an Intel chip, then the module is `kvm-intel`; or if you're using an AMD chip then the module is `kvm-amd`.

Here's an example check:

    ~% modinfo kvm-intel | grep nested
    parm:           nested:bool

In this example (run on Fedora 18) we can see that the `nested` parameter is available and so our platform supports nested virtualization with KVM.

# Goals

1. Absolutely _NO_ configuration necessary to have a working demo cloud for the user to explore.
2. Simple bridge setup on the host. While, yes, we'd like there to be no configuration, a bridge is still required to provide network connectivity to the EVC. We should at least have a script that configures this for the user.

# Resources

* How to install CIAB into a VM - <http://agrimmsreality.blogspot.com/2013/04/cloud-in-box-in-vm-in-nutshell.html>
* Install Eucalyptus 3.1 using SilverEye in our virtualized sandbox lab <http://www.inthevapor.com/2012/10/install-eucalyptus/>

*****

[[category.install]]
[[category.tools]]