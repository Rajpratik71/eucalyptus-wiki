# What's changed in Eucalyptus 4.2.0? 

The intent of this page is to highlight the behavioral changes present in Eucalyptus 4.2.0.

**NOTE: This is not a list of changes in 4.2.0. The release notes are still the most complete source for that information.**
***

# Service changes

## CloudFormation
* Nested stacks no longer pass CAPABILITY_IAM by default [EUCA-10756](https://eucalyptus.atlassian.net/browse/EUCA-10756)
* New "region.region_name" property is now preferred for local region configuration [EUCA-10995](https://eucalyptus.atlassian.net/browse/EUCA-10995)

## CloudWatch
* EC2 system metrics are now dropped when the backlog becomes too large [EUCA-11413](https://eucalyptus.atlassian.net/browse/EUCA-11413)

## DNS
* Legacy DNS implementation removed [EUCA-10438](https://eucalyptus.atlassian.net/browse/EUCA-10438)
* DNS services now run on user facing hosts and are registrable[EUCA-10439](https://eucalyptus.atlassian.net/browse/EUCA-10439)

## EC2
* Many Describe* actions are now handled by user facing service hosts rather than the cloud controller [EUCA-10660](https://eucalyptus.atlassian.net/browse/EUCA-10660)
* Public and Elastic IPs are now allocated non-sequentially [EUCA-3698](https://eucalyptus.atlassian.net/browse/EUCA-3698)
* Image bundle manifest signatures are no longer verified on registration, any X.509 certificate can be used for signing (need not be an IAM signing certificate) [EUCA-10838](https://eucalyptus.atlassian.net/browse/EUCA-10838)
* State for in-use public and elastic IPs is now persisted to metadata_addresses table [EUCA-11072](https://eucalyptus.atlassian.net/browse/EUCA-11072)
* Instance restore for managed modes now defaults to "restore-failed" (restore with failing instance reachability) [EUCA-10674](https://eucalyptus.atlassian.net/browse/EUCA-10674)
* Cloud controller now allocates network resources for managed mode (private ip, network tag/index, mac address) [EUCA-10579](https://eucalyptus.atlassian.net/browse/EUCA-10579) [EUCA-10679](https://eucalyptus.atlassian.net/browse/EUCA-10679)

## EC2 Imaging
* Imaging service now runs in a separate "(eucalyptus)imaging" system account [EUCA-10102](https://eucalyptus.atlassian.net/browse/EUCA-10102)
* Paravirtual images no longer require imaging service [EUCA-11140](https://eucalyptus.atlassian.net/browse/EUCA-11140)
* Some imaging tasks are now performed by the cloud controller rather than on user facing service hosts [EUCA-11443](https://eucalyptus.atlassian.net/browse/EUCA-11443)
* Imaging service implementation uses cloud-formation and the new "esi-*" tools to prepare resources [EUCA-10192].
(https://eucalyptus.atlassian.net/browse/EUCA-10192)

## EC2 VPC
* Default VPC resources are now created on first use of EC2 rather than on account creation [EUCA-10809](https://eucalyptus.atlassian.net/browse/EUCA-10809)

## ELB
* ELB resources are now owned by a separate "(eucalyptus)loadbalancing" system account [EUCA-10102](https://eucalyptus.atlassian.net/browse/EUCA-10102)
* ELB now creates a system owned auto scaling group for each zone [EUCA-11245](https://eucalyptus.atlassian.net/browse/EUCA-11245)
* New properties for tuning scalability are added [EUCA-11127]
(https://eucalyptus.atlassian.net/browse/EUCA-11127)
* HTTPS/SSL listeners are configured with the latest security policy by default [EUCA-10985]
(https://eucalyptus.atlassian.net/browse/EUCA-10985)

## IAM
* HMAC signature replay check removed [EUCA-10878](https://eucalyptus.atlassian.net/browse/EUCA-10878)
* SHA-512 now used for hashing passwords (was MD5) [EUCA-2306](https://eucalyptus.atlassian.net/browse/EUCA-2306)
* Global uniqueness of signing certificates is now enforced [EUCA-10567](https://eucalyptus.atlassian.net/browse/EUCA-10567)
* Authorization information is now cached for a short period [EUCA-10887](https://eucalyptus.atlassian.net/browse/EUCA-10887) 	
* User registration status is removed [EUCA-9481](https://eucalyptus.atlassian.net/browse/EUCA-9481)
* Most policy tables are removed, the textual representation for policy is used at runtime [EUCA-10610](https://eucalyptus.atlassian.net/browse/EUCA-10610)

## Reporting
* Reporting functionality is now disabled by default [EUCA-10611](https://eucalyptus.atlassian.net/browse/EUCA-10611)

# Installation and configuration changes
* Active/passive (HA) cloud controller support removed [EUCA-11163](https://eucalyptus.atlassian.net/browse/EUCA-11163)
* HA-JDBC configuration files no longer present in /var/run/eucalyptus/tx [EUCA-11163](https://eucalyptus.atlassian.net/browse/EUCA-11163)
* Infinispan distributed caching removed [EUCA-10676](https://eucalyptus.atlassian.net/browse/EUCA-10676)
* Managed modes configuration is now centralized as per EDGE/VPCMIDO modes [EUCA-10579](https://eucalyptus.atlassian.net/browse/EUCA-10579)
* Heap size for eucalyptus-cloud service is now set according to system resources by default [EUCA-10794](https://eucalyptus.atlassian.net/browse/EUCA-10794)
* Permissions for log files in /var/log/eucalyptus have changed [EUCA-11171](https://eucalyptus.atlassian.net/browse/EUCA-11171)
* The "eucarc" file for cloud administrators now contains endpoints for bootstrap (euserv) and properties administrative services [EUCA-11351](https://eucalyptus.atlassian.net/browse/EUCA-11351)

