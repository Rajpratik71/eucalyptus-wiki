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

**_[Cloudberry S3 Explorer](http://www.cloudberrylab.com/)_** is a Windows tool that makes managing files in Amazon S3 and Eucalyptus Walrus easy.

**_[s3fs](http://code.google.com/p/s3fs/)_** is a FUSE filesystem that allows you to mount a bucket from Amazon S3 or Eucalyptus Walrus as a local filesystem.

**_[Vagrant AWS Plug-in](https://github.com/mitchellh/vagrant-aws)_**.  Vagrant is a tool for managing virtual machines on a local system, and also provides a plug-in that allows users to manage their AWS instances with Vagrant configuration files.  The plug-in also works for Eucalyptus.

**_[Cyberduck](http://cyberduck.ch/)_** is an FTP, SFTP, WebDAV & cloud storage browser for Mac & Windows.

**_[[Netflix Edda]]_** is a service that polls your AWS resources via AWS APIs and records the results. It allows you to quickly search through your resources and shows you how they have changed over time.

**_[[AWS SDK for PHP]]_** is Amazon's official PHP library for AWS services. Testing with Eucalyptus is not yet complete.

**_[AWS CLI](https://github.com/eucalyptus/eucalyptus/wiki/AWS-CLI)_** is a new AWS tool for controlling multiple AWS and Eucalyptus services from the command line and automating them through scripts.

*****

[[category.aws-compatibility]]