## DesignDoc for the Cloud Admin
### Backing Store (where the buckets are)
Currently, Walrus requires a POSIX filesystem to store buckets and objects. It uses, by default, the filesystem at $EUCALYPTUS/var/lib/eucalyptus/bukkits and creates a directory per bucket and each object is stored as a single file within that bucket directory.

### How is the Backing Store Used
Bucket directories are named exactly as the bucket itself, so bukkits/bucket01 is the directory for the 'bucket01' bucket, as expected. However, object files do not retain their name on the filesystem for several reasons and are instead stored as files which have a random hash as the name. This prevents name conflicts due to object versioning and allows the system to handle object names which are not valid POSIX filenames (as the S3 API allows '/' in object names, for example).

This does of course, mean that the number of buckets in a single Walrus installation is limited by the number of subdirectories allowed by the filesystem. For ext3 that number is 32,000 and for ext4 it is 64,000. XFS and btrfs have much higher limits (i.e. > 1M). XFS also has many other advantageous features such as full metadata journalling that allows repair of the FS without a full fsck run. There is plenty of info online for all the filesystems, but we suggest XFS or ext4. For XFS info see [XFS documentation](http://xfs.org/index.php/XFS_Papers_and_Documentation)

Walrus does not utilize the filesystem for storing metadata, that is all persisted in the database managed by the Cloud Controller (CLC). Therefore, Walrus does not require extended metadata support from the filesystem itself.

### HA (DRBD and more)
To provide high-availability, Walrus leverages DRBD to replicate the backing block storage device at the block level. This was chosen for several reasons, one of which is that it handles replication such that once the block device reports a write is complete the system can be sure that it has been written to disk on both hosts and is not simply in a memory buffer somewhere awaiting a lazy write. One of the key issues with distributed and/or shared filesystems as a replication mechanism is that they typically heavily utilize caching to boost performance (ehem..NFS) at the cost of data consistency in the case of unexpected failures. Walrus uses a local filesystem on top of a replicated block device to ensure that completed writes are truly completed on both hosts.

Walrus uses DRBD for a single volume replicated using replication protocol C ([DRBD Replication](http://www.drbd.org/users-guide/s-replication-protocols.html)) in an active-passive configuration. As per all of Eucalyptus's HA components, one Walrus node is _ENABLED_ and one is _DISABLED_ such that only the _ENABLED_ node services user requests. The enabled node has a DRBD block device (volume) mounted to the filesystem at $EUCALYPTUS/var/lib/eucalyptus/bukkits and acts as primary role. On the disabled Walrus node the DRBD volume is unmounted (to the filesystem) and is the secondary role in DRBD terms. If the disabled Walrus node becomes enabled it becomes primary on the DRBD volume and mounts it in the filesystem.

Walrus does not configure DRBD itself, but expects the administrator to have configured a block-device as a DRBD volume and specifies that volume to Walrus via the `walrus.blockdevice=<device>` and `walrus.resourcename=<drbd resource, i.e. res0>` properties. Those properties are set via `euca-modify-property -p <propertyname>=<propertyvalue>` and can be displayed using `euca-describe-properties`.

### Lifecycle of a Request
Get and Put examples
### Walrus API
Walrus implements much of the Amazon Web Services Simple Storage Service (S3) API: [S3 Documentation](http://aws.amazon.com/documentation/s3/)

Specifically, Walrus supports: get/put/head/post of objects and buckets, object/bucket ACLs, and object/bucket versioning. Walrus does not yet implement static websites, bucket-policies, degraded durability modes, or multi-part uploads. We are, of course, working on many of those features, but can't give a timeline just yet. Walrus supports the S3 SOAP API as well as REST and supports normal S3 authorization-header-based request authentication or query-string authentication.

### Walrus and the Database
Walrus uses a database to manage metadata for buckets and objects as well as ACLs and policies. Walrus has its own database but is currently co-hosted by the Cloud Controller (CLC) with all of the other databases that Eucalyptus uses. The current implementation leverages a PostGreSQL database although version prior to Eucalyptus 3.1 ran on MySQL.

Walrus, like all Eucalyptus components written in Java, utilizes Hibernate to interact with the db layer and database HA is handled via a replicating Hibernate JDBC connection. For more info on how the DB is managed and synchronized for HA, see the CLC documentation.

### Authentication, ACLs, Permissions
See the [S3 Documentation](http://aws.amazon.com/documentation/s3/). Walrus implements and supports S3-style ACLs and IAM policies. It does not support bucket-policies or bucket-lifecycles (yet!).

## DesignDoc for the Developers
code is the ultimate truth here

*****

[[category.components]]
[[category.storage]]