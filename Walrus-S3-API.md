## Walrus supported S3 API operations:
_(as of 3.3.0)_

Service:
* List Buckets (GET on service)

Bucket:
* GET/PUT/DELETE bucket
* GET/PUT/DELETE bucket ACL
* GET/PUT/DELETE bucket versioning status
* GET/PUT/DELETE bucket logging
* GET bucket location (Walrus can handle the messages, but Walrus is single-region)

Object:
* GET/PUT/DELETE object (including object-copy)
* Object versioning
* Object ACLs

## Unsupported (yet) operations
* Bucket policies
* Bucket/object lifecycles
* Multipart-uploads
* Delete Multiple Objects (POST /?delete, not DELETE)
* Storage durability classes/Reduced Redundancy Storage
* Website hosting
* CORS support
* OPTIONS on object
* Bucket request payment
* Bucket tagging
* Bucket notification (for ReducedRedundancy lost objects, not applicable to Walrus yet)

Many of these features are in development now and this list will be updated as support is added.

[[category.storage]]