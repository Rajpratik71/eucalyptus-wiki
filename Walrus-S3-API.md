## Walrus supported S3 API operations:
_(as of 3.3.0)_

Authentication:
* Pre-signed URLs (with expiration)
* Regular REST-style
* Path-style requests (e.g. GET hostname.com/bucket/object)
* Virtual-hosted style requests (e.g. bucket.hostname.com/object)
* AWS STS token support

Authorization:
* Object ACLs (including AWS canned-acls)
* Bucket ACLs (inlcuding AWS canned-acls)
* S3 IAM policy (specified via the EUARE/IAM service endpoint in Eucalyptus)

Service:
* List Buckets (GET on service)

Bucket:
* GET/PUT/DELETE bucket
* GET/PUT/DELETE bucket ACL
* GET/PUT/DELETE bucket versioning status (enable, suspend, etc)
* GET/PUT/DELETE bucket logging
* GET bucket location (Walrus can handle the messages, but Walrus is single-region)

Object:
* GET/PUT/DELETE object (including object-copy)
* Object versioning
* Object ACLs
* POST (also known as browser-based) uploads of objects

## Unsupported (so far) operations
* Bucket policies
* Bucket & object lifecycle
* Multi-part uploads
* Delete Multiple Objects (POST /?delete, not DELETE)
* Storage durability classes/Reduced Redundancy Storage (not yet applicable)
* Website hosting
* CORS support
* OPTIONS on object
* Bucket request payment
* Bucket tagging
* Bucket notification (for ReducedRedundancy lost objects, not applicable to Walrus yet)

Many of these features are in development now and this list will be updated as support is added.

[[category.storage]]