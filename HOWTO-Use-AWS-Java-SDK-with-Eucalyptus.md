## Introduction

The AWS Java SDK is Amazon's official Java library for AWS services. The Java SDK works with Eucalyptus as well, for those services that Eucalyptus supports.  For more information on using the AWS Java SDK, see the [official site](http://aws.amazon.com/sdkforjava/).

## Installation

Download and install the AWS Java SDK according to the instructions on the [AWS website](http://aws.amazon.com/sdkforjava/).

## Configuration and Usage

When using the AWS Java SDK with Eucalyptus, the endpoints must be correctly defined in your code. A simple example of authentication to your Eucalyptus cloud:

```
 private AWSCredentials credentials() {
    // provide access key and secret key
    return new BasicAWSCredentials(
        "XXXXXXXXXXXXXXXXXXXXX",
        "YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY"
    );
  }

  // change to the IP and port of your Eucalyptus CLC
  private String cloudUri( String servicePath ) {
    return
        URI.create( "http://192.168.1.100:8773/" )
            .resolve( servicePath )
            .toString();
  }

  private AmazonEC2 getEc2Client( ) {
    final AmazonEC2 ec2 = new AmazonEC2Client( credentials() );
    ec2.setEndpoint( cloudUri( "/services/Eucalyptus/" ) );
    return ec2;
  }

  final AmazonEC2 ec2 = getEc2Client();
```

## Known Issues

Per the S3 spec, Walrus must be set up to use DNS in order for API calls to work correctly. For details, consult the [Eucalyptus Installation Guide](http://www.eucalyptus.com/docs/eucalyptus/latest/install-guide/setting_up_dns.html).

## Questions/Issues

If you have any questions about using the Java SDK with Eucalyptus, please first check the [knowledgebase](https://engage.eucalyptus.com/customer/portal/articles/search?q=Java%20SDK).  

If you do not find an answer in the knowledgebase, please [ask a question](https://engage.eucalyptus.com/customer/portal/questions/new?q=Java%20SDK).

***
[[category.HOWTO]]