
# Overview
The RiakCS Provider in the OSG is the client used by the OSG to communicate with a RiakCS OSP(backend) for storing user data.


# Interface
The provider interacts with RiakCS using the S3 REST API exclusively. It is accessed via a single endpoint name (dns or IP). The endpoint may be a load balancer as needed, but there is no assumption by the provider client other than the RiakCS cluster is reachable from that endpoint. The RiakCS provider implementation uses the Amazon Java SDK for S3 to communicate with the RiakCS cluster. This will likely change in the future. Version 1.7.1 of the AWS Java SDK is used.


# Credentials/Access Control
The RiakCS provider uses a single set of RiakCS credentials for all OSG access. Thus, all Eucalyptus user data is owned by a single user in the backing RiakCS cluster. All such data is only accessible by the owning user/account in RiakCS (e.g. 'private' acl used).


## Required Credentials
RiakCS Access Key Id, as given by RiakCS. This must be provided to the provider client/OSG by the cloud admin directly.

RiakCS Secrete Key Id.This must be provided to the provider client/OSG by the cloud admin directly.











*****

[[category.storage]]
[[category.confluence]]
[[category.riakcs]]
[[category.objectstorage]]
