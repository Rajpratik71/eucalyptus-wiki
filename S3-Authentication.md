
# Overview
The Eucalyptus Object Storage service supports AWS S3 [version 2](http://docs.aws.amazon.com/AmazonS3/latest/dev/auth-request-sig-v2.html) and [version 4](http://docs.aws.amazon.com/AmazonS3/latest/API/sig-v4-authenticating-requests.html) authentication for REST and form/query parameter requests.

S3 authentication is separate from the rest of Eucalyptus for a few reasons:


* S3 authentication related failures return error messages and codes specific to S3.
* Request payload hashes are computed and handled differently than elsewhere in AWS, particularly for chunked payloads.
* Object expiration is validated in addition to request date.


# Handling
Authentication is handled via the object storage web services pipeline.


* Each request method has a distinct pipeline, such as ObjectStoragePutDataPipeline(.java).
* The first stage to be unrolled (and applied to request handling) in each pipeline is the ObjectStorageUserAuthenticationStage.
* ObjectStorageUserAuthenticationStage, upon being unrolled, adds an ObjectStorageAuthenticationHandler to the pipeline for each request.
* ObjectStorageAuthenticationHandler handles upstream requests by:

    
    * Removing headers with duplicate keys AND values
    * Joining header values for the same key
    * Selecting an S3Authenticator for the request and calling authenticate(request).

    


# Authenticating Requests
The general approach for authenticating requests is to extract information from the request (via headers or query parameters), construct a string to sign which will be compared to the signature in the request, then use JaaS to obtain the user's secret key and use that key to sign the constructed string with the string contained in the request to determine if the request is valid.


# S3Authentication
S3Authentication is the entry point for the object storage authentication code. S3Authentication contains S3Authenticator implementations for each type of request that we validate, version 2 REST and param requests, and version 4 REST and param requests. These implementations extract and validate request data in slightly different ways, and build up a "string to sign" which is then passed on to be authenticated by the ObjectStorageLoginModule. S3Authentication also provides version independent implementations of utility methods to aid in extracting and validating request information.

S3Authentication.login is responsible for attempting authentication, by creating a new ObjectStoragedWrappedCredentials instance for the request data, and calling LoginContext.login which is later handled by the ObjectStorageLoginModule via JaaS. The ObjectStorageWrappedCredentials instance initially contains a "string to sign" that is constructed without the request path. If the initial authentication attempt fails, the another ObjectStorageWrappedCredentials instance is constructed with a "string to sign" that includes the request path, and authentication is retried.


## S3V2Authentication
S3V2Authentication provides version 2 specific methods for extracting, validating and authenticating requests.


## S3V4Authentication
S3V4Authentication provides version 4 specific methods for extracting, validating and authenticating requests.


## ObjectStorageWrappedCredentials
Value object containing credentials and strings to sign for later validation by ObjectStorageLoginModule.


## ObjectStorageLoginModule
ObjectStorageLoginModule is registered as a JaaS login module with Euca on bootstrap via a jar scanner. When S3Authentication.login is called, which in turn calls the JaaS LoginContext.login(), ObjectStorageLoginModule is interrogated to see if it handles ObjectStorageWrappedCredentials data (via the accepts() method), and is then asked to authenticate the ObjectStorageWrappedCredentials instance.

From here, we call the appropriate authentication method for the signature version, 2 or 4, lookup a secret key for the user's access key id and security token, then use the secret key to sign the "string to sign" that we constructed from the request data and verify that it matched the signed string in the request. If so, authentication succeeds, else it fails (and may be retried).



*****

[[category.storage]] 
[[category.confluence]] 
