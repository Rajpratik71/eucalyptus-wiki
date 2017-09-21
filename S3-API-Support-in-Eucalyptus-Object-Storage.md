
## Supported S3 API operations:
 _(as of 4.0.0)_ 

Authentication:


* Pre-signed URLs (with expiration)
* Regular REST-style
* Path-style requests (e.g. GET [hostname.com/bucket/object](http://hostname.com/bucket/object))
* Virtual-hosted style requests (e.g. [bucket.hostname.com/object](http://bucket.hostname.com/object))
* AWS STS token support

Authorization:


* Object ACLs (including AWS canned-acls, e.g. private, public-read, aws-exec-read, ec2-bundle-read, public-read-write...)
* Bucket ACLs (inlcuding AWS canned-acls)
* S3 IAM policy (specified via the EUARE/IAM service endpoint in Eucalyptus)

Service:


* List Buckets (GET on service), including pagination

Bucket:


* GET/PUT/DELETE bucket
* GET/PUT/DELETE bucket ACL
* GET/PUT/DELETE bucket versioning status (enable, suspend, etc)
* GET/PUT/DELETE bucket logging (not present in 4.0, but available in 3.4.x, will be re-introduced post 4.0)
* GET bucket location (no effect, but messages accepted)
* Bucket & object lifecycle (added in Eucalyptus 4.0)
* Bucket tagging

Object:


* HEAD/GET/PUT/DELETE/POST object (including object-copy)
* Object versioning
* Object ACLs
* POST (also known as browser-based) uploads of objects
* Multi-part uploads (added in Eucalyptus 4.0)
    * Initiate, upload part, list parts, list uploads, abort MPU, complete MPU

    


## [](https://github.com/eucalyptus/eucalyptus/wiki/Walrus-S3-API#unsupported-so-far-operations)Unsupported (so far) operations

* Bucket policies
* Delete Multiple Objects (POST /?delete, not DELETE)
* Storage durability classes/Reduced Redundancy Storage (not yet applicable)
* Website hosting
* CORS support
* OPTIONS on object
* Bucket request payment
* Bucket notification (for ReducedRedundancy lost objects, not applicable to Walrus yet)
* Bittorrent support

Many of these features are in development now and this list will be updated as support is added.



*****

[[category.storage]] 
[[category.confluence]] 
