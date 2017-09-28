This wiki is 's notes of Ceph RGW architecture from the following tech talk. This is an artifact of EUCA-11612, understanding the mapping between RGW and RADOS




## RADOS features utilized by higher layer applications


| RADOS feature | Application | 
|  --- |  --- | 
| Partial overwrite | RBD | 
| Extended attributes | RGW uses extended attributes for frequently accessed metadata such as S3 user metadata | 
| Omap key value pair | RGW uses omap for object listing also known as bucket index featureObject map/omap is ordered key value mapping associated with a RADOS object. It is stored in level db instance of the OSD | 
| Atomic read/write |  | 



*****

[[category.storage]] 
[[category.confluence]] 
[[category.objectstorage]]
