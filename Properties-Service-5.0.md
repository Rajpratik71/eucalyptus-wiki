* [Overview](#overview)
* [Survey](#survey)
  * [Property standardization](#property-standardization)
* [Analysis](#analysis)
  * [Document properties](#document-properties)
* [Design and implementation](#design-and-implementation)
  * [Property standardization](#property-standardization)
* [References](#references)



# Overview
Review backlog of properties service bugs and feature requests and address high value issues for 5.0.

The general areas under consideration are:


* property standardization tickets, e.g. all should have defaults
* property ordering constraints, e.g. have to configure block storage manager before other properties are available to use
* ability to hide less relevant properties (filtering)

    
* document properties

    

The aim is to simplify configuration for eucalyptus.


# Survey
Description of backlog items


## Property standardization
Standardization relates to the following:


* Adding missing non-default values so it is clear what is customized on each install
* Property renaming to match some standard
* Flag read-only properties

Property ordering constraintsSome components have configuration that only becomes available once an instance of the related service becomes enabled.



This is causing problems on cloud setup for block and object storage configuration.

Property filteringProperty filtering covers reducing the amount of property information the user is presented with.

This relates to the following:


* Describing only customized properties, i.e. properties with non-default values
* Describing only properties that are relevant to the current configuration

An example of a relevant property filtering would be to hide walrus related properties when using an alternative s3 provider.



Property documents

Properties with structure more complex than a value or value list or with large size.

This relates to the following:


* ability to set a group of properties at once
* ux for document properties

    

Displaying and updating document properties is hard with the current implementation.


# Analysis
Analysis of backlog items cost and impact.

Property ordering constraints

Requiring that services on different hosts are enabled before configuration can be completed makes cloud install too complex.



Generally we could make properties available before components are enabled. This may fail in some cases when the configuration depends on a provider that is not known to the cloud controller, since that is the host on which the property modification occurs. SAN installs could have this issue if there is SAN specific code installed only on a storage controller host. If this were an issue then potential solutions are:


* Do not validate the configuration
* Ensure the same code is installed on the storage controller and cloud controller

Property filteringRelated issues:


* [EUCA-11302 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-11302)

This relates to reducing the amount of information presented to users. This would be addressed by ways to filter or categorize properties other than naming of properties.

We currently support filtering based on property prefix and tool output is grep friendly.

We will add filtering based on whether a property value has been changed (show only customized/non-default properties)

We could introduce filtering based on tags or categories but these would need to be defined which is out of scope for 5.0.

It would be useful to hide properties that are not in use, for example do not show walrus configuration when it is not a registered service. This hiding of properties is more beneficial for a subset of properties for a service, for example not showing san related configuration when using ceph for block storage. Possible tests for in-use properties:


* Instance of related service registered, e.g. walrusbackend
* Defined property has particular value or values, e.g. one.storage.blockstoragemanager=das|overlay

Possible subsets of service properties are:



| Service | Subset | Property | Notes | 
|  --- |  --- |  --- |  --- | 
| blockstorage | ceph-rbd | one.storage.cephconfigfile |  | 
|  |  | one.storage.cephkeyringfile |  | 
|  |  | one.storage.cephsnapshotpools |  | 
|  |  | one.storage.cephuser |  | 
|  |  | one.storage.cephvolumepools |  | 
|  | das | overlay | one.storage.dasdevice |  | 
|  |  | one.storage.storeprefix |  | 
|  |  | one.storage.tid |  | 
|  |  | one.storage.timeoutinmillis |  | 
|  |  | one.storage.volumesdir |  | 
|  |  | one.storage.zerofillvolumes |  | 
|  | san | one.storage.chapuser |  | 
|  |  | one.storage.ncpaths |  | 
|  |  | one.storage.resourceprefix |  | 
|  |  | one.storage.resourcesuffix |  | 
|  |  | one.storage.sanhost |  | 
|  |  | one.storage.sanpassword |  | 
|  |  | one.storage.sanuser |  | 
|  |  | one.storage.scpaths |  | 
|  |  | one.storage.tasktimeout |  | 
| objectstorage | s3provider | objectstorage.s3provider.s3accesskey | for riakcs ceph-rgw | 
|  |  | objectstorage.s3provider.s3endpoint |  | 
|  |  | objectstorage.s3provider.s3endpointheadresponse |  | 
|  |  | objectstorage.s3provider.s3secretkey |  | 
|  |  | objectstorage.s3provider.s3usebackenddns |  | 
|  |  | objectstorage.s3provider.s3usehttps |  | 

Communicating not-in-use properties to tooling should be simple enough using a flag such as we have now for read-only properties. The complexity is in identifying which properties are in-use, this may require service specific property filtering plugins.


## Document properties
Related issues:


* [EUCA-9121 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-9121)
* [EUCA-10665 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-10665)

This relates to improving the UX for document properties and also updating multiple properties at a time.

Examples of document properties:


* authentication.authorization_cache - Guava cache specfication (~6 \* similar properties for other caches)
* authentication.ldap_integration_configuration - LDAP configuration in json format
* cloudformation.swf_activity_worker_config - Configuration for a simpleworkflow client in json format (~8 \* similar properties for decision workers and other services)
* cloud.network.network_configuration - Network configuration in json format
* region.region_configuration - Region configuration in json format

These properties can contain complex hierarchical configuration that cannot be represented using a static set of properties. Cache specification and simpleworkflow json properties cases could be covered by dynamic property support and could be empty if the configuration was not customized.

Tooling for these properties is more complex as they typically need to have valid syntax such as json or xml. The content-type and expected size of properties must be known to tools as it is not provided by the service.

Server side validation of properties is more complex if configuration needs to move through invalid states since properties can only be updated individually.

There are various use-cases for updating multiple properties at once:


* Atomic updates, apply all changes or none, also impacts validation of properties
* Bulk update, submit multiple changes and get a response that summarizes the results (new values, errors, etc)

Support for atomic update of multiple properties would require fairly major change to the properties service for both the update itself and the property validation framework.

Adding metadata to the properties service response for document properties would be reasonably simple.

We have some tools related to specific properties ( **_euca-lictool_** ) and could add add more tooling for use with, or instead of,  **_euctl_** . Tools could also provide a place for additional contextual validation that my be beyond the scope of validation we would want to perform in the properties service (e.g. checking if instances were running if some network configuration was being changed)


# Design and implementation
Covers changes made for 5.0.


## Property standardization
All read-only properties are removed.

Default values are added for all properties that should have them.

Issues in 5.0:


* [EUCA-11828 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-11828)
* [EUCA-11829 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-11829)
* [EUCA-11831 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-11831)
* [EUCA-11832 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-11832)
* [EUCA-11833 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-11833)
* [EUCA-11834 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-11834)
* [EUCA-11835 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-11835)
* [EUCA-13315 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-13315)
* [EUCA-13316 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-13316)

Property ordering constraintsProperty ordering constraints are removed so configuration can be performed directly after registration of a service.

Issues in 5.0:


* [EUCA-13312 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-13312)

Property filteringAdmin tools (euctl) will be updated to allow display of only values that are customized.

Issues in 5.0:


* [EUCA-13313 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-13313)


# References
[EUCA-12428 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-12428) Illities epic with links to relevant bugs and features



*****

[[category.confluence]] 
