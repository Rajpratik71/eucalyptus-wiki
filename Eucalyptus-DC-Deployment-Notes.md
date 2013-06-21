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

### Installation Prep
* Check git repository url
  + Is it the Eucalyptus OSS repository?
  + Is it the Eucalyptus internal repository?
      - Add `.repo` file for the enterprise packages
  + Add `.repo` file for the eucalyptus packages

### Cluster Controller Installation
* On all machines with the CC role:
  + Install the CC: `yum install eucalyptus-cc`
  + If we want VMWare Broker support, then run: `yum install eucalyptus-enterprise-vmware-broker`

### Cloud Controller Installation
* On all machines with the CLC role:
  + If this is a 3.2 cloud run: `yum groupinstall eucalyptus-cloud-controller`
  + Or if 3.3 then run: `yum install eucalyptus-cloud`
  + If we want VMWare Broker support, then run: `yum install eucalyptus-enterprise-vmware-broker-libs`
  + If we want SAN support, then install the proper SAN libs:
      - EMC SAN: `yum install eucalyptus-enterprise-storage-san-emc-libs`
      - NetApp SAN: `yum install eucalyptus-enterprise-storage-san-netapp-libs`
      - Equallogic SAN: `yum install eucalyptus-enterprise-storage-san-equallogic-libs` (only for 3.3 or newer)

### Storage Controller Installation
* On all machines with the SC role:
  + Install the SC package: `yum install eucalyptus-sc`
  + If this is an EMC SAN (i.e., has a provider of **EmcVnxProvider** or **EmcVnxProviderFastSnap**):
      - Download and install the RPM package `NaviCLI-Linux-64-latest.rpm` from the internal mirror
      - Install the EMC package: `yum install eucalyptus-enterprise-san-emc`
  + If this is a NetApp SAN:
      - Install the NetApp package: `yum install eucalyptus-enterprise-san-netapp`
  + If this is an Equallogic SAN:
      - Install the Equallogic package: `yum install eucalyptus-enterprise-san-equallogic`

### Walrus Installation
* On all machines with the Walrus role:
  + Install the Walrus package: `yum install eucalyptus-walrus`

### Node Installation
* Check if user wants euca2ools installed on the NC, and if so: `yum install euca2ools`
* Install the NC package: `yum install eucalyptus-nc`
* On RHEL only, we restart ISCSI: `service iscsid restart`

### Console Installation
* Install the Console package: `yum install eucalyptus-console`
* Start the console: `service eucalyptus-console start`

## Post-install Operations
* Create storage mount point: `mkdir -p /disk1/storage`
* Prepare and mount disk:
  + `mkfs.ext3 -F /dev/sda3`
  + `mount /dev/sda3 /disk1/storage`
  + Add `/etc/fstab` entry: `/dev/sda3    /disk1/storage    ext3    defaults    0 2`
  + `mkdir -p /disk1/storage/eucalyptus/instances`

### Extra Configuration for Source Builds
* Set `EUCALYPTUS` environment variable to `/opt/eucalyptus`
* Create some directories: `mkdir -p /disk1/storage/eucalyptus/var/{lib,log}/eucalyptus`
* Link directories:
  + `ln -sf /disk1/storage/eucalyptus/var/lib/eucalyptus/ /opt/eucalyptus/var/lib/`
  + `ln -sf /disk1/storage/eucalyptus/var/lib/eucalyptus/ /opt/eucalyptus/var/log/`
* If the directory `/opt/eucalyptus/var/lib/eucalyptus/vmware` exists, then do the following:
  + `rm -rf /opt/eucalyptus/var/lib/eucalyptus/vmware`
  + `mkdir -p /disk1/storage/eucalyptus/instances/vmware`
  + `ln -sf /disk1/storage/eucalyptus/instances/vmware /opt/eucalyptus/var/lib/eucalyptus/vmware/`

*****
[[category.releng]]