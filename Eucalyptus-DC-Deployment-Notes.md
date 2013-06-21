# Purpose

This page is meant to be a collection of notes on what Eucalyptus does for internal datacenter deployments (e.g., deploying Eucalyptus for QA or development). It's come time to rethink how we do deployment, and this page documents all the things that happen during that process.

**NOTE:** Things that we probably shouldn't do anymore will be ~~crossed out~~.

## Cobbler Provisioning

* Cobbler provisions machines for a deployment of Eucalyptus.
* Uses Supported OS: CentOS/RHEL 6.x

## Pre-install Operations

* Parse cloud configuration (memo field)
* ~~Adjust routes: `route del 192.168.51.150 dev eth0`~~
* ~~Modifies hostname to `eucahost-C-D.eucalyptus` except when using kickstart~~
* Copies remote ssh configuration to `/root/.ssh/config`
* Set `PERSISTENT_DHCLIENT=1` for default interfaces (i.e., eth0 and em1)
* Disable firewall by running `rm -f /etc/sysconfig/iptables` and `service iptables stop`
  + This should probably be `service iptables stop` and `chkconfig iptables off`
  + We might want to consider _not_ stopping iptables
* ~~Set IP for internal distro mirrors in `/etc/hosts`~~
* Insert authorized keys and known hosts (including public key for the developer, if they supply one)
* ~~Increase loop devices to 256~~
* ~~Set lots of kernel parameters~~
* Add EPEL repository
* Add Elrepo repository
* Add extra user specified repositories

### On RHEL Only

* Download scripts for registering RHEL
* Perform RHEL registration

## Dependency Installation
* Run `yum update`
* Install ntp with `yum install ntp`
* Sync clock with `ntpdate qa-server`
* `chkconfig ntpd on`
* `hwclock --systohc`
* Start the ntp service with `service ntpd start`

## Source Build Dependency Installation
* Install development packages for source builds
* Install dependency packages for source builds

## Environment Configuration
* ~~Run `iptables -L` to load necessary kernel modules~~

### Node Controller Configuration
* Update primary ethernet configuration file `/etc/sysconfig/network-scripts/ifcfg-em1` to use bridge (em1)
  + Append the line `BRIDGE="br0"`
* Add bridge interface configuration file `/etc/sysconfig/network-scripts/ifcfg-br0`
  + Bridge configuration file content:
    `DEVICE="br0"
    TYPE="Bridge"
    BOOTPROTO="dhcp"
    ONBOOT="yes"
    DELAY=0`
* Remove second interface configuration file `/etc/sysconfig/network-scripts/ifcfg-em2`
* Run `service networking restart`

## Eucalyptus Installation
* Check git repository url
  + Is it the Eucalyptus OSS repository?
  + Is it the Eucalyptus internal repository?
    * Add `.repo` file for the enterprise packages

*****
[[category.releng]]