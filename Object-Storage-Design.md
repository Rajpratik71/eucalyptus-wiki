
# Design for Object Storage

## [](https://github.com/zhill/architecture/blob/dev/zhill/objectstorage-arch/features/object-storage/docs/scalable_walrus.md#overview)Overview
The pre-4.0 Walrus implementation is limited to a single host for servicing S3 requests as well as storing actual user data. This limits the system to scale-up models rather than scale-out. The objective of this design is to achive scale-out capabilities for Walrus in the dimensions of both IO/throughput and storage capacity.

Scalable Object Storage is a division of the Walrus implementation into two distinct pieces: Object Storage Gateway and multiple back-end providers (Walrus, RiakCS, S3) that provide the persistent storage. The Object Storage Gateway is client-facing and receives, authenticates, and routes requests to the appropriate backend. The Object Storage Gateway is on the data path for all data get/put operations but does not store data directly. It is stateless. The back-end services provide the persistent storage and state management for data objects and user metadata.Object Storage implements the Amazon S3 API:[Amazon Simple Storage Service (S3)](http://docs.aws.amazon.com/AmazonS3/latest/dev/Welcome.html)


## Guiding Principles:

1. Everything will fail
1. Never do now what you can do later if the user can't tell the difference (make it asynchronous)
1. Assume the backends are very dumb and not rely on any advanced features. This principle allows advanced features developed in the front-end to be applied to all backends and gives uniform features regardless of deployment configuration
1. Opaque user data is the responsibility of the backend. Eucalyptus should store as little information as possible to service requests
1. Lock-free: do not have any global locks or require global coordination unless absolutely necessary (almost never)


## [](https://github.com/zhill/architecture/blob/dev/zhill/objectstorage-arch/features/object-storage/docs/scalable_walrus.md#system-components)System components
The Scalable Object Storage is composed of the following components:


*  **Object Storage Gateway (OSG)**  - Multiplicity: many. This is the S3 API endpoint(s) that take user requests, manage metadata as needed, and dispatch requests to the OSP(s) to store/retrieve data.
*  **Object Storage Provider (OSP)**  - Multiplicity: many. This component (not necessarily part of Eucalyptus itself), is responsible for storing, retrieving, deleting, updating the object and bucket data.
*  **Database/Metadata persistence**  - Multiplicity: up to 2 (as of Eucalyptus 4.0). Object storage leverages the standard Eucalyptus database system (PostgreSQL + Hibernate) for the persistence of system metadata.
*  **Identity management**  - The standard Eucalyptus IAM implementation accessed via internal libraries. This is opaque to external users. All identities are managed by Eucalyptus IAM, not the Object Storage system iteslf.


## [](https://github.com/zhill/architecture/blob/dev/zhill/objectstorage-arch/features/object-storage/docs/scalable_walrus.md#object-storage-gateway-osg-overview)Object Storage Gateway (OSG)
The OSG receives user requests and authenticates and authorizes them using the standard Eucalyptus identity services (EUARE/IAM) as well as the Eucalytus internal account services (currently, the auth DB directly). If a message is accepted as authenticated then it is checked for authorization against IAM policies and if allowed, is dispatched to the appropriate Object Storage Provider (OSP). The interface between the OSG and OSP is an implementation detail of the OSG plugin used to communicate with the OSP directly.

The OSG is always run in an active/active configuration such that there may be many currently active OSG handling user requests. This mode of operation is independent of that of the OSP, and thus decouples the scale characteristics of the front and back-ends of the system. This is intended for both legacy support (Walrus is active/passive) as well as quick restart and recovery of the front-end components even in the event of complete backend failure. If no OSPs are available the OSGs may still handle requests and simply return HTTP-50X errors that the service is unavailable.


### External Interface
[Amazon S3 REST API](http://docs.aws.amazon.com/AmazonS3/latest/API/APIRest.html)


### Dependencies

* Eucalyptus IAM/Accounts
* Eucalyptus persistence/db
* OSP(s) for user data persistence


## [](https://github.com/zhill/architecture/blob/dev/zhill/objectstorage-arch/features/object-storage/docs/scalable_walrus.md#object-storage-provider-osp-overview)Object Storage Provider (OSP)
The Object Storage Provider is the "backend" of Object Storage. This compnent is responsible for persisting the actual user data and must support CRUD operations on data. This may or may not be an internal component of Eucalpytus. It may be an external service, eg. S3 or RiakCS.

Current supported OSPs:


* Walrus (non-HA) & Walrus (HA w/DRBD),
* [RiakCS](http://basho.com/riak-cloud-storage/)

Potential future OSPs:


* [Ceph](http://ceph.com/) (prototype using RadosGW already exists),
* [Swift](http://docs.openstack.org/developer/swift/)
* [Gluster](http://www.gluster.org/)
* many many others


### [](https://github.com/zhill/architecture/blob/dev/zhill/objectstorage-arch/features/object-storage/docs/scalable_walrus.md#external-interfaces)External Interfaces
Any API invocable from the OSG itself, but an OSP must support CRUD operations on objects and buckets.

[](https://github.com/zhill/architecture/blob/dev/zhill/objectstorage-arch/features/object-storage/docs/scalable_walrus.md#functional-design)Functional DesignThe OSG(s) handle _all_ client requests. Any client wishing to use Eucalyptus for Object Storage should be configured to connect to the OSG(s) using Eucalyptus user credentials and the S3 API. The OSP is unknown to the end-user and should be considered an internal component of Eucalyptus.

General request flow:


* User request enters OSG via S3 API.
* Request is mapped to internal bucket/key names for dispatch to OSP
* Request to OSP is configured and dispatched to OSP from OSG by OSP client running in the OSG.
* OSP returns result to OSG
* OSG maps result as needed and returns result to user as an S3 API response


## Asynchronous Tasks
Object deletions, bucket deletions, lifecycle expiration, and version history cleanup are all done asynchronously by a set of asynchronous tasks that run a predefined intervals. They are:


### Bucket Lifecycle Reaper
This job runs every day exactly once and scans and expires objects as defined by bucket lifecycle policy rules. See the bucket lifecycle design for more information. It runs nightly like a cron-job.


### Object Reaper
The Main Object Reaper scans the  _objects_  table for objects in the 'deleting' state and sends delete requests to the OSP for each. Once it gets positive confirmation that the delete succeeded it removes the db record for that object. It runs every 60 seconds. This reaper also looks for expired objects that were stuck in the 'creating' state due to failure/termination of an OSG during an operation. The 'creationExpiration' timestamp in the object db record indicates when the _creating_  state is no longer valid and the reaper finds these records and transitions them to _deleting_  state to be cleaned up.


### Bucket Reaper
The bucket reaper runs every 60 seconds and performs basically the same function as the object reaper, but just for buckets that are marked _deleting._  It may fail a bucket deletion if the objects in the bucket are all in _deleting_  but have not been reaped yet, in this case the backend may return a 'bucket not empty' error which the reaper accepts and leaves the bucket record untouched to by tried again later after the object reaper has fully cleaned the bucket on the backend/OSP.


## Basic Object GET/PUT Design
[[Object Versioning Design|Object-Versioning-Design]][[Bucket Lifecycle Design|Bucket-Lifecycle-Design]][[RiakCS Provider Design|RiakCS-Provider-Design]]
## WalrusProvider Design
[[OSG User and Internal Metadata|Object-Storage-Metadata]][](https://github.com/zhill/architecture/blob/dev/zhill/objectstorage-arch/features/object-storage/docs/scalable_walrus.md#osg-dependencies)[[OSG Security Considerations|Object-Storage-Security]][](https://github.com/zhill/architecture/blob/dev/zhill/objectstorage-arch/features/object-storage/docs/scalable_walrus.md#osg-internal-metadata)[[Configuration of Object Storage|Object-Storage-Configuration]]

*****

[[category.storage]] 
[[category.confluence]]
[[category.objectstorage]]
