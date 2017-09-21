This document represents the design and description of the process of upgrading a 3.4.x system running Walrus to a 4.0 system running one or more OSGs + Walrus.


### User-impacting changes
3.4.x S3 API endpoint is Walrus with service path: /services/Walrus4.0 S3 API endpoint is OSG(s) with service path: /services/objectstorage
### Administrative actions

* Run yum upgrade
* Run euca_conf --register-osg <config>
* Run euca-modify-property -p objectstorage.providerclient=walrus

Upgrading a Walrus system will require registering at least one OSG. Walrus is still part of the system but is only a backend component.
### Design of DB and Data Upgrade Process
Components Involved
* Walrus
    * Modify object and bucket metadata, including changing ownership of all buckets and objects to an internal user
    * Flush image cache metadata and disk content
    * Flush snapshot-specific metadata leaving snapshots as regular objects
    * Copy configuration (quotas etc) up to OSG db and flush from Walrus db.
    * Modify records for:
    * Bucket
    * Object
    * Combine key + versionId to create new key, versioning is no longer needed or supported on backend

    
    * Grants on Buckets/Objects
    * Parse into ACLs to translate to OSG db
    * After translation, remove all grants, every object/bucket in Walrus is 'private'

    

    

    
* Storage Controller
    * Populate snapshot metadata with size and path information, some copied from Walrus metadata

    
* OSG
    * Populate the new eucalyptus_osg db with all the proper bucket, object, and config entities
    * Bucket
    * Assign bucketuuid = bucketname where that is the bucket-name in Walrus db
    * Set state = extant

    
    * Object
    * Assign objectuuid = objectKey+versionId in Walrus Db.

    
    * Config

    

    



Walrus to OSG upgrade whiteboarding



*****

[[category.storage]] 
[[category.confluence]] 
