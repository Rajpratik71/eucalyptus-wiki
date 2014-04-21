# What's changed in Eucalyptus 4.0.0?

The intent of this page is to highlight as many of the changes present in Eucalyptus 4.0.0 as possible with particular emphasis on behavioral changes that do not involve API or CLI changes.

**NOTE: This is not a list of all changes in 4.0.0. The release notes are still the most complete source for that information.**
***

## AWS Compatibility
***
## Services
### Persona induced Changes
* Default roles are now setup:
```
[root@g-15-08 eutester]# euare-rolelistbypath
arn:aws:iam::362121614306:role/eucalyptus/AccountAdministrator
arn:aws:iam::362121614306:role/eucalyptus/InfrastructureAdministrator
arn:aws:iam::362121614306:role/eucalyptus/ResourceAdministrator
[root@g-15-08 eutester]#
```
## Cloud admin GUI has been removed and IAM functionality moved to the User Console
### Things that are missing
1. Account Signup
2. See Service Components and state
3. Change VM types (done via CLI)
4. Create/delete accounts (done via CLI)
5. Downloading X.509 Certs (done via CLI)

### Service Life-cycle
### Installation and Configuration Changes
### HA-specific Changes
1. Dump-restore is now the default sync strategy. More info about HA-JDBC and its config can be found here: http://ha-jdbc.github.io/doc.html

### DNS Changes
### ELB Changes
1. ELB Session Stickiness
2. SSL certs

***
## Images and Disk Geometry
### Image Management/Upload Changes
### Image Service Changes
### Block Device Mapping Changes
### VmwareBroker Changes
***
## Networking
### Edge-networking Changes
* daemon/services/executable renamed to eucanetd

### SYSTEM Mode Changes
* Deprecated

### STATIC Mode Changes
* Deprecated

***
## Storage
### Object Storage Changes
### Walrus Changes
### SAN-related Changes
***
## Graphical Console Changes

1. Fully redesigned 
2. Added user/group/policy administration for account admins

***

## euca2ools Changes
### Eustore Changes

* Eustore commands have been removed from Euca2ools
* emis.eucalyptus.com will now be the landing point to get an image
* euca-install-image script will exist to bundle/upload/register an image in 1 step

***

## Logging and Troubleshootability Changes

* [Logging Enhancements](https://eucalyptus.atlassian.net/secure/Dashboard.jspa?selectPageId=15705)

***
## Deprecations 
### (no more bug fixes; no longer QA'ed; no SLA; may / may not be able to use the feature)

* SYSTEM and STATIC networking modes