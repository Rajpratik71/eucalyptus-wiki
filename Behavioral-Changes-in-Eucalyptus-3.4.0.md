# What's Changed in Eucalyptus 3.4.0?

The intent of this page is to highlight as many of the changes present in Eucalyptus 3.4.0 as possible with particular emphasis on behavioral changes that do not involve API or CLI changes.

**NOTE: This is not a list of all changes in 3.4.0. The release notes are still the most complete source for that information.**

### Service Life-cycle
* 'eucalyptus-cloud' start-up is notably slower than in 3.3.x due to much more robust start-up checks, extensive logging of the start-up process, and serialization of service start-up to remove race-conditions and add stability.
* Starting a non-CLC java service, let's call it serviceA, before the CLC is running (e.g. starting a remote SC before CLC) will result in the service only partially bootstrapping and then waiting for the CLC. Once the CLC is running, the serviceA will intentionally begin a timer to wait for the system to stabilize. Logs will indicate this. serviceA will not continue bootstrapping until that timer expires, which is approximately 3 minutes. During this time any 'euca-describe-services' calls to the CLC will return the state of serviceA as 'NOTREADY'.

## HA-specific Changes

### DB Sync
* Ability to change sync strategy to dump restore (not fully tested thus not the default yet)

## DNS Changes
* DNS properties have been moved from the ```experimental.dns``` namespace to just ```dns```

## ELB Changes
* Allow NTP server to be set by euca property
* Restricted ports config property

## CloudWatch Changes
* Property added to disable cloudwatch completely: ```cloudwatch.disable_cloudwatch_service```

## Walrus Changes
* CanonicalIds are now used for owner identification in bucket and object ACLs. Previously, Account numbers were used as CanonicalIds but now a different id, unique for each account is created on upgrade and will be used for Walrus ACLs. The way to determine your account's CanonicalId is to get the ACL for a bucket your account owns. This is consistent with S3 practices.
* BucketACL can be set with grantee/user's email address as well. The email address can be found with the following command,
```bash
# update user with email address
euare-userupdateinfo -k email -i admin@example.com -u admin

# get email address associated with user
euare-usergetinfo -u admin
email	admin@example.com
```
* Set BucketACL with email address (example with AWS Java SDK),
```java
    Grant grant = new Grant( new EmailAddressGrantee( "admin@example.com" ), Permission.Read );
    AccessControlList acl = s3Client.getBucketAcl( "bucketName" );
    acl.getGrants().add( grant );
    s3Client.setBucketAcl( "bucketName", acl );
```

## Image Management/Upload Changes
* New configurable maximum allowed image size. A configurable property: ```cloud.images.max_image_size_gb``` lets the Eucalyptus admin set the maximum image size(GB) allowed for instance-storage images at registration time. The default value is 30GB (the same as the default Walrus image cache size). Attempting to register an instance-store image that exceeds this size results in a registration error indicating that the image is too large. To disable the check altogether simply set the value to a non-positive integer.

## SANs
### Equallogic
* Allow pool name to be set explicitly. It is not required, but the cloud admin has the ability to change the default. Default pool is still "default"

## AWS Compatibility
### Deregister Image
* When the image is deregister, it will no longer be visible to any non-eucalyptus account. From Eucalyptus admin account the image will be shown as ``deregistered`` when ``verbose`` flag is applied. e.g ``euca-describe-images verbose``. To remove the image completely, either the image owner or the Eucalyptus admin will have to deregister the image again, e.g ``euca-deregister emi-80263E5C``
* When the image is deregistered completely, any account that will try to deregister the image again should get an AWS semantic error,
``euca-deregister: error (InvalidAMIID.NotFound): The image ID 'emi-80263E5C' does not exist``
* If the image is in either deregistered state or removed completely, RunInstance will fail with the following error when the image is deregistered completely,
``euca-run-instances: error (RunInstancesType): Failed to lookup image named: emi-80263E5C``
* If the image is deregistered when there is an instance associated with the image, the instance will still be running, but but the image (EMI) won't be usable any more. Deregister again on the same image will fail with the following error,
``euca-deregister: error (InvalidAMIID.Unavailable): The image ID 'emi-C50A428B' is no longer available``
The image will not be visible to non-eucalyptus account/user, but Eucalyptus admin will be able to see it in ``deregistered`` state with ``verbose`` flag. When all the instances associated with that image will be ``terminated``, the image will be cleaned up automatically in the background.
