Eucalyptus strives for 100% API compatibility with all currently supported AWS services. Our goal is to be a "drop-in replacement AWS region".  We currently recommend the following open source tools for use with both AWS and Eucalyptus.

## Recommended tools

_**[AWS Java SDK](https://github.com/eucalyptus/eucalyptus/wiki/HOWTO-Use-AWS-Java-SDK-with-Eucalyptus)**_ is Amazon's official Java library for AWS services.  The Java SDK works with Eucalyptus as well with minor configuration changes. 

**_[AWS Ruby SDK](https://github.com/eucalyptus/eucalyptus/wiki/HOWTO-Use-AWS-Ruby-SDK-with-Eucalyptus)_** is Amazon's official Ruby library for AWS services. The Ruby SDK works with Eucalyptus as well, with some minor patches, for those services that Eucalyptus supports. 

**_[s3cmd](https://github.com/eucalyptus/eucalyptus/wiki/HowTo-use-s3cmd-with-Eucalyptus)_** is the preferred command-line tool for working with files in AWS S3 and Eucalyptus Walrus.

**_[[AWS Toolkit for Eclipse]]_** is a plug-in for the Eclipse Java IDE that makes it easier for developers to develop, debug, and deploy Java applications using Amazon Web Services and Eucalyptus.

**_[Fog](https://github.com/eucalyptus/eucalyptus/wiki/HOWTO-Use-Fog-with-Eucalyptus)_** is a popular Ruby library for management of various cloud resources.

**_[[jclouds]]_** allows provisioning and control of cloud resources, including blobstore and compute, from Java and Clojure. The jclouds API allows developers to use both portable abstractions and cloud-specific features.

**_[EucaLobo](http://testingclouds.wordpress.com/2013/06/18/getting-started-with-eucalobo/)_** is a client-side application for managing AWS and Eucalyptus cloud resources with an easy-to-use graphical user interface.

## Other tools

**_[s3curl](http://www.eucalyptus.com/eucalyptus-cloud/tools/s3curl)_** is a tool that allows users to interact with Eucalyptus Walrus and AWS S3.

**_[Vagrant AWS Plug-in](https://github.com/mitchellh/vagrant-aws)_**.  Vagrant is a tool for managing virtual machines on a local system, and also provides a plug-in that allows users to manage their AWS instances with Vagrant configuration files.  The plug-in also works for Eucalyptus.

**_[Cyberduck](http://cyberduck.ch/)_** is an FTP, SFTP, WebDAV & cloud storage browser for Mac & Windows.

**_[[Netflix Edda]]_** is a service that polls your AWS resources via AWS APIs and records the results. It allows you to quickly search through your resources and shows you how they have changed over time.

**_[[AWS SDK for PHP]]_** is Amazon's official PHP library for AWS services. Testing with Eucalyptus is not yet complete.

* _**AWS SDK for Java**_ 
 * Project URL: http://aws.amazon.com/sdkforjava/
 * Versions tested: [AWS JAVA SDK 1.4.1, Euca 3.3 m5]
 * Testplan: https://github.com/eucalyptus/eucalyptus/wiki/Aws-SDK-for-Java-Testplan
 * Driver: **[[Tony Beckham]]**
 * Known bugs: [EUCA-5515](https://eucalyptus.atlassian.net/browse/EUCA-5515) but reported fixed in 3.3 testing branch
 * Notes: known issues with Walrus should be fixed as of 3.3m4, needs retesting
 * Last update: 09 April 2013

* _**Netflix Asgard**_
 * Project URL: FIXME
 * Driver: [[grze]]
 * Known bugs: FIXME
 * Notes: demoed at Netflix, 13 Mar 2013.
 * Last update: 13 Mar 2013

* _**Netflix Chaos Monkey**_
 * Project URL: FIXME
 * Driver: [[grze]]
 * Known bugs: FIXME
 * Notes: demoed at Netflix, 13 Mar 2013.
 * Last update: 13 Mar 2013

* _**AWS CLI**_
 * Project URL: http://aws.amazon.com/cli/
 * Driver: Andy Grimm
 * Known bugs: FIXME
 * Notes: http://agrimmsreality.blogspot.com/2013/01/using-aws-cli-with-eucalyptus.html
 * Last update: 3 Jan 2013

* _**Jenkins for EC2**_
 * Project URL: https://wiki.jenkins-ci.org/display/JENKINS/Amazon+EC2+Plugin
 * Driver: [[Lester Wade]]
 * Known bugs: FIXME
 * Notes: Version 1.17 of the EC2 plugin does not work. Jenkins tries to get endpoints via the register URL which fails unless endpoint is https://cloudIP:8443/register. Once past this point, connectivity test yields "login failure: all modules ignored". Version 1.14 works however.
 * Last update: FIXME

*****

[[category.aws-compatibility]]