# What's changed in Eucalyptus 4.4.0?

The intent of this page is to highlight the behavioral changes present in Eucalyptus 4.4.0.

**NOTE: This is not a list of changes in 4.4.0. The release notes are still the most complete source for that information.**
***
# Installation and configuration changes
## Eucalyptus
* Properties (euctl) and empyrean/bootstrap (euserv-*) requests are now handled by the cloud controller [EUCA-12926](https://eucalyptus.atlassian.net/browse/EUCA-12926)

## Management Console
* Non-configurable and developer-only settings have been removed from the default console.ini file. [GUI-2844](https://eucalyptus.atlassian.net/browse/GUI-2844)

# Service changes
## General
* Mule is no longer used for messaging.
* Thread naming is changed [EUCA-12357](https://eucalyptus.atlassian.net/browse/EUCA-12357)

## EC2
* Instances and other resources are no longer owned by the eucalyptus account after their owning iam user is deleted [EUCA-12921](https://eucalyptus.atlassian.net/browse/EUCA-12921)

## EC2/VPC
* NAT gateway ENI metadata changes [EUCA-12850](https://eucalyptus.atlassian.net/browse/EUCA-12850)
* No error returned when deleting a Nat Gateway which does not exist [EUCA-12845](https://eucalyptus.atlassian.net/browse/EUCA-12845)

## ELB
* Access log metrics are updated to match aws [EUCA-12865](https://eucalyptus.atlassian.net/browse/EUCA-12865)
* Instance type 'm1.medium' is now the new default for ELBs [EUCA-12834](https://eucalyptus.atlassian.net/browse/EUCA-12834)
* ELB backend service is removed [EUCA-12030](https://eucalyptus.atlassian.net/browse/EUCA-12030)
* ELB is updated to use swf [EUCA-11885](https://eucalyptus.atlassian.net/browse/EUCA-11885)

## Auto Scaling
* ASG fails to distribute instances across AZ in some cases [EUCA-12846](https://eucalyptus.atlassian.net/browse/EUCA-12846)

## CloudFormation
* Template resource properties are now validated by default [EUCA-10359](https://eucalyptus.atlassian.net/browse/EUCA-10359)

## CloudWatch
* cloudwatch.disable_cloudwatch_service property is now cloudwatch.enable_cloudwatch_service [EUCA-9482](https://eucalyptus.atlassian.net/browse/EUCA-9482) 	

## IAM
* Policies with invalid JSON syntax are now rejected by default [EUCA-12829](https://eucalyptus.atlassian.net/browse/EUCA-12829)
* IAM aws:userid policy variable for assumed role now includes session name [EUCA-12802](https://eucalyptus.atlassian.net/browse/EUCA-12802)

## STS
* Session name is now validated when assuming a role [EUCA-12747](https://eucalyptus.atlassian.net/browse/EUCA-12747)

# Networking changes
* Support for managed modes is removed
* Cloud properties are updated [EUCA-11830](https://eucalyptus.atlassian.net/browse/EUCA-11830)