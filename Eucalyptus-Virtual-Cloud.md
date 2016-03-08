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
+ Parallels Desktop
  - <https://forum.parallels.com/threads/modyfying-vm-options-in-vagrant.315327/>


## Does _Not_ Support Nested Virtualization

+ Virtualbox
  - <https://www.virtualbox.org/ticket/4032>

## Virtualization Platforms We Want to Use

1. KVM (since we know this works)
2. Parallels Desktop (works and Vagrant provider is free)
3. VMware Workstation/Player/Fusion (since this could run on a Mac, though there may be legal issues with this)

These are our top three, but getting this working on other platforms would be an awesome bonus.

## Check for Nested Virtualization in KVM

If you'd like to check your system to see if it supports nested virtualization in KVM, you can check the parameters exposed by the kernel module. If you're using an Intel chip, then the module is `kvm-intel`; or if you're using an AMD chip then the module is `kvm-amd`.

Here's an example check:

    ~% modinfo kvm-intel | grep nested
    parm:           nested:bool

In this example (run on Fedora 18) we can see that the `nested` parameter is available and so our platform supports nested virtualization with KVM.

# Tool Evaluation

List various tools for evaluation that could be used to build EVC images.

## Veewee

[Homepage](https://github.com/jedi4ever/veewee)

### Overview

Veewee is a framework for building virtual machines for Vagrant, KVM and other providers. It has a simple configuration which consists of a virtual machine **definition**. The definition is a ruby script which may also call out to post installation scripts written in bash. This is actually a great tool since we can write a single configuration for building an image for multiple hypervisors.

### Problems

I was unable to get Veewee to build a CentOS image. Instead of injecting the kickstart configuration into the ramdisk image sends keystrokes to anaconda typing in the location of the kickstart file on a webserver (which is started by Veewee). The webserver was started successfully and I verified that I could view the kickstart file from my web browser, but the virtual machine was unable to reach it.

## Vagrant with Parallels

Add at least this to Vagrentfile:
>     config.vm.provider "parallels" do |v|

>        v.memory = 4096

>        v.cpus = 2

>        v.customize ["set", :id, "--nested-virt", "on"]

>        v.update_guest_tools = true

>      end

## Vagrant with VirtualBox

While this is a great approach, VirtualBox does not support nested virtualization.

## Oz

[Homepage](https://github.com/clalancette/oz/wiki)

## Virt-install/KVM

Probably the most simple and straightforward way to build a virtual machine.

## AMI-Creator

[Homepage](https://github.com/katzj/ami-creator)

Probably the _next_ most simple and straightforward way to build a virtual machine.

# Goals

1. Absolutely _NO_ configuration necessary to have a working demo cloud for the user to explore.
2. Simple bridge setup on the host. While, yes, we'd like there to be no configuration, a bridge is still required to provide network connectivity to the EVC. We should at least have a script that configures this for the user.

# Resources

* How to install CIAB into a VM - <http://agrimmsreality.blogspot.com/2013/04/cloud-in-box-in-vm-in-nutshell.html>
* Install Eucalyptus 3.1 using SilverEye in our virtualized sandbox lab <http://www.inthevapor.com/2012/10/install-eucalyptus/>

*****

[[category.install]]
[[category.tools]]