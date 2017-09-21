* [Overview](#overview)
* [Impact](#impact)
  * [Configuration](#configuration)
  * [Behaviour](#behaviour)
* [Design / Implementation](#design-/-implementation)
  * [Details](#details)
    * [Async Requests Connection Pooling](#async-requests-connection-pooling)
    * [AWS SDK for Java Connection Pooling](#aws-sdk-for-java-connection-pooling)
    * [HTTPS / AWS Signature V4 for Internal SOAP Pipeline](#https-/-aws-signature-v4-for-internal-soap-pipeline)
    * [HTTPS for AWS SDK for Java](#https-for-aws-sdk-for-java)
  * [Troubleshooting](#troubleshooting)
* [References](#references)

# Overview
This document covers changes to internal messaging for the 5.0 release. Internal messaging refers to messages been eucalytpus-cloud (Java) components on different hosts:


* System messages, e.g. bootstrap/empyrean
* Messages between ufs and backend for a particular service , e.g. autoscaling
* Messages between services using async requests, e.g. cloudformation → ec2
* Messages between services using the aws sdk for java, e.g. cloudformation → s3


# Impact

## Configuration
Properties are added to allow tuning or disabling the internal messaging changes.



| Property | Ticket | Default | Description | Comments | 
|  --- |  --- |  --- |  --- |  --- | 
| bootstrap.webservices.client_internal_connect_timeout_millis | EUCA-13251 | 3000 | Socket connection timeout | Previously not configurable | 
| bootstrap.webservices.client_http_pool_acquire_timeout | EUCA-13251 | 60000 | Pool checkout timeout |  | 
| bootstrap.webservices.client_message_log_whitelist | EUCA-13251 |  | Matches messages for logging | Similar to context message logging added in 4.4 | 
| bootstrap.webservices.client_internal_hmac_signature_enabled | EUCA-13292 | true | Enables hmac signatures | When false WS-Security is used | 
| bootstrap.webservices.ssl.client_https_enabled | EUCA-13293 | true | Enables https between hosts | For internal SOAP pipeline | 
| bootstrap.webservices.ssl.client_https_server_cert_verify | EUCA-13293 | true | Enables certificate checks | For internal SOAP pipeline when using https | 
| bootstrap.webservices.ssl.client_ssl_ciphers | EUCA-13293 | RSA+AES:+SHA:!EXPORT:!EXPORT1025:!MD5:!ECDHE | Ciphers to enable | Can be more restrictive than server ciphers | 
| bootstrap.webservices.ssl.client_ssl_protocols | EUCA-13293 | TLSv1.2 | Protocols to enable | Can be more restrictive than server protocols | 

To revert to pre 5.0 behaviour for message security you would set:


```bash
bootstrap.webservices.client_internal_hmac_signature_enabled=false
bootstrap.webservices.ssl.client_https_enabled=false
```

## Behaviour
Load on the system is reduced.

Message throughput will be improved where cpu usage (signing) was a bottleneck.

Higher number of established connections / connections stay open longer.


# Design / Implementation

## Details

### Async Requests Connection Pooling
Client use of Netty is upgraded to 4.x to make use of connection polling functionality. This meant some handlers are reimplemented using netty 4 apis, these are prefixed with "Io".

The  _AsyncRequestChannelPoolMap_ handles pooling and is used from  _AsyncRequestHandler_ .

We use netties _SimpleChannelPool_  /  _FixedChannelPool_  with our own _ChannelHealthChecker_ that implements pooling limits based on the number of requests handled and the amount of time a connection has been idle.


### AWS SDK for Java Connection Pooling
We now take advantage of the built it connection polling by sharing a  _AmazonHttpClient_ between all instances of service specific clients such as  _AmazonSimpleWorkflowClient._ 

In order to configure the shared client we extend the base sdk classes with our own, such as  _EucaSimpleWorkflowClient_  and  _EucaHttpClient_  for simpleworkflow.


### HTTPS / AWS Signature V4 for Internal SOAP Pipeline
The pipeline for internal soap is created in  _InternalClientChannelInitializer_ .

We use the netty  _SslHandler_  with an engine created via our  _SslSetup_  utility, when disabled the handler is not added to the pipeline.

For aws hmac signature v4 there is a new  _IoInternalHmacHandler._ Unlike the WS-Security handler this is placed after soap serialization as it does not modify the message body. The new handler makes use of the AWS SDK for Java  _AWS4Signer_ to sign request messages. Signing uses credentials from the token service which are cached for a few minutes. The caching is handled in the  _IoInternalHmacHandler_  so that we can avoid blocking concurrent credential access during a typical refresh (on expiry)


### HTTPS for AWS SDK for Java
Not implemented, we may want to do this in 5.0 for consistent use of https between java components.


## Troubleshooting
Lower level settings for async requests connection pooling can be modified on each host by setting system properties:


```java
com.eucalyptus.util.async.channelReuseMaxIdle=45000
com.eucalyptus.util.async.channelReuseMaxRequests=75
```
it is not expected that the above default values will generally need to be changed.


# References

* [EUCA-13251 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-13251)
* [EUCA-13252 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-13252)
* [EUCA-13253 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-13253)
* [EUCA-13292 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-13292)
* [EUCA-13293 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-13293)





*****

[[category.confluence]] 
