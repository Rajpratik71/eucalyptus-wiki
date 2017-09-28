
## User Metadata
Per S3-specification, users may include metadata in object PUT operations using the special header prefix: 'x-amz-'. Example: 'x-amz-mymetadata: my metadatavalue1'. This metadata is returned with GET requests on the object as well as HEAD requests against the object.

The OSG does not persist this metadata or manage it directly itself, it is the responsibility of the OSP to manage user metadata. The guiding principle is that all opaque user data should be handled by the OSP and user metadata is opaque to the system.


## Internal Metadata
The OSG itself keeps metadata about buckets and objects.

Persistence Context/DB: 'eucalyptus_osg' database


* Objects - com.eucalyptus.objectstorage.ObjectEntity class, 'objects' table
* Buckets - com.eucalyptus.objectstorage.Bucket class, 'buckets' table
* Parts - com.eucalyptus.objectstorage.PartEntity class, 'parts' table
* Bucket LifecycleRules - com.eucalyptus.objectstorage.BucketLifecycleRule class, 'lifecycle_rules' table.
* ObjectEntity and PartEntity have a FK dependency on Bucket


### [](https://github.com/zhill/architecture/blob/dev/zhill/objectstorage-arch/features/object-storage/docs/scalable_walrus.md#entity-states)Entity states and Transitions:

* ObjectEntity: creating, extant, mpu-pending, deleting
* BucketEntity: creating, extant, deleting
* PartEntity: creating, extant, deleting

 **Entity State Semantics**  _creating:_ Entity has been initialized and persisted by OSG and the operation is in progress.

 _extant:_ Entity was successfully created and is available to users.

 _mpu-pending:_ Multipart Upload, for ObjectEntity records to indicate the presence of a valid uploadId but the MPU upload has not been completed or aborted. The record in this state contains a valid (known by the backend/OSP) uploadID, but does not have any content size or eTag yet.

 _deleting:_ Entity is marked for deletion. It is no longer visible to users via the API. This is persisted to allow asynchronous cleanup on failure conditions as needed

Entity State TransitionsObjectEntity:

ObjectEntityStateTransitions

PartEntity:


1. creating ==> extant
1. creating ==> deleting
1. extant ==> deleting

BucketEntity:


1. creating ==> extant
1. creating ==> deleting
1. extant ==> deleting




## Object/Bucket Mapping From OSG to OSP
In order to ensure data safety and facilitate features such as [object versioning](http://docs.aws.amazon.com/AmazonS3/latest/dev/Versioning.html), the OSG performs a mapping between the bucket/key that the user requests and the actual bucket/key that is presented to the OSP for persistence. The mapping is to identify each bucket with a UUID as well as each object for each PUT operation with a UUID. Thus, each object PUT has a unique key for the OSP even if the user specifies the same key. The UUIDs are of type 4 (pseudo-randomly generated), so there is no direct computational mapping from a key name to a UUID. This characteristic is necessary to ensure that 2 keys will never overwrite each other in the OSP, and it facilitates advanced features like versioning that are not available in many OSP implementations natively.

Example:

User sends OSG: PUT mybucket/mykey

OSG Maps it to: PUT 1231231233-1231231231-1231321321/456456-4545646-45645465

The OSP receives: PUT 1231231233-1231231231-1231321321/456456-4545646-45645465

If the user sends a 2nd PUT for the same key, 'mybucket/mykey' it gets a different UUID

OSG Maps it to: PUT 1231231233-1231231231-1231321321/456456-4545646-xxxxxx

The OSP receives: PUT 1231231233-1231231231-1231321321/456456-4545646-xxxxx







*****

[[category.storage]] 
[[category.confluence]] 
[[category.objectstorage]]
