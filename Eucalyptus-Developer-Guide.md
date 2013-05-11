This page provides an introduction and launch point for getting involved in the Eucalyptus code base and writing code for or with Eucalyptus.

Eucalyptus is a private-IaaS platform. Specifically, Eucalyptus provides the Amazon Web Services abstractions and services in your own private infrastructure. It is *highly* recommended to become very familiar with the Amazon EC2 an S3 services prior to exploring Eucalyptus and the Eucalyptus code. Here are some good references to begin with:
* [Amazon EC2 Documentation](http://aws.amazon.com/documentation/ec2/)
* [Amazon S3 Documentation](http://aws.amazon.com/documentation/s3/)

## Getting started: Building and Installing Eucalyptus from Source
See the INSTALL file in the Eucalyptus source tree here on Github for installation instructions: [Source Install](https://github.com/eucalyptus/eucalyptus/blob/master/INSTALL)

In the future we will provide a ready-to-go development VM.

## Resources for Technologies that Eucalyptus Uses
Below is a collection of links that provides general information to the different technologies found with in Eucalyptus.
  
Programming links : Below is a collection of links to the different programming/scripting language's documentation found within eucalyptus.  

  * C : http://cprogramminglanguage.net/ -- The Eucalyptus Cluster Controller (CC) and Node Controller (NC) are primarily written in C.
 
  * JAVA : http://www.oracle.com/technetwork/java/javase/overview/index.html - The Eucalyptus Cloud Controller (CLC), Storage Controller (SC), and VMware-Broker (not open-source due to license restrictions) are written primarily in Java. We are currently using Java 7.

  * Python : http://www.python.org/ - Much of the tooling and CLI ([euca2ools](https://github.com/eucalyptus/euca2ools)) for Eucalyptus are written in Python. [Eutester](https://github.com/eucalyptus/eutester), our testing framework is also written in python.

  * Groovy : http://groovy.codehaus.org/ - Groovy is sprinkled around the system, but in the web stacks of the Java components primarily. Most message types are defined in Groovy as well as many of the database scripts we use.

  * BASH : http://www.gnu.org/software/bash/manual/bashref.html
 
  * Perl : http://www.perl.org/


Technology links : Below is a collection of links of various technologies found in eucalyptus.

  * VMWare : http://www.vmware.com/
 
  * XEN : http://www.xen.org/
 
  * KVM : http://www.linux-kvm.org/page/Main_Page - If you want to make any changes to the NC, know KVM inside and out.
 
  * libvirt : http://libvirt.org/ - Libvirt is the primary library we use to interact with KVM, anyone working on the NC should know libvirt (and its limitations) well.
  
  * iSCSI : http://en.wikipedia.org/wiki/ISCSI - ISCSI is the data protocol used for the data path of our EBS implementation. NCs connect to SANs or SCs directly with iSCSI to provide block devices to VM instances.
 
  * Mule : http://www.mulesoft.com/ - Mule is used by Eucalyptus to pass messages between services internally. It sits just above our message stack (HTTP/HTTPs and REST/SOAP) and is used to move POJO messages between internal service queues.
 
  * Netty : http://www.jboss.org/netty - Netty provides the core messaging stack for Eucalyptus by providing non-blocking IO queues that can be pipelined to allow very high throughput messaging for Eucalyptus.
 
  * SOAP : http://www.w3.org/TR/soap/ (Specifications)
 
  * REST : http://en.wikipedia.org/wiki/Representational_State_Transfer
 
  * Hibernate : http://www.hibernate.org/ - Eucalytpus utilizes Hibernate for object persistence in the database. Currently we leverage Postgres SQL behind Hibernate for all metadata persistence.
 
  * ANT : http://ant.apache.org/
 
  * MAKE : http://www.gnu.org/software/make/manual/html_node/index.html
  
  * BOTO : http://code.google.com/p/boto/
  
  * LINUX SCSI Target Framework : http://stgt.sourceforge.net/ - Used by the Storage Controller to provide linux LVM volumes to remote NCs for EBS volume functionality.

  * AXIS2C : http://axis.apache.org/axis2/c/core/ - The web services server used for the CC/NC components.

## High-level Eucalyptus Architecture
See the architecture docs repository: [Eucalyptus Architecture Docs](http://github.com/eucalyptus/architecture-docs)

## Cloud Controller
* General design principles and considerations
* Eucalyptus Web Stack
* Eucalyptus Service Endpoints
* Eucalyptus API request entry points
* Persistence/Database (eucalyptus_cloud, eucalyptus_config, ...)

## Cluster Controller
* [CC WSDL](https://github.com/eucalyptus/eucalyptus/blob/master/wsdl/eucalyptus_cc.wsdl)
* [Debugging C components](Debugging-Eucalyptus-C-language-components)

## Storage Controller
* Storage Controller Service Endpoints
* Storage Controller API request entry points
* Persistence/Database for Storage Controller (eucalyptus_config, eucalyptus_storage database)
* [SC WSDL](https://github.com/eucalyptus/eucalyptus/blob/master/wsdl/eucalyptus_sc.wsdl) - This is use *only* for communication between the SC and the NC for volume Export/Unexport in preparation or teardown of EBS volumes.

## Node Controller
* [CC WSDL](https://github.com/eucalyptus/eucalyptus/blob/master/wsdl/eucalyptus_nc.wsdl)
* [Debugging C components](Debugging-Eucalyptus-C-language-components)