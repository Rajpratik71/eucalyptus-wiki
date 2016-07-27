# What's changed in Eucalyptus 4.3.0?

The intent of this page is to highlight the behavioral changes present in Eucalyptus 4.3.0.

**NOTE: This is not a list of changes in 4.3.0. The release notes are still the most complete source for that information.**
***
# RHEL / CentOS 7 specific changes
The changes in this section are only relevant when 4.3.0 is installed on RHEL / CentOS 7.
* The startup.log is no longer written, /var/log/messages contains startup logging

# Installation and configuration changes
## Eucalyptus
* Java 8 is now used. Notable differences include SSL/TLS functionality, garbage collection, and permgen removal.
* Credential download functionality is removed [EUCA-10503](https://eucalyptus.atlassian.net/browse/EUCA-10503)
* The administration HTTP(S) ports (8080/8443) are no longer used 
* Multicast is switched from a global to local address [EUCA-8371](https://eucalyptus.atlassian.net/browse/EUCA-8371)
* We no longer use bitronix [EUCA-11933](https://eucalyptus.atlassian.net/browse/EUCA-11933)

## Eucalyptus Console
* Memcached installed by default and uses unix domain socket [GUI-2341](https://eucalyptus.atlassian.net/browse/GUI-2341)
* Concurrency library has been switched from gevent to eventlet to help with CentOS 7 installation [GUI-2473](https://eucalyptus.atlassian.net/browse/GUI-2473)

# Service changes
## General
* We now return a versioned server header in all service responses [EUCA-5037](https://eucalyptus.atlassian.net/browse/EUCA-5037)

## EC2
* Volume attachment now verifies that the instance is either running or stopped [EUCA-12373](https://eucalyptus.atlassian.net/browse/EUCA-12373)
* We now use udev to obtain EBS volume device info, not scsi_id [EUCA-12066](https://eucalyptus.atlassian.net/browse/EUCA-12066)
* Some invalid API parameters would have been incorrectly ignored [EUCA-12049](https://eucalyptus.atlassian.net/browse/EUCA-12049)
* Libvirtd is no longer restarted by the node controller on start [EUCA-11998](https://eucalyptus.atlassian.net/browse/EUCA-11998)
* Instance metadata has additional items such as dynamic data [EUCA-5601](https://eucalyptus.atlassian.net/browse/EUCA-5601)
* Volume quotas are now enforced when running instances. Launching an instance will now fail if the caller would exceed a volume limit [EUCA-9339](https://eucalyptus.atlassian.net/browse/EUCA-9339)

## EC2/VPC
* Eucanetd service no longer stops running for some error conditions [EUCA-12165](https://eucalyptus.atlassian.net/browse/EUCA-12165)
* Eucanetd no longer requires reverse DNS [EUCA-11997](https://eucalyptus.atlassian.net/browse/EUCA-11997)
* Network view now omits more information when unused (dhcp option sets, security groups)
* Route tables / routes are implemented [EUCA-11571](https://eucalyptus.atlassian.net/browse/EUCA-11571)

# S3
* IAM resource matching is now case insensitive changing the behaviour of existing policies  [EUCA-11534](https://eucalyptus.atlassian.net/browse/EUCA-11534)