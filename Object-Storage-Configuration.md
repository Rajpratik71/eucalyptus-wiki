
## Selecting the OSP
One of the first considerations during configuration of the object storage system is deciding which OSP to use. Eucalyptus currently supports 2 OSPs: Walrus (including HA Walrus), and [RiakCS.](http://basho.com/riak-cloud-storage/)

The OSP selection must be made before any OSGs can become functional. The specified value is used for all OSGs and is configurable after registering at least one OSG.


*  _objectstorage.providerclient=[walrus|riakcs]_ 



 _RiakCS/S3Provider Configuration_ These configuration properties are only applicable for the 'riakcs', and 's3' provider clients. Using Walrus does not require any of these to be set.


*  _objectstorage.s3provider.s3endpoint=[S3 REST API endpoint that OSGs will use to interact with OSP]–_ This may be a single host or the IP/DNS name of a load balancer by which all RiakCS nodes are available
*  _objectstorage.s3provider.s3accesskey=[accesskey credential for OSP/backend]–_ The RiakCS or S3 access key to be used by the OSG for communicating with the backend
*  _objectstorage.s3provider.s3secretkey=[secretkey credential for OSP/backend]–_ The RiakCS or S3 secret key to be used by the OSG for communicating with the backend. This is stored in encrypted form in the database (encrypted with the OSG's public key).


## [](https://github.com/zhill/architecture/blob/dev/zhill/objectstorage-arch/features/object-storage/docs/scalable_walrus.md#configuring-the-osgs)Advanced configuration of the OSG(s)

*  _objectstorage.queue_size=[number of chunks to buffer before declaring timeout]_ -- Sets the amount of 100K data buffers that the OSG will allow to be kept before failing a PUT operation because the data is not being transfered to the OSP quickly enough relative to the sender

    

    
*  _objectstorage.bucket_creation_wait_interval_seconds=[seconds to wait max for bucket creation on OSP]_ -- Controls the amount of time a bucket can be in the 'creating' state awaiting a response from the OSP. This is typically only used to cleanup metadata in the DB in the case where an OSG fails mid-operation and cannot complete the state update for the bucket


*  _objectstorage.bucket_naming_restrictions=[dns-compliant|extended]_ -- Determines how the OSG will validate bucket names during bucket creation. 'dns-compliant' is strict DNS compliance and S3 compatible for non-US-Standard regions. 'extended' is the naming scheme that includes extra, non-dns characters as implemented by S3's US-Standard region.


*  _objectstorage.cleanup_task_interval_seconds=[seconds between cleanup tasks, default=60]_ -- Time between cleanup task executions that clear buckets/objects that were not fully uploaded or failed state due to OSG failures.

    

    
*  _objectstorage.dogetputoncopyfail=[true|false]_ -- Sets the OSG to use GET-then-PUT if the OSP does not support object copy operations natively.

    

    
*  _objectstorage.failed_put_timeout_hrs=[# of hours to allow an object to remain in 'creating' state before deciding it has failed, default=168]_ -- Used to determine, if all other methods fail, when to time-out a PUT operation

    

    
*  _objectstorage.max_buckets_per_account=[bucket count, default=100]_ 
*  _objectstorage.max_total_reporting_capacity_gb=[some number of GBs, default: 2147483647]_ -- This is the number used for reporting capacity usage. This does NOT limit or restrict usage, only for reporting % usage as (total stored)/(reporting capacity). This may result in usage being > 100% in the Eucalyptus reporting system.



*****

[[category.storage]] 
[[category.confluence]] 
[[category.objectstorage]]
