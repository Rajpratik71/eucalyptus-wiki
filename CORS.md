The landing page for all things CORS (Cross-Origin Resource Sharing).

  * [Eucalyptus](#eucalyptus)
  * [Behavior observations](#behavior-observations)
    * [Preflight OPTIONS Requests](#preflight-options-requests)
    * [Cross-Origin Requests](#cross-origin-requests)
    * [Headers returned: Conflict between AWS doc and behavior](#headers-returned:-conflict-between-aws-doc-and-behavior)
    * [Allowed Methods: Conflict between AWS doc & behavior](#allowed-methods:-conflict-between-aws-doc-&-behavior)
    * [Origin Port: Conflict between AWS and IETF spec](#origin-port:-conflict-between-aws-and-ietf-spec)
* [Design and Implementation](#design-and-implementation)
  * [Data flow summary](#data-flow-summary)
    * [PUT CORS Configuration](#put-cors-configuration)
    * [GET CORS Configuration](#get-cors-configuration)
* [Development To-Do's](#development-to-do's)
* [Testing](#testing)


Feature InformationGeneralAn excellent book "CORS In Action" by Monsur Hossain explains how clients and Web browsers use CORS, and how to implement CORS on the server side. It's available here, among other places:

[https://www.safaribooksonline.com/library/view/cors-in-action/9781617291821/](https://www.safaribooksonline.com/library/view/cors-in-action/9781617291821/) (HPE employees have free site-wide license access)

[https://www.amazon.com/CORS-Action-Creating-consuming-cross-origin/dp/161729182X](https://www.amazon.com/CORS-Action-Creating-consuming-cross-origin/dp/161729182X)

The CORS W3 specification can be found here: [Cross-Origin Resource Sharing](http://www.w3.org/TR/cors/)

A short overview of CORS can be found in the [Cross-origin resource sharing Wikipedia](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) page.



AmazonThe CORS implementation in AWS is explained in the [Amazon Simple Storage Service Developer Guide](http://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html). From that guide: "Cross-origin resource sharing (CORS) defines a way for client web applications that are loaded in one domain to interact with resources in a different domain. With CORS support in Amazon S3, you can build rich client-side web applications with Amazon S3 and selectively allow cross-origin access to your Amazon S3 resources."

The details of the CORS portions of the Amazon S3 API can be found in the [Amazon S3 REST API](http://docs.aws.amazon.com/AmazonS3/latest/API/Welcome.html) documentation.

Specifically, see the [PUT Bucket CORS](http://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketPUTcors.html), [GET Bucket CORS](http://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketGETcors.html), and the [OPTIONS (Preflight)](http://docs.aws.amazon.com/AmazonS3/latest/API/RESTOPTIONSobject.html) pages.


## Eucalyptus
We support the S3 CORS feature in Eucalyptus. It is nearly fully compatible with the AWS CORS feature. See "Compatibility Conflicts" below.



Top-level: [[Feature Details: CORS Configuration for S3 Buckets|CORS-Configuration-for-S3-Buckets]]

Architecture page:[ Architecture for CORS in Eucalyptus](http://docs.aws.amazon.com/AmazonS3/latest/dev/cors.html)


## Behavior observations
Some interesting tidbits about how CORS works:


### Preflight OPTIONS Requests
Resource target
* The target can be either a bucket or an object. 
* The object in the request doesn't have to exist. 
* Responses are the same regardless. It's a response based just on the bucket's CORS config.

Preflight requests with bad/missing required headers
* Origin and Method header validations performed before bucket lookup.
* Same output whether any CORS config exists or not. 



Handling of AllowedOrigin of '\*'When a preflight request matches a rule with Allowed Origin of '\*', as in:




```xml
  <CORSRule>

    <ID>Any origin can HEAD</ID>

    <AllowedOrigin>*</AllowedOrigin>

    <AllowedMethod>HEAD</AllowedMethod>

  </CORSRule>

```


The undocumented AWS behavior is that if the only AllowedOrigin is '\*' (allow any origin), then the resource does not allow credentials. Therefore:


* No "Access-Control-Allow-Credentials: true" header
* The "Access-Control-Allow-Origin:" header shows '\*' instead of the origin requested.

This matches W3 spec:

From [https://www.w3.org/TR/cors/#resource-preflight-requests](https://www.w3.org/TR/cors/#resource-preflight-requests):

 **6.2 Preflight Request** 

If the resource [supports credentials](https://www.w3.org/TR/cors/#supports-credentials) add a single [Access-Control-Allow-Origin](https://www.w3.org/TR/cors/#http-access-control-allow-origin) header, with the value of the [Origin](https://www.w3.org/TR/cors/#http-origin) header as value, and add a single [Access-Control-Allow-Credentials](https://www.w3.org/TR/cors/#http-access-control-allow-credentials) header with the [case-sensitive](https://www.w3.org/TR/cors/#case-sensitive) string "true" as value.

 Otherwise, add a single [Access-Control-Allow-Origin](https://www.w3.org/TR/cors/#http-access-control-allow-origin) header, with either the value of the [Origin](https://www.w3.org/TR/cors/#http-origin) header or the string "\*" as value.

 The string "\*" cannot be used for a resource that [supports credentials](https://www.w3.org/TR/cors/#supports-credentials).


### Cross-Origin Requests

* If no Origin request header, nothing is different in the response.
* If there is an Origin header, but no rule matches the given Origin and HTTP verb, nothing is different in the response.
* If there is an Origin header, and any rule matches the given Origin and HTTP verb, the same response headers are returned as the ones returned by an OPTIONS request with the same Origin and Access-Control-Request-Method. However:
    * Request headers are not checked against the Allowed Headers when matching against CORS rules.
    * The Access-Control-Allow-Headers response header is never returned in the response.

    


* The contents of a GET response do not change for cross-origin requests! If requesting an object, and no CORS rule matches the given Origin, the contents of the object are still returned (if authorized), but they won't contain any of the extra response headers. This lack of headers tells the browser to deny the client's request, so the client will never see the object's contents.


* 
    * If a browser does a preflight request for that GET, before issuing the GET request, if no CORS rule matches, then the response to the preflight will tell the browser to deny the client's request without ever sending the actual GET request.
    * But, browsers don't send preflight requests for GET, POST, or HEAD unless the Content-Type is not one of \[application/x-www-form-urlencoded, multipart/form-data, text/plain] or if there's any additional request headers besides \[Accept, Accept-Language, Content-Language]. (See "CORS in Action" book.)

    
* The actions taken by HEAD, PUT, POST, and DELETE do not differ whether the request is a cross-origin request or not. The response headers are also the same except for the extra CORS response headers when a rule is matched.
* The presence of the CORS extra response headers depends only on the bucket in the URL. The object can exist or not, and can by missing (a bucket-only resource URL)

Compatibility Conflicts
### Headers returned: Conflict between AWS doc and behavior
OPTIONS responses from AWS differs from [http://docs.aws.amazon.com/AmazonS3/latest/API/RESTOPTIONSobject.html](http://docs.aws.amazon.com/AmazonS3/latest/API/RESTOPTIONSobject.html):


1. The Etag header in the doc is not in the actual response.
1. The Access-Control-Allow-Credentials and Vary actual headers are not in the doc.

Match AWS actual responses for behavioral compatibility.


### Allowed Methods: Conflict between AWS doc & behavior
In [http://docs.aws.amazon.com/AmazonS3/latest/API/RESTOPTIONSobject.html](http://docs.aws.amazon.com/AmazonS3/latest/API/RESTOPTIONSobject.html) :





| Access-Control-Allow-Methods | The HTTP method that was sent in the original request. | 

Actual behavior shows  _all methods_  in rule, not just the one in the request.

Either way is compliant with w3 spec:

From [https://www.w3.org/TR/cors/#access-control-allow-methods-response-header](https://www.w3.org/TR/cors/#access-control-allow-methods-response-header):

 **5.5 Access-Control-Allow-Methods Response Header** 

The Access-Control-Allow-Methods header indicates, as part of the response to a [preflight request](https://www.w3.org/TR/cors/#preflight-request), which methods can be used during the [actual request](https://www.w3.org/TR/cors/#actual-request).

From [https://www.w3.org/TR/cors/#resource-preflight-requests](https://www.w3.org/TR/cors/#resource-preflight-requests):

 **6.2 Preflight Request** 

Add one or more [Access-Control-Allow-Methods](https://www.w3.org/TR/cors/#http-access-control-allow-methods) headers consisting of (a subset of) the [list of methods](https://www.w3.org/TR/cors/#list-of-methods).

 Since the [list of methods](https://www.w3.org/TR/cors/#list-of-methods) can be unbounded, simply returning the method indicated by [Access-Control-Request-Method](https://www.w3.org/TR/cors/#http-access-control-request-method) (if supported) can be enough.

Use AWS behavior (show all methods) which is OK by W3, for AWS behavioral compatibility.


### Origin Port: Conflict between AWS and IETF spec
 **AWS says:** 

From [http://docs.aws.amazon.com/AmazonS3/latest/dev/cors-troubleshooting.html](http://docs.aws.amazon.com/AmazonS3/latest/dev/cors-troubleshooting.html):

"The scheme, the host, and the port values in the Origin request header must match the AllowedOrigin in the CORSRule. For example, if you set the CORSRule to allow the origin [http://www.example.com](http://www.example.com), then both [https://www.example.com](https://www.example.com) and [http://www.example.com:80](http://www.example.com:80) origins in your request do not match the allowed origin in your configuration."

 **IETF says:** 

From [http://tools.ietf.org/html/rfc6454](http://tools.ietf.org/html/rfc6454):

"All of the following resources have the same origin:

[http://example.com/](http://example.com/)

[http://example.com:80/](http://example.com:80/)

[http://example.com/path/file](http://example.com/path/file)

 Each of the URIs has the same scheme, host, and port components."



AWS works as described in AWS doc: With no port, e.g. 'Origin: [http://www.example1.com'](http://www.example1.com%27) matches the Allowed Origin of [http://example.com/](http://example.com/).

With port 80 defined, e.g. 'Origin: [http://www.example1.com:80](http://www.example1.com:80)' does  _not_  match the Allowed Origin of [http://example.com/](http://example.com/). You get the error "This CORS request is not allowed." in the response body.

Follow AWS instead of IETF, for AWS compatibility, and it's more restrictive (safer).

Furthermore, AWS's way makes more sense! AWS just gets the Origin as a string. It never connects to that Origin. So it has no way of knowing if [www.example1.com](http://www.example1.com) is listening on port 80 or some other port, so it can't know if [www.example1.com](http://www.example1.com) and [www.example1.com:80](https://eucalyptus.atlassian.net/wiki/www.example1.com:80) are the same listening server and thus the same Origin.


# Design and Implementation

## Data flow summary

### PUT CORS Configuration
1. PUT uploads XML CORS config to Euca

2. Pipeline routes it to ObjectStorageRESTBinding.java

3. Convert from XML into DOM (SAX)

4. Convert from DOM into Java object structure (Groovy)

5. ObjectStorageGateway.java validates certain object data

6. DbBucketCorsManagerImpl.java converts String arrays to JSON strings

7. Convert objects (incl JSON strings) into DB entity of rows and columns

8. Replace existing Hibernate DB table with new entity


### GET CORS Configuration
1. GET routes through pipeline

2. Query the DB

3. Parse into Java object structure

4. Convert objects into XML body of response (JiBX)


# Development To-Do's
In priority order, highest first.

64incompleteAdd to body of Preflight responses, only when error status codes are returned:<Method>PUT</Method><ResourceType>OBJECT</ResourceType>

Insert whatever the Access-Control-Request-Method in the request is, "PUT" in this example. Insert either BUCKET or OBJECT (this example) depending on the resource accessed. This is to be compatible with AWS, not shown in any CORS spec.

65incompleteAfter anonymous (represented by the "nobody" fake account) creates an object in a bucket with a bucket ACL that allows AllUsers WRITE permission, checking the object's ACL with e.g. /services/objectstorage/bucket1/byanon1.txt?aclshows theDisplayName "nobody" in the Owner stanza, butDisplayName is blank in theGrantee section, for the same ID. No effect on behavior other than response consumers may not be able to show the proper grantee of an ACL to users in human-readable outputs. The ID is correct, and programmatic uses of it will be fine. (Not CORS-specific, but we added the ability in [EUCA-12174](https://eucalyptus.atlassian.net/browse/EUCA-12174) for anonymous to create objects for CORS usage.)66incompleteInvestigate: According to the "[CORS In Action](https://www.safaribooksonline.com/library/view/cors-in-action/9781617291821/)" book, Chapter 6 "Best Practices": Redirects cause the browser to re-send the request to the redirected place with the Origin set to "null", (and Referer set to the original URL e.g. [http://localhost:1111/client.html](http://localhost:1111/client.html)) which should cause the server to respond with good CORS headers! We don't. AWS doesn't either, unless I just couldn't give it a Referer that they accept. I don't know their criteria, tried some random site URLs and even [https://s3.amazonaws.com](https://s3.amazonaws.com) but never got CORS headers in the responses.79incompleteCan a Euca bucket host a Web HTML file (w/ JavaScript) to serve up to a Web browser? Can a browser point to the OSG's endpoint / bucket / object directly to get the HTML file? (Not cross-origin) If so, it could then also make cross-origin from that HTML page to an object in another bucket (owned by another user/account)? Set up default Web site in bucket w/ wide-open perms80incompleteUse [https://github.com/eucalyptus/eucalyptopotamus](https://github.com/eucalyptus/eucalyptopotamus) as a sample Web server?81incompleteOn exceptions/errors, CORS headers are still returned in AWS responses. Eucalyptus doesn't include the CORS headers.Non-trivial, would change a lot of code. Is it really needed?

Test from browsers: does their behavior differ between AWS and Eucalyptus if CORS headers exist or not, in those error cases? Doubtful. But if so, we might need to add those CORS headers.

82incompleteDelay seen between when setting a CORS config returns to the caller, and when it's available for a Get. Currently a 10 sec sleep in n4j S3CorsTests.java, investigate that more. Is it only when testing against AWS or Euca or both?

83incompleteUse more @Nonnull annotations where appropriate, e.g. in DbBucketCorsManagerImpl.java 



68incompleteInvestigate: Does AWS support the HTTP PATCH method for CORS? Not documented. If so, should we? (discussed in the "[CORS In Action](https://www.safaribooksonline.com/library/view/cors-in-action/9781617291821/)" book).



69incompleteAdd javadocs



70incompleteRefactoring CORS DB-related code:



71incompleteBucketCorsManager.java –deleteCorsRules(String bucketUuid, TransactionResource tran) refers toTransactionResource which is a database construct. It should probably be local to the implementation DbBucketCorsManagerImpl and could even be a private method since it’s not invoked by users of the BucketCorsManager API. BucketLifecycleManager.java has a similar issue.



72incompleteDbBucketCorsManagerImpl.java – Try using google collections and transform (Lists.transform() or Iterables.trasform() methods) to create a list of things from another list of things



73incompleteDbBucketCorsManagerImpl.java line 111, 168 – avoid deprecated methods unless there is no way around them, such as Entities.query(). Transactions.java offers a rich API for database operations. For instance, to look up and return a unique value, use Transactions.one(example).You don’t have to open or close the transaction since it’s all handled in Transactions.java. For deleting based on a check, use Transactions.deleteAll(@Nonnull final EntityRestriction<T> restriction,@Nonnull Predicate<? super T> precondition). Define the check using Google predicate. Bucket Lifecycle code has the same issues.

74incompleteAdd unit tests, to e.g. ObjectStorageGatewayTest.java. But current set of nephoria, n4j, and test scripts are fairly comprehensive functional tests.



75incompleteAdd CorsConfig boolean to Bucket table in DB, true= CORS config exists. That way, if no config exists, and an Origin header does exist, we avoid a DB lookup on the CORS table. But a request to a bucket that expects a CORS config when a config doesn't exist should be a rare case.



76incompleteRefactoring: S3BucketOperations.groovy–S3 facing entities such as CorsRule and CorsCofniguration could be defined in S3BaseModel.groovy. Not sure why S3BucketOperations.groovy even exists. File has a comment saying not used by anything yet except for lifecycle stuff, should move the lifecycleconstructs to S3BaseModel.groovy as well.

This seems to be a placeholder for the unsupported (yet) operations. But there are other things that are “real” too, like DeleteMultipleObjects. Could move all the “real” stuff to S3BaseModel.groovy and rename this file to e.g. S3UnsupportedOperations.groovy. Could comment out everything too, no need to compile it.

77incompleteRefactoring: Instead of changing every outbound handler, could create a new handler for CORS processing and add it to the outbound stages.

Turns out to be not much CORS code in the handlers, works correctly now.

78incompleteRefactoring: Format code using google style guide and remove unused imports. Both these actions can be combined with save event in Eclipse -[http://goo.gl/YD9X7E](http://goo.gl/YD9X7E). Google style guide -[https://goo.gl/i2A5EL](https://goo.gl/i2A5EL), copy the xml content to a file and import it in eclipse. Set the line limit to 150. Use it for both Java and Groovy files.


# Testing
See the [[CORS Testing page|CORS-Testing]].





*****

[[category.storage]] 
[[category.confluence]] 
