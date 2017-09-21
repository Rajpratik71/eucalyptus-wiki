This document details the EC2/VPC service implementation (API) in Eucalyptus 4.3.

* [Scope](#scope)
  * [API](#api)
  * [Long Identifiers](#long-identifiers)
  * [Related Service Changes](#related-service-changes)
  * [Notes](#notes)
  * [VPC Cloud Properties](#vpc-cloud-properties)
* [Administration](#administration)
  * [Verbose Listing](#verbose-listing)
  * [Resource Deletion](#resource-deletion)
* [API Details](#api-details)
  * [Actions](#actions)
  * [Resource Filtering](#resource-filtering)



## Scope
This section gives a high-level summary of the scope of the VPC service implementation in the 4.3 release.


### API
The EC2 API is updated to the 2015-10-01 version to include NAT gateway actions.

At a high level, the API actions related to the following VPC resources are implemented:


* DHCP Options
* Internet Gateways
* NAT Gateways
* Network Interfaces
* Network Acls / Entries
* Route Tables / Routes
* Subnets
* Vpcs

The above list includes these new 4.3 items:


* NAT Gateways
* Network Interfaces - support for secondary interfaces and attach / detach

APIs related to the following VPC resources are not implemented:


* VPC Peering Connections
* VPN Connections / Routes
* VPN Gateway
* Customer Gateway
* Private IP addresses

Resource tagging and filtering is implemented for implemented actions. Resource-level IAM permissions are supported for VPC resources.

Full details of implemented actions are provided later in this document.


### Long Identifiers
NAT gateways use the new long identifier format with 17 hex characters instead of 8.


### Related Service Changes
The following services were updated in 4.3 to support new EC2/VPC functionality:


* CloudFormation - support for secondary network interfaces and NAT gateways
* ELB support


### Notes

* Multiple IP addresses for a network interface are not supported


### VPC Cloud Properties
The following cloud properties are added for VPC in 4.3:



| Property | Default Value | Description | 
|  --- |  --- |  --- | 
| cloud.vpc.natgatewaysperavailabilityzone | 5 | Maximum number of NAT gateways for each availability zone | 


## Administration
This section covers Eucalyptus specific administrative functionality for VPC.


### Verbose Listing
Describe actions allow administrators to specify the text "verbose" as a resource identifier to list resources from other accounts.


### Resource Deletion
Administrators can delete VPC resources from other accounts. Resource identifiers can be obtained from a verbose listing of resources.

For NAT gateways in addition to transitioning the resource to the "deleting" state the administrator can force the immediate deletion of the NAT gateway by invoking the delete action a second time.


## API Details

### Actions
The following EC2/VPC related actions are implemented in 4.3:



| Action | Notes | 
|  --- |  --- | 
| AssociateDhcpOptions\*\* |  | 
| AssociateRouteTable |  | 
| AttachInternetGateway |  | 
| AttachNetworkInterface\* |  | 
| CreateDhcpOptions\*\* |  | 
| CreateInternetGateway |  | 
| CreateNatGateway\* | We do not currently throw IdempotentParameterMismatch when reusing a token but with changed parameters | 
| CreateNetworkAcl\*\* |  | 
| CreateNetworkAclEntry\*\* |  | 
| CreateNetworkInterface |  | 
| CreateRoute\* | For GatewayId, InstanceId, NetworkInterfaceId or NatGatewayId, not for VpcPeeringConnectionId. | 
| CreateRouteTable |  | 
| CreateSubnet |  | 
| CreateVpc |  | 
| DeleteDhcpOptions |  | 
| DeleteInternetGateway |  | 
| DeleteNatGateway\* |  | 
| DeleteNetworkAcl |  | 
| DeleteNetworkAclEntry |  | 
| DeleteNetworkInterface |  | 
| DeleteRoute |  | 
| DeleteRouteTable |  | 
| DeleteSubnet |  | 
| DeleteVpc |  | 
| DescribeAccountAttributes |  | 
| DescribeDhcpOptions |  | 
| DescribeInternetGateways |  | 
| DescribeNatGateways\* |  | 
| DescribeNetworkAcls |  | 
| DescribeNetworkInterfaceAttribute |  | 
| DescribeNetworkInterfaces |  | 
| DescribeRouteTables |  | 
| DescribeSubnets |  | 
| DescribeVpcAttribute |  | 
| DescribeVpcs |  | 
| DetachInternetGateway |  | 
| DetachNetworkInterface\* |  | 
| DisassociateRouteTable |  | 
| ModifyNetworkInterfaceAttribute |  | 
| ModifySubnetAttribute |  | 
| ModifyVpcAttribute\*\* |  **EnableDnsSupport**  does nothing,  **EnableDnsHostnames** controls public host name for instances / interfaces in API responses | 
| ReplaceNetworkAclAssociation |  | 
| ReplaceNetworkAclEntry |  | 
| ReplaceRoute |  | 
| ReplaceRouteTableAssociation |  | 
| ResetNetworkInterfaceAttribute |  | 

\* New or updated for 4.3.

\*\* API only, underlying functionality not implemented


### Resource Filtering
The following resource filters are added in 4.3:



| Action | Filter | 
|  --- |  --- | 
| DescribeNetworkInterfaces | attachment.nat-gateway-id | 
| DescribeRouteTables | route.destination-prefix-list-id | 
|  | route.nat-gateway-id | 
| DescribeNatGateways | nat-gateway-id | 
|  | state | 
|  | subnet-id | 
|  | vpc-id | 



*****

[[category.confluence]] 
