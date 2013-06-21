# Purpose

This page is meant to be a collection of notes on what Eucalyptus does for internal datacenter deployments (e.g., deploying Eucalyptus for QA or development). It's come time to rethink how we do deployment, and this page documents all the things that happen during that process.

## Cobbler Provisioning

* Cobbler provisions machines for a deployment of Eucalyptus.
* Uses Supported OS: CentOS/RHEL 6.x

## Pre-install Operations

* Parse cloud configuration (memo field)
* Adjust routes: `route del 192.168.51.150 dev eth0`
* Modifies hostname to `eucahost-C-D.eucalyptus` except when using kickstart. (should we do this?)
* Copies remote ssh configuration to `/root/.ssh/config`
* Set `PERSISTENT_DHCLIENT=1` for default interfaces (i.e., eth0 and em1)
* Disable firewall by running `rm -f /etc/sysconfig/iptables` and `service iptables stop`
  + This should probably be `service iptables stop` and `chkconfig iptables off`
  + We might want to consider _not_ stopping iptables
* Set IP for internal distro mirrors in `/etc/hosts` (should not need this)
* Insert authorized keys and known hosts (including public key for the developer, if they supply one)
* Increase loop devices to 256 (don't think this is necessary)

### On RHEL Only

* Download scripts for registering RHEL
* Perform RHEL registration

*****
[[category.releng]]