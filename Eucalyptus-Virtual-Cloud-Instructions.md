# EVC Alpha 1 Release

### Virtual Machine Download
[evc-3.3.0-alpha-1.tar.gz](http://downloads.eucalyptus.com/software/evc/testing/evc-3.3.0-alpha-1.tar.gz)

## System Requirements

* Fedora 18 or newer (or a modern Linux installation that supports nested KVM). 
* A system that supports virtualization. Be sure to check the BIOS to make sure that virtualization is enabled for your system.

## Installing KVM

Install KVM using your package manager:

    yum install kvm

or

    apt-get install kvm

## Enabling nested KVM 

The following instructions are for Fedora.

First, make sure KVM kernel modules are loaded:

    modprobe kvm
    modprobe kvm_intel

Or if you're on an AMD system:

    modprobe kvm
    modprobe kvm_amd

Second, check if nested KVM is enabled:

    cat /sys/module/kvm_intel/parameters/nested

If it returns 'Y' then you have nested KVM enabled!

If it returns 'N' then you need to enable nested KVM by running:

    echo "options kvm-intel nested=1" > /etc/modprobe.d/kvm-intel.conf

Or if you have an AMD chip then replace **kvm-intel** with **kvm-amd**.

## Unpacking the EVC

Copy the EVC to your system (if you got this from a USB stick) and run the following commands:

    cd my/install/location
    tar -zxvf evc-filename.tar.gz (check the filename)
    cd evc-3.3.0-alpha (cd into the resultant directory)

## Setting Directory Permissions

The parent directory that you unpack the EVC to will need execute permissions
for other users, e.g.:

    chmod o+x my/install/location

More information on why this is necessary is available [here](http://libvirt.org/drvqemu.html#securitydac).

## Running the EVC

The EVC must be run as the root user on your system and must be run from the
directory where the EVC is unpacked.

    sudo ./run-evc.sh

## Shutting down the EVC

You can shut your EVC down like any other virtual machine. Once it is down, you
can remove the virtual network bridge by running:

    sudo ./teardown-evc.sh

## Logging into the EVC instance

To log into the EVC instance, be sure that you have a VNC viewer of some type installed.  (Note that you can access the EVC through your web browser without using vnc.)

## EVC Credentials

Once you've booted your EVC you can access services with the following credentials:

* Demo User
  + Login: demo
  + Password: demo

* Superuser:
  + Login: root
  + Password: foobar

* User Console Login:
  + URL: https://192.168.42.2:8888
  + Account: demo
  + User: admin
  + Password: admin

* Admin Console Login:
  + URL: https://192.168.42.2:8443
  + Account: eucalyptus
  + User: admin
  + Password: admin

*****
[[category.install]]