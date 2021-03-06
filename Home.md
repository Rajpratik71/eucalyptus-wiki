Eucalyptus is **open source** software for building **AWS-compatible private clouds.**

This Eucalyptus wiki is a collection of **technical documents** for cloud users, cloud administrators, code contributors and partners.

For the official Eucalyptus web site, visit [eucalyptus.com](https://www.eucalyptus.com).


### Need Help?

Check out the user and administration [documentation](http://docs.hpcloud.com/eucalyptus/), or connect with Eucalyptus on [IRC](http://webchat.freenode.net/?channels=eucalyptus), and the [euca-users mailing list](https://github.com/eucalyptus/eucalyptus/wiki/Eucalyptus-Mailing-Lists). 


### Resources

**About Eucalyptus** &mdash; Eucalyptus is a set of web services, modeled after and compatible with Amazon Web Services (AWS).  Written mostly in Java, Eucalyptus integrates components from over 100 open source projects, tested and packaged into a single easy-to-install and easy-to-use product. Eucalyptus runs on virtualized infrastructure (Linux+KVM).

**Acronym** &mdash; Eucalyptus is actually an acronym, which stands for "Elastic Utility Computing Architecture, Linking Your Programs To Useful Systems".

**Architecture** &mdash; You can see the design documents for Eucalyptus in our [architecture repository](http://github.com/eucalyptus/architecture/wiki).

**AWS Tools** &mdash; Eucalyptus is AWS Compatible.  See the AWS tools we recommend, and how to configure them for use with Eucalyptus, on our [[AWS Tools]] page.

**Bugs** &mdash; You can file [bugs on our Jira instance](https://eucalyptus.atlassian.net).

**Code** &mdash; See the code for the [latest Eucalyptus release](https://github.com/eucalyptus/eucalyptus/tree/master).  You can also see [all of our projects on Github](http://github.com/eucalyptus).

**Contribute** &mdash; If you'd like to contribute, please check out the [Contributor Guidelines](wiki/Contributing) first. We also have some more information on [our Git setup](wiki/Documentation-Contributions) especially focused on sending documentation patches.

**Documentation** &mdash; Reference all of our product [documentation](http://docs.hpcloud.com/eucalyptus/). 

**Feedback** &mdash; You can always ask questions on any of our [[Eucalyptus mailing lists]]. 

**Install** &mdash; To install Eucalyptus, try the one-line installer with [Faststart](https://www.eucalyptus.com/get-started). After install completes (usually in less than 30 minutes) you can immediately begin using your private cloud platform with the default account, user and password.

**IRC** &mdash; You can find us on Freenode [IRC on #eucalyptus](http://webchat.freenode.net/?channels=eucalyptus)

**Mailing List** &mdash; Join one of the [[Eucalyptus mailing lists]] to keep up with latest developments and join our users and developers in conversation.

**Nightlies** &mdash; You can install nightly builds of the latest Eucalyptus development builds by following [these instructions](http://www.eucalyptus.com/docs/latest/ig/installing_euca_nightlies.html).

**People** &mdash; Lots of great people contribute to Eucalyptus. Meet [some of them](wiki/category.people).

**Questions** &mdash; You can ask [questions on StackOverflow](http://stackoverflow.com/search?tab=active&q=eucalyptus) or [Quora](http://www.quora.com/Eucalyptus-Systems). 

**Videos** &mdash; You can see how Eucalyptus works on videos at [YouTube](https://www.youtube.com/playlist?list=PL8SRnLljMoSOatd8q796qMptF1fA1e6Q3&spfreload=10).



### For Cloud Users

* [Bundling Images](wiki/Bundling-Images) - How to use euca-bundle-vol to "rebundle" modified VM instances back into the Eucalyptus image catalog as a new EMI.
* [Instance Best Practices](wiki/Instance-Best-Practices) - Recommendations regarding best practices associated with instance and image management.
* [Convert AMI to EMI](wiki/Convert-AMI-to-EMI) - How to import an AMI (Amazon Machine Image) into Eucalyptus as an EMI (Eucalyptus Machine Image).
* [Kexec Loader](wiki/Kexec-loader) - How to use a specialized EKI and ERI to boot the EMI's own in-filesystem kernel and ramdisk using GRUB.
* [Using PHP with Eucalyptus](wiki/Using-PHP-with-Eucalyptus) - How to use the Amazon AWS PHP SDK with Eucalyptus.
* [Starter Images](https://github.com/eucalyptus/eucalyptus/wiki/Starter-Images) - Starter Images that can be accessed via emis.eucalyptus.com or eustore.
* [Stackato Image](https://github.com/eucalyptus/eucalyptus/wiki/Stackato-Image) - Information regarding status of getting [Stackato](http://docs.stackato.com/index.html) image to work on Eucalyptus.
* [Fog](https://github.com/eucalyptus/eucalyptus/wiki/Fog) - Information regarding status of [Fog](http://fog.io/) working with Eucalyptus
* [Ansible](https://github.com/eucalyptus/eucalyptus/wiki/Ansible) - Using [Ansible](http://ansible.cc) to orchestrate EC2 and Eucalyptus instances; deployment and configuration management.

### For Cloud Administrators

* [Network Troubleshooting FAQ](wiki/Network-Troubleshooting-FAQ) - How to solve common VM networking problems.
* [Eutester and Nagios](wiki/Integrating-Eutester-and-Nagios) - How to setup and configure Eutester and Nagios to carry out a simple test of cloud availablity.
* [Keepalived VIP on Cluster Controllers](https://github.com/eucalyptus/eucalyptus/wiki/Keepalived-VIP-for-Node-Controller-Gateways) - How to install and configure keepalived to be used as a shared IP for Node Controller gateways when using MANAGED mode and a private network.
* [Eucalyptus Storage](https://github.com/eucalyptus/eucalyptus/wiki/Storage) - High-level overview of storage in Eucalyptus including Walrus and the Storage Controller and how they work. Also includes some best practices and recommendations.


### For Developers

* [Design Docs](wiki/DesignDocs) - Internal Eucalyptus design documentation.
* [Debugging Eucalyptus Java-language components](wiki/Debugging-Eucalyptus-Java-language-components) - Advice on working with Eucalyptus from an IDE.
* [Debugging Eucalyptus C-language components](wiki/Debugging-Eucalyptus-C-language-components) - Tools and advice to debug the components of Eucalyptus written in C** &mdash; Node Controller and Cluster Controller.
* [Image deployment and construction](wiki/Image-deployment-and-construction) - Details of how disk images are deployed in the system
* [Developer Guide (incomplete)](wiki/Eucalyptus-Developer-Guide) - A work-in-progress guide to getting started on the Eucalyptus code and architecture.
* [Generating API WSDLS](wiki/Generating-Eucalyptus-API-WSDLs) - How to generate the WSDLS for Eucalyptus services.