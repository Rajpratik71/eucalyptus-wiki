# EVC Alpha 1 Release

## System Requirements

Fedora 18 or newer (or a modern Linux installation that supports nested KVM).

## Directory Permissions

The parent directory that you unpack the EVC to will need execute permissions
for other users. For instance, if you were to unpack the EVC into your home
directory, you would need to run:

    chmod o+x ~

More information is available [here](http://libvirt.org/drvqemu.html#securitydac).

## Enabling nested KVM in Fedora

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

    cd my/install/location
    tar -zxvf evc-3.3.0-alpha-1.tar.gz
    cd evc-3.3.0-alpha-1

## Running the EVC

The EVC must be run as the root user on your system and must be run from the
directory where the EVC is unpacked.

    sudo ./run-evc.sh

## Shutting down the EVC

You can shut your EVC down like any other virtual machine. Once it is down, you
can remove the virtual network bridge by running:

    sudo ./teardown-evc.sh

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
[[category:install]]