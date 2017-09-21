This document details the region federation implementation in Eucalyptus 4.2.

* [Overview](#overview)
* [Implementation](#implementation)
  * [Region configuration](#region-configuration)
  * [Home region interceptor](#home-region-interceptor)
  * [Identity service](#identity-service)
* [References](#references)



## Overview
Region federation allows distinct Eucalyptus cloud deployments to share IAM functionality so they appear to end users (not administrators) as separate regions of a single cloud.

In the 4.2 release there was insufficient time to implement region federation with the desired architecture. The 4.2 region federation implementation uses the existing persistence mechanisms so has severe functional limitations:


* All regions must be available for global modifications to be performed
* A users home region must be available for them to use other regions

these limitations mean the 4.2 region federation implementation is largely unsuitable for geographically distributed use.


## Implementation
This section give a high level summary of the new components

![](images/services/identity-federation.png)


### Region configuration
Regions are configured via cloud properties:


```
region.region_configuration = 
region.region_enable_ssl = true
region.region_name = 
region.region_ssl_ciphers = RSA:DSS:ECDSA:TLS_EMPTY_RENEGOTIATION_INFO_SCSV:!NULL:!EXPORT:!EXPORT1024:!MD5:!DES:!RC4
region.region_ssl_default_cas = true
region.region_ssl_protocols = TLSv1.2
region.region_ssl_verify_hostnames = true
```
aside from the TLS properties thereis the region configuration JSON and the region name. The region name can be configured by itself, the property is for use whereever the name of the local region is required, for example it is used in the output for the EC2 DescribeRegions action.

The configuration JSON is used with region federation to describe the available regions and their associated properties. This document property is expected to be the same on all clouds that are federated.


### Home region interceptor
The home region interceptor intercepts IAM requests and dispatches them to the right region for handling. A user of the IAM service takes advantage of this functionality when they attempt to use the IAM service against a region other than their home region (the region that owns their accounts metadata)


### Identity service
The identity service handles requests between federated regions. Identity service clients use HTTPS by default with hostname validation, if the certificate authority is recognised then no explicit configuration is necessary, otherwise the region configuration JSON can be used to configure the fingerprint of the expected service certificate. Authentication of clients is via the WS-Security signature in the SOAP service request, the expected client certificate for the region must be configured. There are also CIDR configuration properties available to allow the service to reject requests from unexpected locations.

The identity service handles the following types of request:


* Decoding security tokens
* Describing identity metadata such as accounts, principals, roles and instance profiles
* Euare account credential lookup / certificate signing
* Global name reservation
* Tunnelling IAM actions

Decoding security tokens must be performed by the region that created the token (as determined by partitioning of the associated access key identifier) as an architectural requirement was that regions must not share secrets. Tokens are immutable and can be cached indefinitely so decoding is a performance hit that should not be incurred frequently.

Describing account metadata is used for authentication and also where service reference IAM resources, such as running an EC2 instance with an instance profile.

Euare account credential lookup and certificate signing is necessary to support running an ELB with a server certificate from another region.

Global name reservation is used for identifiers that cannot be partitioned, account aliases and signing certificate identifiers for example. When creating a new account alias all regions must be available and must allow a reservation to be created for the name.

Tunnelling IAM actions is used for dispatching IAM requests to their home region as part of the home region interceptor.


## References

* [[Region Identity Federation Architectural Analysis (Arch)|Region-Identity-Federation-Architectural-Analysis]]
* [[4.2 Federation (QA)|4.2-Federation]]



*****

[[category.confluence]] 
