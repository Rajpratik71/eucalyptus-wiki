
### Problem statement:
The current OSG implementation uses the eucalyptus_shared database with the eucalyptus_osg schema to store internal metadata about objects and buckets. These include: acls, ownership, bucket lifecycle configs, cors configs, etc. All subresources in the S3 API are managed as entities in the postgresql db. This poses a significant scale, throughput, and availability concern since the size of that db grows with the number of objects + buckets in the system which is not bounded by physical resources in the same way as instances are nor is there currently a viable active-active approach for the DB itself.


### Proposed Solutions:

1. Use the backend as a key-value store for all metadata about objects and buckets (including listings).
    1. Each S3 subresource is mapped to a distinct key and value and is versioned in coordination with the parent object
    1. object?acl→ hash(object)_acl, ?lifecycle→ hash(object)_lifecycle

    
    1. Object names must be computable based on bucket+key rather than a UUID so that any UFS can fetch them without a lookup to a mapping service to find the UUID for the requested object. (cloud_id) +SHA1(bucket+key) + version_id
    1. Challenge for performance is maintaining the list of the latest versions for handling requests
    1. Caches for bucket->object mapping, write-thru

    
    1. Store original object name in backend metadata on object? Space restrictions?
    1. Would allow reconstruction of mapping/caches on total failure/shutdown and required cache priming.
    1. Must store original object name somewhere for generating object listings and version listings

    
    1. Handoff of caches/partition space to other OSGs?
    1. Adds complexity, but since effectively stateless, can be somewhat inefficient if needed (consistent hashing is best)

    

    
    1. Current metadata managers in OSG are abstracted to allow reasonable replacement of DB usage with something else

    
1. Use the backend object metadata to store S3 sub-resources and metadata about objects (distinct from user metadata)
    1. Depends on each back-end's metadata capabilities and limits.
    1. Must consider user metadata + system metadata against total limit imposed by the backend system (e.g. S3 API user-metadata is 2KB limit, so must preserve that).

    









*****

[[category.storage]] 
[[category.confluence]] 
[[category.objectstorage]]
