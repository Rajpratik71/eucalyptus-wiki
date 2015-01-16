# What's changed in Eucalyptus 4.1.0? 
(Early draft. Look for more updates to this mid-2014)

The intent of this page is to highlight as many of the changes present in Eucalyptus 4.1.0 as possible with particular emphasis on behavioral changes that do not involve API or CLI changes.

**NOTE: This is not a list of all changes in 4.1.0. The release notes are still the most complete source for that information.**
***

## AWS Compatibility

***
## Services
### VPC
A tech preview implementation of EC2-VPC is added.

### EBS
Snapshots can now be shared between accounts.

### Instance Timeout and Status
The default timeout for instances is increased from 12 hours to 180 days. On upgraded systems the administrator should adjust the value accordingly.

When an instance is not being reported by an NC the instance will now show a failing status check. The instance status is also now available as a CloudWatch metric.

### CloudFormation
The CloudFormation service is no longer a tech preview.

### Simple Workflow
A tech preview implementation of Simple Workflow is added, and will be installed by default for use by CloudFormation. The Simple Workflow service is not available for use by regular accounts/users by default, but can be enabled via a cloud property.

### Credential Changes
There are now default limits for IAM access key and signing certificate credentials, these limits can be modified via the "authentication.*" cloud properties.

Credential downloads in 4.1 will (by default) only include a signing certificate if the user does not have a certificate. This behavior can be changed using the "authentication.*" cloud properties.

## IAM
EC2 resource ARNs now allow a region wild-card. ELB resource ARNs are now permitted in policy.

Policy documents returned from the service are now urlencoded.

### Database
Schemas are now used rather than databases for partitioning. The new "eucalyptus_shared" database now contains all the schemas.

Database connections to the local host now use the localhost interface rather than the registration interface.

A shorter database password is now used, this allows standard PostgreSQL tools to connect using the password.

### Listeners
SSLv3 is now disabled by default and the available SSL protocols can be configured via a new cloud property.

### Service Life-cycle
### Installation and Configuration Changes
### HA-specific Changes
### DNS Changes
***
## Reporting and Quota Changes
### Quotas and Policies
User quotas for EC2 addresses were incorrectly counting addresses in use by the account, not the user. The address usage by the user is now used when checking the quota.

### Account provisioning
***
## SLAs and Placement Control
*** 
## Images and Disk Geometry
### Image Management/Upload Changes
### Image Service Changes
### Image Validation Service Changes
### Block Device Mapping Changes
### VmwareBroker Changes
***
## Networking
### VPC Networking Changes

***
## Storage
### Object Storage Changes
### Walrus Changes
### SAN-related Changes
### Storage Controller (Block storage) Changes
***
## Graphical Console Changes
### User Console 
#### Multi-cloud
### IAM Account Admin Console
### Cloud Resource Admin Console
***
## euca2ools Changes
***
## Eustore Changes
***
## Logging and Troubleshootability Changes
***
## Cloud Operational Changes
### Eucalyptus Monitoring Changes
### Backup and Restore Changes
### Audit Logging Changes
### Resource Management Changes
***
## Performance Changes
***
## Packaging and Distribution Changes
***
## Documentation Changes (non-content related)
***
## QA Changes
***
## Deprecations 
(no more bug fixes; no longer QA'ed; no SLA; may / may not be able to use the feature)
***