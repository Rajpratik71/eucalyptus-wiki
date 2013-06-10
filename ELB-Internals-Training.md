## Feature Overview
These training materials cover the internals of the ELB (Elastic Load Balancer) image. After reading through these, you should have a basic understanding of what components make up the ELB image, how it is installed and configured, and how to troubleshoot problems with an ELB instance.

#### Image Components
There are a handful of components that are necessary for an ELB image to function. An engineer could create their own ELB image on any Linux platform as long as the following essentials are included:

* [HAProxy](http://haproxy.1wt.eu/) version 1.5dev18
* [Load Balancer Servo](https://github.com/eucalyptus/load-balancer-servo) version 1.0.0
* [Python Boto](https://github.com/boto/boto) version 2.8.0

The project that is used to build the official ELB image is available [here](https://github.com/eucalyptus/load-balancer-image).

#### Difference between the _Production_ and _Development_ ELB image
When installing the ELB image, there are two different versions that are available. These are the _Production_ and _Development_ versions of the image. You will almost always want to install the _Production_ version of the ELB image. The difference between the two is that the _Production_ version of the image installs the released RPM package for the Servo service, while the _Development_ version grabs the latest code from the _master_ branch of the Github repository. The purpose of the latter is to facilitate development of the Servo code; it is targeted at developers who are troubleshooting a specific problem and would like to modify the source code to submit a patch.

#### User level operation and tooling
From a user perspective, there is nothing that they need to know about the internal workings of the ELB image or the Servo process that runs in the ELB instance. This is essentially a block box. The only way that users should ever interact with the image/instance is indirectly through the _eulb_ command line tools or through the AWS compatible APIs that Eucalyptus exposes.

## Administrative Tasks
Here we'll talk about the basic tasks for dealing with ELB images including installation and configuration.

### Installing the ELB image
In order to register an ELB image with your cloud, you first have to install the RPM package for the ELB on the machine hosting your primary _Cloud Controller_ (CLC).

    $ yum install eucalyptus-load-balancer-image

Installing the package will place a `.tgz` file containing the EMI into `/usr/share/eucalyptus-load-balancer-image/`. It will also install a command line tool for working with ELB images as `/usr/bin/euca-install-load-balancer`. To verify that these files are available after installation of the RPM package you can run:

    $ rpm -ql eucalyptus-load-balancer-image

When you do this you should see output similar to the following:

    Output here...

### Setting Load Balancer Properties

## Debugging through log messages
* Show in one of the use cases from above the log messages that are expected to be seen across the various components and how one can understand where an issue is stemming from.

## Gotchas
* This section should show any caveats or known bugs that will trip up users in the field.

*****
[[category.Training]]