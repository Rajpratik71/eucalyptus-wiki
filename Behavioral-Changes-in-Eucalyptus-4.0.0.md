# What's changed in Eucalyptus 4.0.0?

The intent of this page is to highlight as many of the changes present in Eucalyptus 4.0.0 as possible with particular emphasis on behavioral changes that do not involve API or CLI changes.

**NOTE: This is not a list of all changes in 4.0.0. The release notes are still the most complete source for that information.**
***

## AWS Compatibility
***
## Services
### Persona-induced Changes
* Default roles are now setup:
```
[root@g-15-08 eutester]# euare-rolelistbypath
arn:aws:iam::362121614306:role/eucalyptus/AccountAdministrator
arn:aws:iam::362121614306:role/eucalyptus/InfrastructureAdministrator
arn:aws:iam::362121614306:role/eucalyptus/ResourceAdministrator
[root@g-15-08 eutester]#
```
### Service separation-induced Changes
* Admin tools that previously used EC2_URL to locate other services (e.g. empyrean) now default to localhost instead.  Use the --url option if you need to run admin tools from somewhere other than the CLC.
* Services are now registered (Auto Scaling, CloudWatch, EC2, ELB, IAM, STS) with euca-(de)register-service
* User-api service group can be used to register all user facing services (the above plus S3/object storage)

## Cloud admin GUI has been removed and IAM functionality moved to the User Console
### Things that are missing
1. Account Signup
2. See Service Components and state
3. Change VM types (done via CLI)
4. Create/delete accounts (done via CLI)
5. Downloading X.509 Certs (done via CLI)
6. Report Generation (done via CLI)

### Service Life-cycle
### Installation and Configuration Changes
### HA-specific Changes
1. Dump-restore is now the default sync strategy. More info about HA-JDBC and its config can be found here: http://ha-jdbc.github.io/doc.html
2. IPs returned in credentials can now be configured through a configurable property

### DNS Changes
1. DNS listener address can now be configured through a system property (defaults to bind-addr)
2. IPs returned on service DNS names can now be configured through a system property

### EC2 Changes
1. Resource identifiers now all lower case ([EUCA-8501](https://eucalyptus.atlassian.net/browse/EUCA-8501))
2. Multiple security groups for instances now supported (EDGE mode only)
3. Terminated instance cache removed (terminated instances remain in DB until they expire)
4. Endpoint for EC2 is now http://compute.example.com:8773/services/compute the old (eucalyptus) host and path are still supported.

### ELB Changes
1. ELB Session Stickiness
2. SSL certs
3. ELB image is now HVM rather than paravirt
4. ELB now uses DNS to locate cloud services

***
## Images and Disk Geometry
### Image Management/Upload Changes
* Paravirtualized(PV) images require kernel and ramdisk ID supplied when either bundling or registering.
* Running a PV image will trigger image conversion at the first time the image is run. The instance will stay in pending until conversion completes. The conversion delay is proportional to the size of EMI and typically will take couple minutes. If multiple PV images are run the same time, the delay will be multiplied by number of such EMIs. 
* After a PV image is converted, subsequence run-instance will not incur image conversion (no delay).
* The PV images and instances in conversion are tagged with 'euca:image-conversion-state' and 'euca:image-conversion-status'. The tag will disappear when the image is fully converted.

### Image Service Changes
* For import-volume, import-instance, and running PV images, there will be a worker VM that takes care of image copying and conversion.
* By default, there is one imaging worker VM per availability zone.
* Worker VM and it's associated resources (such as autoscaling group) are tagged with 'euca-internal-imaging-worker'.
* A worker VM is automatically run when the system is given import tasks or PV instance run for the first time (across the cloud).
* It is OK to terminate a worker VM.
* It is NOT ok to delete autoscaling group, launch configs, security groups tagged as imaging worker.
* If resources other than worker VM is deleted (by accident), setting property 'imaging.imaging_worker_enabled' false will clean up resources held by imaging workers. Then import tasks or pv image run will setup resources again properly.
* One can change desired_capacity of worker VM's autoscaling group, if the system expects high load of image conversion or import tasks.
* The size of EMI that can be converted is limited by VM type of the worker VM. In a nutshell, VM type should be larger than the size of PV EMI's root disk. One can change the VM type by setting 'imaging.imaging_worker_instance_type'
 
### Block Device Mapping Changes
### VmwareBroker Changes
***
## Networking
### Edge-networking Changes
* daemon/services/executable renamed to eucanetd
* most site-specific configuration is now done through uploading a JSON format config file to Eucalyptus via euca-modify-property -f cloud.network.network_configuration=</path/to/JSON/config/file>.  There are still a limited number of basic networking configuration settings still required in eucalyptus.conf on CC/NCs.

### SYSTEM Mode Changes
* Deprecated

### STATIC Mode Changes
* Deprecated

***
## Storage
### Object Storage Changes
* The OSG is a new component and is the sole provider of the S3 API in Eucalyptus
* Service path is now: /services/objectstorage, but /services/Walrus is honored and handled by the OSG itself.
* walrus.max_bucket_size_mb is no longer supported. Size limitations must be enforced via IAM policies.

### Walrus Changes
* Walrus component in 3.4.x is now called 'WalrusBackend' and has a new service path: /services/WalrusBackend
* Walrus is no longer user-accessible. It only responds to requests from the eucalyptus/admin user (and even that will be disabled in 4.1).
* Walrus no longer needs to be directly reachable from NCs. Only OSGs need reachability. And only the OSG needs to be able to reach Walrus, though Walrus must be able to reach the DB etc and have multicast.
* Total snapshot size limit is no longer an object-storage/Walrus property, but a 'storage' property. See below for more details.
* Max buckets and bucket size properties are no longer enforced by Walrus. All usage limitations are handled by the OSG.

### EBS/SAN-related Changes
Total snapshot size allowed for the whole cloud is now a global storage property, not a walrus property: storage.global_total_snapshot_size_limit_gb=50 (default is 50GB). This is because the object storage service doesn't treat snapshots specially, only the Storage Controllers know that the objects in object storage are actually snapshots.

***
## Graphical Console Changes

1. Fully redesigned 
2. Added user/group/policy administration for account admins
3. SSL configuration is different, not on by default
4. users won't see data change on-screen except for items in transitional state. Previous version would automatically refresh data shown on-screen.

***

## euca2ools Changes
### Eustore Changes

* Eustore commands have been removed from Euca2ools
* emis.eucalyptus.com will now be the landing point to get an image
* euca-install-image script will exist to bundle/upload/register an image in 1 step

***

## Logging and Troubleshootability Changes

* [Logging Enhancements](https://eucalyptus.atlassian.net/secure/Dashboard.jspa?selectPageId=15705)
* cloud-requests.log now logs requests (Java components)

***
## Deprecations 
### (no more bug fixes; no longer QA'ed; no SLA; may / may not be able to use the feature)

* SYSTEM and STATIC networking modes