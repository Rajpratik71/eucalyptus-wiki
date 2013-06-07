## Introduction
jclouds allows provisioning and control of cloud resources, including blobstore and compute, from Java and Clojure. The jclouds API allows developers to use both portable abstractions and cloud-specific features.

This wiki page documents how you can use jclouds to provision and control cloud resources with Eucalyptus.

## Prerequisites
For EC2 the following jclouds modules are required:

* ec2 (org.jclouds.api)
* jclouds-compute (org.jclouds)
* jclouds-core (org.jclouds)
* jclouds-scriptbuilder (org.jclouds)
* sts-1.6.0 (org.jclouds.api)

You may also want a logging module, such as one of:

* jclouds-log4j (org.jclouds.driver)
* jclouds-slf4j (org.jclouds.driver)

The Maven "groupId" for each module is shown in brackets.

If you do not have local resources to deploy Eucalyptus, you may request a free account on the [http://www.eucalyptus.com/eucalyptus-cloud/community-cloud Eucalyptus Community Cloud].

## Configuration for Eucalyptus
This section describes how to configure jclouds to communicate with your Eucalyptus cloud.

```java
final Iterable<Module> modules = ImmutableSet.<Module>of( new Log4JLoggingModule( ) );
final Properties properties = new Properties();
properties.setProperty( Constants.PROPERTY_ENDPOINT, "http://communitycloud.eucalyptus.com:8773/services/Eucalyptus/" );
final ContextBuilder builder ContextBuilder.newBuilder( "ec2" )
    .credentials( accessKeyIdentifier, secretKey )
    .modules( modules )
    .overrides( properties );
final ComputeService compute = builder.buildView( ComputeServiceContext.class ).getComputeService();
final EC2Client ec2Client = builder.<RestContext<EC2Client, EC2AsyncClient>>build( ).getApi();
```

## Latest Testing
Tests for jclouds are being added to Eutester4J.

We currently have tests for the following capabilities:

* ComputeService
    * Listing images 
    * Listing instances
* EC2Client
    * Describing elastic ips
    * Describing key pairs
    * Describing security groups
    * Describing snapshots
    * Describing volumes

Versions used:

* jclouds 1.6.0-rc.1
* Eucalyptus: 3.3.0

~~~~~~~~~~~~~~~~~~
[[category.HOWTO]]
[[category.aws-compatibility]]