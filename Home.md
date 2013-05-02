**Eucalyptus is open source software for building AWS-compatible private clouds.**

This Eucalyptus wiki is a collection of technical documents for cloud users, cloud administrators, code contributors and partners.

For the official Eucalyptus web site, visit [eucalyptus.com](http://www.eucalyptus.com).

If you'd like to contribute, please check out the [Contributor Guidelines](wiki/Contributing) first. We also have some more information on [our Git setup](wiki/Documentation-Contributions) especially focused on sending documentation patches.

You can always ask questions on any of our [[Eucalyptus mailing lists]], or send a question to [feedback@eucalyptus.com](mailto:feedback@eucalyptus.com). 

You can find us on Freenode IRC in #eucalyptus or #eucalyptus-docs, or at any of our [community events](wiki/Community-Events)

### About Eucalyptus

Eucalyptus is a set of web services, modeled after and compatible with Amazon Web Services (AWS).  Written mostly in Java, Eucalyptus integrates components from over 100 open source projects, tested and packaged into a single easy-to-install and easy-to-use product. Eucalyptus runs on virtualized infrastructure (Linux+KVM or VMware).

Eucalyptus is actually an acronym, which stands for "Elastic Utility Computing Architecture, Linking Your Programs To Useful Systems.

### Follow Us

[code on Github](http://github.com/eucalyptus) | [bugs on Jira](https://eucalyptus.atlassian.net) | [questions on StackOverflow](http://stackoverflow.com/search?tab=active&q=eucalyptus) | [discussions on Quora](http://www.quora.com/Eucalyptus-Systems) | [videos on Vimeo](https://vimeo.com/eucalyptus/videos) | [presentations on Slideshare](http://www.slideshare.net/eucalyptus/presentations) | [tweets on Twitter](http://twitter.com/eucalyptus) | [friends on Facebook](http://www.facebook.com/pages/Eucalyptus-Systems-Inc/164828240204708) | [contacts on LinkedIn](http://www.linkedin.com/company/eucalyptus-systems-inc.?trk=hb_tab_compy_id_420170) | [+1s on Google Plus](https://plus.google.com/110470516344014754435?prsrc=3)

### AWS Compatibility

* [[AWS Compatibility]] - tracking compatibility of all AWS ecosystem tools with Eucalyptus.

### Pages for Cloud Users

* [Bundling Images](wiki/Bundling-Images) - How to use euca-bundle-vol to "rebundle" modified VM instances back into the Eucalyptus image catalog as a new EMI.
* [Convert AMI to EMI](wiki/Convert-AMI-to-EMI) - How to import an AMI (Amazon Machine Image) into Eucalyptus as an EMI (Eucalyptus Machine Image).
* [Kexec Loader](wiki/Kexec-loader) - How to use a specialized EKI and ERI to boot the EMI's own in-filesystem kernel and ramdisk using GRUB.
* [Using PHP with Eucalyptus](wiki/Using-PHP-with-Eucalyptus) - How to use the Amazon AWS PHP SDK with Eucalyptus.
* [Starter Images](https://github.com/eucalyptus/eucalyptus/wiki/Starter-Images) - Starter Images that can be accessed via emis.eucalyptus.com or eustore.
* [Stackato Image](https://github.com/eucalyptus/eucalyptus/wiki/Stackato-Image) - Information regarding status of getting [Stackato](http://docs.stackato.com/index.html) image to work on Eucalyptus.
* [Fog](https://github.com/eucalyptus/eucalyptus/wiki/Fog) - Information regarding status of [Fog](http://fog.io/) working with Eucalyptus
* [Ansible](https://github.com/eucalyptus/eucalyptus/wiki/Ansible) - Using [Ansible](http://ansible.cc) to orchestrate EC2 and Eucalyptus instances; deployment and configuration management.
### Pages for Cloud Administrators

* [Network Troubleshooting FAQ](wiki/Network-Troubleshooting-FAQ) - How to solve common VM networking problems.
* [Fedora 18 Single System Install](wiki/Fedora-18-Single-System-Install) - How to install a full Eucalyptus test environment on a single system.
* [Eutester and Nagios](wiki/Integrating-Eutester-and-Nagios) - How to setup and configure Eutester and Nagios to carry out a simple test of cloud availablity,
* [Keepalived VIP on Cluster Controllers](https://github.com/eucalyptus/eucalyptus/wiki/Keepalived-VIP-for-Node-Controller-Gateways) - How to install and configure keepalived to be used as a shared IP for Node Controller gateways when using MANAGED mode and a private network
* [Eucalyptus Storage](https://github.com/eucalyptus/eucalyptus/wiki/Storage) - High-level overview of storage in Eucalyptus including Walrus and the Storage Controller and how they work. Also includes some best practices and recommendations.

### Developer Resources

* [Design Docs](wiki/DesignDocs) - Internal Eucalyptus design documentation.
* [Debugging Eucalyptus Java-language components](wiki/Debugging-Eucalyptus-Java-language-components) - Advice on working with Eucalyptus from an IDE.
* [Debugging Eucalyptus C-language components](wiki/Debugging-Eucalyptus-C-language-components) - Tools and advice to debug the components of Eucalyptus written in C: Node Controller and Cluster Controller.
* [UI Guide](wiki/UI-Guide) - Information on the layout of the web UI and how to modify it.

### Finding Pages

* [page listing](wiki/_pages)
* [[index of categories]]

We've got some great people working on Eucalyptus: meet some of them.  
* [[category.people]]

Look forward to seeing you soon!