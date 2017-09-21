This document details the EC2/VPC service implementation (API) in Eucalyptus 4.1.

* [Scope](#scope)
  * [API](#api)
  * [Instance Support Services](#instance-support-services)
  * [Related Service Changes](#related-service-changes)
  * [Notes](#notes)
* [Installation / Configuration](#installation-/-configuration)
  * [Enable VPC Networking](#enable-vpc-networking)
  * [VPC Cloud Properties](#vpc-cloud-properties)
* [Administration](#administration)
  * [Create Default VPC](#create-default-vpc)
  * [Verbose Listing](#verbose-listing)
  * [Resource Deletion](#resource-deletion)
* [Troubleshooting](#troubleshooting)
  * [Network Information](#network-information)
  * [MAC Addresses](#mac-addresses)
* [API Details](#api-details)
  * [Actions](#actions)
  * [Resource Tagging](#resource-tagging)
  * [Resource-Level Permissions](#resource-level-permissions)
  * [Resource Filtering](#resource-filtering)



## Scope
This section gives a high-level summary of the scope of the VPC service implementation in the 4.1 release.


### API
At a high level, the API actions related to the following VPC resources are implemented:


* DHCP Options
* Internet Gateways
* Network Interfaces (primary only)
* Network Acls / Entries
* Route Tables / Routes
* Subnets
* Vpcs

APIs related to the following VPC resources are not implemented:


* VPC Peering Connections
* VPN Connections / Routes
* VPN Gateway
* Customer Gateway
* Private IP addresses

Resource tagging and filtering is implemented for implemented actions. Resource-level IAM permissions are supported for VPC resources.

Full details of implemented actions are provided later in this document.


### Instance Support Services
Instance meta-data is supported for VPC instances (configuration required)

VPC DNS is not supported in 4.1, so the VPC attributes related to DNS are always treated as false.


### Related Service Changes
The following services were updated in 4.1. to support EC2/VPC:


* Auto Scaling - Running instances in a VPC
* CloudFormation - VPC resource support
* ELB - Support for internet-facing ELB with VPC


### Notes

* Multiple network interfaces for an instance are not supported
* Multiple IP addresses for a network interface are not supported


## Installation / Configuration

### Enable VPC Networking
VPC networking can be enabled via the network configuration JSON introduced for EDGE networking in 4.0. An example "cloud.network.network_configuration" value with VPC enabled is:


```js
{
    "Mode": "VPCMIDO",
    "InstanceDnsServers": [
        "10.111.5.69"
    ],
    "Subnets": [
    ],
    "PublicIps": [
        "10.111.100.0",
        "10.111.100.1",
        "10.111.100.2",
        "10.111.100.3",
        "10.111.100.16",
        "10.111.100.17"
    ],
    "Clusters": [
        {
            "Subnet": {
                "Subnet": "172.32.192.0",
                "Netmask": "255.255.255.0",
                "Name": "172.32.192.0",
                "Gateway": "172.32.192.1"
            },
            "PrivateIps": [
            ...
```
The  _Mode_  option is used to enable VPC functionality. Valid values for the mode are  _EDGE_  and  _VPCMIDO_ , if the option is not present the value defaults to  _EDGE_ .


### VPC Cloud Properties
The following cloud properties are used with VPC in 4.1:



| Property | Default Value | Description | 
|  --- |  --- |  --- | 
| cloud.vpc.defaultvpc | true | Enable default VPC functionality. A default VPC will be created for new accounts when VPC networking is also enabled. | 
| cloud.vpc.networkaclspervpc | 200 | Limit | 
| cloud.vpc.routespertable | 50 | Limit | 
| cloud.vpc.routetablespervpc | 200 | Limit | 
| cloud.vpc.rulespernetworkacl | 20 | Limit | 
| cloud.vpc.rulespersecuritygroup | 50 | Limit | 
| cloud.vpc.securitygroupspernetworkinterface | 5 | Limit | 
| cloud.vpc.securitygroupspervpc | 100 | Limit | 
| cloud.vpc.subnetspervpc | 200 | Limit | 


## Administration
This section covers Eucalyptus specific administrative functionality for VPC.


### Create Default VPC
To create a default VPC for an account use the regular  _euca-create-vpc_  command but specify an account identifier in place of the CIDR block:


```bash
# euare-accountcreate -a test
test    355430790927
# euca-create-vpc 355430790927
VPC    vpc-94c10eab    available    172.31.0.0/16    dopt-f8b7bd60    default
```
This command can be re-run to update an existing default VPC if there are new availability zones (so new default subnets are required)


### Verbose Listing
Describe actions allow administrators to specify the text "verbose" as a resource identifier to list resources from other accounts.


### Resource Deletion
Administrators can delete VPC resources from other accounts. Resource identifiers can be obtained from a verbose listing of resources.


## Troubleshooting

### Network Information
On the enabled CLC, the current network state is available in the  _/var/run/eucalyptus/global_network_info.xml_ file. This file contains both configuration (e.g. from the network JSON) and resource information for instances, security groups, vpcs, etc.


### MAC Addresses
In VPC, MAC addresses are allocated by the EC2 service and not by the back-end. The MAC address is no longer based on the instance identifier, but instead on the identifier of the primary network interface for the instance.


## API Details

### Actions
The following EC2/VPC related actions are implemented in 4.1:



| Action | Notes | 
|  --- |  --- | 
| AssociateDhcpOptions |  | 
| AssociateRouteTable |  | 
| AttachInternetGateway |  | 
| CreateDhcpOptions |  | 
| CreateInternetGateway |  | 
| CreateNetworkAcl |  | 
| CreateNetworkAclEntry |  | 
| CreateNetworkInterface |  | 
| CreateRoute | For GatewayId only, not for InstanceId, VpcPeeringConnectionId, or NetworkInterfaceId. | 
| CreateRouteTable |  | 
| CreateSubnet |  | 
| CreateVpc |  | 
| DeleteDhcpOptions |  | 
| DeleteInternetGateway |  | 
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
| DescribeNetworkAcls |  | 
| DescribeNetworkInterfaceAttribute |  | 
| DescribeNetworkInterfaces |  | 
| DescribeRouteTables |  | 
| DescribeSubnets |  | 
| DescribeVpcAttribute |  | 
| DescribeVpcs |  | 
| DetachInternetGateway |  | 
| DisassociateRouteTable |  | 
| ModifyNetworkInterfaceAttribute |  | 
| ModifySubnetAttribute |  | 
| ModifyVpcAttribute | API only, related functionality not implemented | 
| ReplaceNetworkAclAssociation |  | 
| ReplaceNetworkAclEntry |  | 
| ReplaceRoute |  | 
| ReplaceRouteTableAssociation |  | 
| ResetNetworkInterfaceAttribute |  | 


### Resource Tagging
Tagging is supported for the following resources:


* DhcpOptionSet
* InternetGateway
* NetworkAcl
* NetworkInterface
* RouteTable
* Subnet
* Vpc


### Resource-Level Permissions


| Action | Resource | Condition Keys | 
|  --- |  --- |  --- | 
| AuthorizeSecurityGroupEgress | Security grouparn:aws:ec2: _region_ : _account_ :security-group/ _security-group-id_  | ec2:Regionec2:Vpc | 
| AuthorizeSecurityGroupIngress | Security grouparn:aws:ec2: _region_ : _account_ :security-group/ _security-group-id_  | ec2:Regionec2:Vpc | 
| DeleteDhcpOptions | DHCP options setarn:aws:ec2: _region_ : _account_ :dhcp-options/ _dhcp-options-id_  | ec2:Region | 
| DeleteInternetGateway | Internet gatewayarn:aws:ec2: _region_ : _account_ :internet-gateway/ _igw-id_  | ec2:Region | 
| DeleteNetworkAcl | Network ACLarn:aws:ec2: _region_ : _account_ :network-acl/ _nacl-id_  | ec2:Regionec2:Vpc | 
| DeleteNetworkAclEntry | Network ACLarn:aws:ec2: _region_ : _account_ :network-acl/ _nacl-id_  | ec2:Regionec2:Vpc | 
| DeleteRoute | Route tablearn:aws:ec2: _region_ : _account_ :route-table/ _route-table-id_  | ec2:Regionec2:Vpc | 
| DeleteRouteTable | Route tablearn:aws:ec2: _region_ : _account_ :route-table/ _route-table-id_  | ec2:Regionec2:Vpc | 
| DeleteSecurityGroup | Security grouparn:aws:ec2: _region_ : _account_ :security-group/ _security-group-id_  | ec2:Regionec2:Vpc | 
| RevokeSecurityGroupEgress | Security grouparn:aws:ec2: _region_ : _account_ :security-group/ _security-group-id_  | ec2:Regionec2:Vpc | 
| RevokeSecurityGroupIngress | Security grouparn:aws:ec2: _region_ : _account_ :security-group/ _security-group-id_  | ec2:Regionec2:Vpc | 
| RunInstances | Network interfacearn:aws:ec2: _region_ : _account_ :network-interface/\*arn:aws:ec2: _region_ : _account_ :network-interface/ _eni-id_  | ec2:AvailabilityZoneec2:Regionec2:Subnetec2:Vpc | 
|  | Security grouparn:aws:ec2: _region_ : _account_ :security-group/ _security-group-id_  | ec2:Regionec2:Vpc | 
|  | Subnetarn:aws:ec2: _region_ : _account_ :subnet/ _subnet-id_  | ec2:AvailabilityZoneec2:Regionec2:Vpc | 

Note that the "ec2:Region" condition key evaluates as the value "eucalyptus" in 4.1.


### Resource Filtering


| Action | Filter | 
|  --- |  --- | 
| DescribeDhcpOptions | dhcp-options-id | 
|  | key | 
|  | value | 
|  | tag-key | 
|  | tag-value | 
|  | tag: _key_  | 
| DescribeInternetGateways | attachment.state | 
|  | attachment.vpc-id | 
|  | internet-gateway-id | 
|  | tag-key | 
|  | tag-value | 
|  | tag: _key_  | 
| DescribeNetworkAcls | association.association-id | 
|  | association.network-acl-id | 
|  | association.subnet-id | 
|  | default | 
|  | entry.cidr | 
|  | entry.egress | 
|  | entry.icmp.code | 
|  | entry.icmp.type | 
|  | entry.port-range.from | 
|  | entry.port-range.to | 
|  | entry.protocol | 
|  | entry.rule-action | 
|  | entry.rule-number | 
|  | network-acl-id | 
|  | vpc-id | 
|  | tag-key | 
|  | tag-value | 
|  | tag: _key_  | 
| DescribeNetworkInterfaces | addresses.private-ip-address | 
|  | addresses.primary | 
|  | addresses.association.public-ip | 
|  | addresses.association.owner-id | 
|  | association.allocation-id | 
|  | association.association-id | 
|  | association.ip-owner-id | 
|  | association.public-ip | 
|  | association.public-dns-name | 
|  | attachment.attachment-id | 
|  | attachment.instance-id | 
|  | attachment.instance-owner-id | 
|  | attachment.device-index | 
|  | attachment.status | 
|  | attachment.attach.time | 
|  | attachment.delete-on-termination | 
|  | availability-zone | 
|  | description | 
|  | group-id | 
|  | group-name | 
|  | mac-address | 
|  | network-interface-id | 
|  | owner-id | 
|  | private-ip-address | 
|  | private-dns-name | 
|  | requester-id | 
|  | requester-managed | 
|  | source-dest-check | 
|  | status | 
|  | subnet-id | 
|  | vpc-id | 
|  | tag-key | 
|  | tag-value | 
|  | tag: _key_  | 
| DescribeRouteTables | association.route-table-association-id | 
|  | association.route-table-id | 
|  | association.subnet-id | 
|  | association.main | 
|  | route.destination-cidr-block | 
|  | route.gateway-id | 
|  | route.instance-id | 
|  | route.vpc-peering-connection-id | 
|  | route.origin | 
|  | route.state | 
|  | route-table-id | 
|  | vpc-id | 
|  | tag-key | 
|  | tag-value | 
|  | tag: _key_  | 
| DescribeSubnets | availability-zone | 
|  | available-ip-address-count | 
|  | cidr-block | 
|  | default-for-az | 
|  | state | 
|  | subnet-id | 
|  | vpc-id | 
|  | tag-key | 
|  | tag-value | 
|  | tag: _key_  | 
| DescribeVpcs | cidr | 
|  | dhcp-options-id | 
|  | isDefault | 
|  | state | 
|  | vpc-id | 
|  | tag-key | 
|  | tag-value | 
|  | tag: _key_  | 



*****

[[category.confluence]] 
