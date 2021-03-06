Information and artifacts to be used in testing and giving demos of CORS.

See also the parent page [[CORS|CORS]]

* [Automated Tests](#automated-tests)
    * [nephoria](#nephoria)
    * [n4j](#n4j)
    * [ant](#ant)
* [Recorded Demos](#recorded-demos)
* [Test or Demo Procedures](#test-or-demo-procedures)
  * [AWS demos/tests](#aws-demos/tests)
  * [Eucalyptus demos/tests](#eucalyptus-demos/tests)
    * [CORS configuration management demo/test](#cors-configuration-management-demo/test)
    * [Preflight OPTIONS requests demo/test](#preflight-options-requests-demo/test)
    * [Cross-Origin accesses demo/test](#cross-origin-accesses-demo/test)
    * [Web browser based demo/test](#web-browser-based-demo/test)
* [Testing Notes](#testing-notes)



# Automated Tests

### nephoria
The python-based CORS tests are here:

[https://github.com/eucalyptus/nephoria/blob/master/nephoria/testcases/s3/cors_tests.py](https://github.com/eucalyptus/nephoria/blob/master/nephoria/testcases/s3/cors_tests.py)

These tests cover:


* config management: setting, getting, validating, and deleting the CORS config on a bucket.
* preflight requests: sending HTTP OPTIONS requests and validating the HTTP responses.


### n4j
The Java-based CORS tests are here:

[https://github.com/eucalyptus/n4j/blob/master/src/main/java/com/eucalyptus/tests/awssdk/S3CorsTests.java](https://github.com/eucalyptus/n4j/blob/master/src/main/java/com/eucalyptus/tests/awssdk/S3CorsTests.java)

These tests cover:


* config management: setting, getting, validating, and deleting the CORS config on a bucket.
* cross-origin requests: sending HTTP S3 requests and validating the CORS HTTP headers (if any) returned in the responses.


### ant
There are no ant-based unit tests at this time.


# Recorded Demos
Recorded video demos of CORS can be found here.

For Eucalyptus 4.4:

[[Eucalyptus 4.4 Sprint 2|Eucalyptus-4.4-Sprint-2]]

There is one by Lincoln on the CORS back-end and HTTP service, and one by Kamal on the CORS front-end of the eucaconsole. Lincoln's demo is similar to the demo for 4.3 Sprint 5 (below), but it does  _not_  cover the CLI-based demo shown in 4.3 Sprint 4 (below).



For Eucalyptus 4.3:

[https://hpenterprise.sharepoint.com/teams/euca](https://hpenterprise.sharepoint.com/teams/euca)

Select Documents (on the left) … Eucalyptus Demos … 4.3 Sprint Demos, Sprint 4 & 5 Yellow/Gold team

 **Sprint 4:**  Preflight requests, against AWS and Euca, with s3curl (CLI-based)

 **Sprint 5:**  Web browser-based demo of preflight & cross-origin requests. Note: The Sprint 5 video is ~15 secs ahead of the audio!


# Test or Demo Procedures

## AWS demos/tests
To understand how AWS responds to various CORS requests, and to define the expected results for the Eucalyptus implementation, here is the document containing the set of HTTP requests and responses used during CORS development.

The same document in PDF or editable Word doc format: (originally from Microsoft OneNote 2013)






## Eucalyptus demos/tests
For any of the following, you'll need to install the Eucalyptus-specific version of s3curl on that client machine. Refer to the [[S3Curl Wiki page|S3Curl]] for details.

For all of the following except the Web browser demo/test, use this CORS configuration file:

CORS test configuration (XML)


### CORS configuration management demo/test
For the v4.3 Sprint 2 demo (not recorded), the following process shows how you can put, get, and delete a CORS configuration on a bucket, using s3curl CLI commands from a client external to the cloud. It also shows what a configuration looks like in the Eucalyptus postgres database.

CORS configuration management demo (txt)


### Preflight OPTIONS requests demo/test
For the v4.3 Sprint 4 demo, the following script issues CORS HTTP OPTIONS (preflight) requests and shows the responses, again using s3curl.

CORS preflight requests demo (bash script)


### Cross-Origin accesses demo/test
For feature testing, the following script performs a set of bucket and object operations as cross-origin requests and shows the responses, again using s3curl.

CORS cross-origin requests demo (bash script)


### Web browser based demo/test
For the v4.3 Sprint 5 and v4.4 Sprint 2 demos, the following process was used. You can follow this to see how CORS works in Eucalyptus or give your own demo.

The same document in PDF or editable Word doc format: (originally from Microsoft OneNote 2013)





You'll need to untar these files onto your Web server that hosts the html and JavaScript page:

CORS Web server files for demo (tar)

You'll need to untar these files onto the client machine where you'll issue s3curl CLI commands to the Eucalyptus UFS host:

CORS CLI client files for demo (tar)


# Testing Notes

POSTing an HTML form, from a Web browser to an S3 bucket using CORS. The bucket ACL must be set to allow AllUsers WRITE ability, since browser access is "anonymous" (no S3 Authorization header with an account)

Test actions that cause a preflight request before the cross-origin request, and those that just do a cross-origin request. If the client request has only these headers, no preflight will be sent: (if other headers, preflight will be sent)
    * Accept 
    * Accept-Language 
    * Content-Language 
    * Content-Type, but only if the value is one of the following: 
    * application/x-form-urlencoded 
    * multipart/form-data 
    * text/plain 

To send an anonymous request to S3, issue a regular authorized request using e.g. s3curl --debug ..., then issue the equivalent curl command it shows, without the Authorization header.

For eucaconsole to upload images/snapshots (EBS) to S3 URL, but to get around Web browser's single-origin policy, it buffers object onto console UFS host, then tells UFS to upload to OSG, bypassing the browser. CORS removes buffering need and allows browser to upload to S3 bucket directly.

CORS headers can be returned in the HTTP response for any S3 operation, even those that would not be expected for CORS usage. 

Test CORS config validation error checking more: missing origin or method in a rule, >100 rules, rule IDs that are non-unique or >255 chars, negative MaxAgeSeconds.

Test using [s3cmd](http://s3tools.org/s3cmd). Can use --add-header="Origin:http://www.example.com". 


*****

[[category.storage]] 
[[category.confluence]] 
[[category.cors]]
[[category.objectstorage]]
