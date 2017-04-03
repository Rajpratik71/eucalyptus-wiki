## Introduction

The AWS Java SDK is Amazon's official Java library for AWS services. The Java SDK works with Eucalyptus as well, for those services that Eucalyptus supports.  For more information on using the AWS Java SDK, see the [official site](http://aws.amazon.com/sdkforjava/).

## Installation

Download and install the AWS Java SDK according to the instructions on the [AWS website](http://aws.amazon.com/sdkforjava/).

## Configuration and Usage

When using the AWS Java SDK with Eucalyptus, the endpoints must be correctly defined in your code. A simple example of using the EC2 API of your Eucalyptus cloud:

```java
  AmazonEC2 ec2 = new AmazonEC2Client( new BasicAWSCredentials(
    // provide access key and secret key
    "AKIEXAMPLEEXAMPLE",
    "SECRETKEYHERESECRETKEYHERESECRETKEYHERE"
  ) );

  // change to the IP and port of your Eucalyptus CLC
  ec2.setEndpoint( "http://192.168.1.100:8773/services/Eucalyptus/" );
```

An example for S3:

```java
  AmazonS3 s3 = new AmazonS3Client( new BasicAWSCredentials(
    // provide access key and secret key
    "AKIEXAMPLEEXAMPLE",
    "SECRETKEYHERESECRETKEYHERESECRETKEYHERE"
  )  );

  // change to the IP and port of your Eucalyptus CLC
  s3.setEndpoint( "http://192.168.1.100:8773/services/Walrus/" );

  // configure path-style S3 access if desired
  s3.setS3ClientOptions( new S3ClientOptions().withPathStyleAccess( true ) );
```

If using an IP address in the endpoint URL then path-style access will be automatically used by the AWS Java SDK. If using a host name then path-style access must be enabled using the client options if desired. When not using path-style access DNS resolution for bucket names must be available. For details on setting up DNS, consult the [Eucalyptus Installation Guide](http://www.eucalyptus.com/docs/eucalyptus/latest/install-guide/setting_up_dns.html).

In addition to EC2 and S3 the AWS Java SDK can be used against the IAM, STS , Auto Scaling, CloudWatch and ELB services in Eucalyptus.

## Version Compatibility

The below versions of the AWS Java SDK are recommended for each Eucalyptus version:

| With Eucalytpus | use SDK version | for EC2 API |
|-----------------|-----------------|-------------|
| 3.3.0           | latest          | 2013-02-01  |
| 3.2.(0,1,2)     | 1.3.23          | 2012-10-01  |
| 3.1.(0,1,2)     | 1.3.10          | 2012-05-01  |

## Questions/Issues

If you have any questions about using the Java SDK with Eucalyptus, please first check the [knowledgebase](https://engage.eucalyptus.com/customer/portal/articles/search?q=Java%20SDK).  

If you do not find an answer in the knowledgebase, please [ask a question](https://engage.eucalyptus.com/customer/portal/questions/new?q=Java%20SDK).

***
[[category.HOWTO]] 
[[category.tools]] 
[[category.aws-compatibility]]