
# Amazon VPC features implementation status as of 2017-01 (Eucalyptus <span style="color: rgb(51,153,102);">4.2</span>/<span style="color: rgb(51,102,255);">4.3</span>/<span style="color: rgb(0,0,255);">4.4</span>/<span style="color: rgb(0,204,255);">5.0</span>).
<span style="color: rgb(255,0,0);">Features not supported in Eucalyptus</span> <span style="color: rgb(51,153,102);">Implemented in eucanetd VPCMIDO 4.2</span> <span style="color: rgb(51,102,255);">Implemented in eucanetd VPCMIDO 4.3 (no new VPC features planed for 4.4)</span> <span style="color: rgb(0,204,255);">Implemented in eucanetd VPCMIDO 5.0</span> <span style="color: rgb(0,0,0);">Features possibly implemented in frontend services or not responsibility of eucanetd</span> <span style="color: rgb(51,153,102);">Virtual Private Cloud (VPC)</span> A VPC is a virtual network dedicated to an account. VPCs can be created, configured, and deleted. VPCs are controlled and<span style="color: rgb(0,0,0);"> implemented </span>mostly in Eucalyptus frontend. VPCs without running instances do not use resources in Eucalyptus backend. Basic features of VPC are implemented in Eucalyptus.


* a default VPCis created for each account
    * always a /16CIDR block
    * instances created without VPC specification are created in the default VPC

    
* Availability Zones (implemented in the frontend)
* attaching and detaching an Internet gateway (since 4.3) 
* endpoints for S3 (not yet supported) 
* DNS hostnames - instances receive (or not) DNS hostnames (not yet supported) 
* Hardware tenancy - whether or not instances run in dedicated hardware (not yet supported) 
* VPC CIDR block should be >= /28 and <= /16
* a VPC can span multiple AZs
* to delete a VPC, all instances in the VPC must be terminated.
* on VPC delete, all its components are deleted
    * VPN connections are not deleted (not yet supported) 
    * if a default VPC is deleted, it can only be restored by contacting AWS support

    
* a flow log can be created on a VPC to capture traffic flowing to and from interfaces on the VPC (not yet supported) 
* a VPC can have tags associated
* a VPC can have DHCP options set (not yet supported) 
* a default Security Group is associated with a VPC on creation
    * the default security group allows inboundtraffic from instances assigned to the same security group and all outbound traffic
    * instances created without specifying a security group are automatically associated with the VPC default security group

    

IPv6 support (not yet supported) 
* IPv6 CIDR block can optionally be assigned to a VPC (not yet supported) 
    * when creating a VPC (not yet supported) 
    * associating an IPv6 CIDR block on an extant VPC (not yet supported) 

    
* VPC IPv6 CIDR blocks have fixed prefix length of /56 (not yet supported) 
* Users cannot choose the range of addresses or the IPv6 block size (not yet supported) 
* VPC subnet IPv6 CIDR blocks have fixed prefix length of /64 (not yet supported) 
* IPv6 CIDR blocks can be disassociated from a subnet (not yet supported) 
* IPv6 CIDR blocks can be disassociated from a VPC (not yet supported) 
* No elastic IPv6 addresses (IPv6 addresses are all public) (not yet supported) 
* VPC VPN connections, customer gateways, NAT devices and VPC endpoints are not supported (not yet supported) 
* Instance types M3 and G2 are not supported (not yet supported) 
* Within a VPC, IPv6 operation of the following is similar (same) of IPv6: (not yet supported) 
    * Security Groups (not yet supported) 
    * Route Tables (not yet supported) 
    * Network ACLs (not yet supported) 
    * VPC Peering (not yet supported) 
    * Internet Gateway (not yet supported) 
    * Direct Connect (not yet supported) 
    * VPC Flow Logs (not yet supported) 
    * DNS resolution (not yet supported) 

    

VPC subnet or subnet (since 4.2) A subnet is a range (Classless Inter-Domain Routing - CIDR - block) of IP addresses within a VPCIP address range. Subnets are controlled and implemented mostly in Eucalyptus frontend. Subnets without running instances do not use resources in Eucalyptus backend. Basic features of subnet are implemented in Eucalyptus.


* for each default VPC, a default subnet is created ineach Availability Zone
    * always a /20 CIDR block
    * instances created without subnet specification are created in the default subnet
    * when a new AZ becomes available, a default subnet is added to all default VPCs that have not been modified
    * if a default subnet is deleted, it can only be restored by contacting AWS support

    
* non-default subnets are private when created (i.e., public IP address is not associated with instances launched in non-default subnets)
* subnet's public IP address attribute - whether or not public IP addresses are assigned to instances on creation
* type of subnets:
    * public subnet: traffic is routed to an Internet gateway (since 4.3) 
    * private subnet:does not have aroute to an Internet gateway (since 4.3) 
    * VPN-only subnet: does not have a route to an Internet gateway but do have a route to a virtual private gateway (not yet supported) 
    * in Eucalyptus, the distinction between public and private networks is whether or not instances can have public/elastic IPs

    
* reserved IPs (example of a /24 subnet):
    * .0 - network address
    * .1 -used by the VPC router
    * .2 - AWS provided DNS (not yet supported) 
    * .3 - reserved forfuture use
    * .255 - broadcast address (AWS does not support broadcast)

    
* a subnet cannot span multiple AZs
* subnets within a VPC cannot have overlapping CIDR blocks
* to delete a subnet, all instances in the subnet must be terminated
* a network ACL can be associated with a subnet (not yet supported)  (since 5.0) 
* a flow log can be created on a subnet to capture traffic flowing to and from interfaces on the subnet (not yet supported) 
* a subnet can have tags associated

IPsec Virtual Private Network (VPN)connection (not yet supported) Amazon VPC allows users toextend their VPCs to an external network through the use of IPsec VPN. VPN connection requires: (not yet supported) 


* virtual private gateway attached to VPC (not implemented in Eucalyptus) (not yet supported) 
* customer gateway (external to Eucalyptus, fully controlled by users) (not yet supported) 
* overlaps of CIDR blocks of VPC and on-premise CIDR blocks are not allowed - i.e., instances in VPC cannot access on-premise resources with IPs in the VPC CIDR block range. (not yet supported) 

VPC peering (not yet supported) VPC peering allows one to establish connection between instances in different VPCs (not yet supported) 


* VPCs can be owned by the same of different accounts (not yet supported) 
* Overlapping CIDR blocks are not allowed. (not yet supported) 
* peering VPCs in different regions not supported (not yet supported) 
* given a pair of VPCs, only one peering is allowed (not yet supported) 
* MTU across VPC peering connection is 1500 bytes (not yet supported) 
* Security Group of peer VPCs cannot be referenced (not yet supported) 
* VPCs are required to be within a single region (not yet supported) 
* peering relationships are not transitive: peering pairs must be explicitly established (not yet supported) 
* requester VPC sends a request to peer VPC; peer VPC must accept to activate the peering (not yet supported) 
* route tables must be manually adjusted (not yet supported) 

Route Table (since 4.2) Each VPC subnet can be associated with a Main Route Table (automatically created), or a Custom Route Table (user-defined). (since 4.3) 


* route tables define rules to direct traffic from subnets (since 4.3) 
* a subnet can only be associated with one route table (since 4.3) 
* one route table can be associated with multiple subnets (since 4.3) 
* Main Route Table can be modified (since 4.3) 
* Main Route Table cannot be deleted (since 4.3) 
* a custom route table can only be deleted if there are no subnets associated (since 4.3) 
* <span style="color: rgb(0,0,0);">Main Route Table can be replaced with a custom route table (so that it becomes the default route table when new subnets are created)</span>
* All route tables contains a local route that enables communication within VPC. This route cannot be deleted. (since 4.3) 
* Route targets can be: (not yet supported) 
    * Internet Gateway ID (since 4.3) 
    * Virtual Private Gateway ID (not yet supported) 
    * NAT Gateway ID (since 4.3) 
    * NAT Instance ID (the instance should have exactly one ENI) (since 4.3) 
    * ENI ID (since 4.3) 
    * VPC peering ID (not yet supported) 
    * VPC endpoint ID (not yet supported) 

    
* Routes through user-created Internet gateways, virtual private gateways, NAT instance/gateway, peering connection, or a VPC endpoint needs to be explicitly added by users (since 4.3) 
* Route propagation can be enabled/disabled (not yet supported) 

Security Group (since 4.2) Security Groups control inbound and outbound traffic at the instance (and/or interface)level.


* instances can be associated with one or more SGs (since 4.2) 
    * SGs are associated with interfaces. Instance SG refers to the SG associated with the primary interface (since 4.2) 

    
* instances created without SG being specified, belongs to the default SG of the VPC
* SG rules are not applied to reserved IPs and link-local addresses (169.254.0.0/16)
* supports allow rules only (since 4.2) 
* stateful: return traffic automatically allowed (since 4.2) 
* all rules are evaluated before decision (since 4.2) 
* applies to instances that are members to the SG (since 4.2) 
* SG rules can only reference SGs within a VPC
* all protocols with a number can be specified in the rule (since 4.2) 
* a SG can be only deleted if there is no interface assigned to it

Access Control List(ACL) (since 5.0) <span style="color: rgb(0,51,102);">Access Control Lists control inbound and outbound traffic at the subnet level.</span>


* <span style="color: rgb(0,0,0);">default ACL is associated with a subnet on creation</span>
* ACLrules are not applied to reserved IPs and link-local addresses (169.254.0.0/16) (since 5.0) 
* supports allow and deny rules (since 5.0) 
* stateless: return traffic must be explicitly allowed by rules (since 5.0) 
* rules are processed in number order (since 5.0) 
    * highest number for a rule is 32766 (since 5.0) 

    
* ACL rules automaticallyapplied toall instances in the associated subnet (since 5.0) 

Flow Log (not yet supported) Allows users to monitor accepted/rejected IP traffic going to/from instances. (not yet supported) 


* can monitor a VPC, a subnet, or an individual instance (interface) (not yet supported) 
* Flow Log data is published to CloudWatch Logs (not yet supported) 
* Can only enable flow logs to VPCs in an account (even if an external VPC is peered) (not yet supported) 
* Flow logs cannot be tagged (not yet supported) 
* Flow logs cannot be changed (needs to be deleted and created anew) (not yet supported) 
* Flow log actions do not support resource-level permissions (not yet supported) 


## VPC-specific IAM policies
Access to VPC resources can becontrolled through IAM policies

Private IP Address (since 4.2) 
* When an instance is createda primary private IP is specified or automatically assigned
* Secondary private IP addresses cam be assigned to interfaces (not yet supported) 
* Secondary private IP addresses can be reassigned from one interface to another (not yet supported) 

Public IP Address (since 4.2) 
* a public IP address can be assigned to an instance on creation


* public IP addresses are released on instance termination
* public IP addresses cannot be manually associated or disassociated

Elastic IP Address (since 4.2) 
* a static public IP
* belongs to a network interface
* when associating an elastic IP to a primary interface with public IP, the public IP is released. When disassociating an elastic IPfrom an interface that had a public IP address, a new public IP address getsassigned
* elastic IPs can be moved among VPCs


## Elastic Network Interface (ENI)

* Attributes of ENI
    * primary private IP address (since 4.3) 
    * one or more secondary private IP address (not yet supported) 
    * one Elastic IP per private IP address (since 4.3) 
    * one public IP address on the primary ENI - public IP is assigned only if the ENI is created with the instance; if an extant ENI is specified as primary on instance creation, no public IP gets assigned
    * one or more security groups (since 4.3) 
    * MAC address (since 4.3) 
    * source/destination check flag (since 4.3) 
    * description

    
* ENIs can be:
    * created
    * attached to an instance - hot attach (running instance), warm attach (stopped instance), cold attach (instance creation)
    * detached from an instance

    
* ENI attributes follow the ENI as it moves from one instance to another (since 4.3) 
* primary ENI cannot be detached
* ENIs attached to an instance must reside in the same AZ
* Secondary ENIs may require users to manually <span style="color: rgb(68,68,68);">bring up the second interfaces, configure the private IP addresses, and modify the guest OS route table accordingly</span>

ec2-net-utils (not yet supported) 
* set of scripts that automate the configuration of ENIs (not yet supported) 
* compatible with Amazon Linux (not yet supported) 

ClassicLink (not yet supported) Enables EC2-classic instances (on 10.0.0.0/8 CIDR) to communicate with VPC instances using private addresses (not yet supported) 

VPC Endpoint (not yet supported) 
* enables a private connection between VPC instances and AWS service (not yet supported) 
* multiple endpoint routes to different services in a route tableare allowed (not yet supported) 
* multiple endpoint routes to the same service in different route tables are allowed (not yet supported) 
* multiple endpoints to the same server in a single route table is not allowed (not yet supported) 
* explicit add, modify, and delete of an endpoint route using route table APIs not allowed (not yet supported) 
* endpoint routes are automatically added when associating a route table with an endpoint (not yet supported) 

Internet Gateway (since 4.3) 
* allows communication between instances and the Internet (since 4.3) 
* associating an Internet Gateway to a subnet is not sufficient to provide Internet access to instances - public IP assignment or elastic IP association is required (since 4.3) 

Egress-Only Internet Gateway (not yet supported) 
* allows outbound communication between instances and the Internet using IPv6 addressing (not yet supported) 
* prevents hosts in the Internet to initiate connections with instances (not yet supported) 

NAT Gateway (since 4.3) 
* <span style="color: rgb(0,0,0);">recommended by AWS over NAT instance</span>
* <span style="color: rgb(0,0,0);">public subnet and an (not yet supported) Elastic IP must be specified (Eucalyptus backend allows private subnet) (since 4.3) </span>
* implemented with redundancy in an AZ (not yet supported) 
* <span style="color: rgb(0,0,0);">users need to update route tables to point to NAT gateway(s)</span>
* a NAT Gateway can be used by instances in different subnets and AZs (since 4.3) 
* supports bursts of up to 10 Gbps (depends on deployment hardware and/or topology) (not yet supported) 
* <span style="color: rgb(0,0,0);">Elastic IP association cannot be changed</span>
* NAT gateway supports TCP, UDP, and ICMP (since 4.3) 
* <span style="color: rgb(0,0,0);">Security Groups cannot be associated</span>
* Network ACL of the subnet that the NAT Gateway resides is applied (since 5.0) 
* ports 1024-65535 is used (since 4.3) 
* <span style="color: rgb(0,0,0);">ENI of NAT gateway can be viewed but not changed</span>
* <span style="color: rgb(0,0,0);">cannot be accessed by ClassicLink</span>
* <span style="color: rgb(0,0,0);">cannot send traffic over VPC endpoints, VPN connections, AWS Direct Connect, and VPC peering</span>
* <span style="color: rgb(0,0,0);">no support for IPsec</span>
* support up to 65000 simultaneous connections (since 4.3) 
* <span style="color: rgb(0,0,0);">IAM users needs permission to create, describe, and delete NAT gateways</span>
* <span style="color: rgb(0,0,0);">no support for resource-level permissions for any of ec2:\*NatGateway\*</span>

NAT Instance (all VPC features needed are in place, instance image is not provided) (since 4.3) 
* Amazon provided Amazon Linux AMIs configured to run as NAT instances (not yet supported) 
* iptables IP masquerading is used inside the instance (not yet supported) 
* can be accessed, modified, and saved as user's AMI (not yet supported) 
* SrcDestCheck attribute must be disabled (since 4.3) 
* <span style="color: rgb(0,0,0);">port forwarding supported - requires user configuration inside the guest</span>


## Dynamic Host Configuration Protocol (DHCP) Option Sets

* customize the options filed of DHCP messages


    * domain-name-servers - up to four comma separated DNS servers (not yet supported) 
    * domain-name - space separated domain names (few guest OS accept multiple domain names) (not yet supported) 
    * ntp-servers - up to four NTP server IPs (not yet supported) 
    * netbios-name-servers - up to four NetBIOS name server IPs (not yet supported) 
    * netbios-node-type - 1, 2, 4, or 8 (2 is recommended) (not yet supported) 

    
* DHCP options cannot be changed after creation
* propagation of newly associated DHCP options depends onDHCP lease expiration

Amazon DNS Server (not yet supported) 
* AmazonProvidedDNS maps to DNS server on the base of the VPC network range plus 2 (e.g., 10.0.0.2 on 10.0.0.0/16) (not yet supported) 
* AmazonProvidedDNS relies on Amazon Route53 (not yet supported) 
* 169.254.169.253 can be used by instances that allows DNS servers in link-local range (Windows 2008 disallows) (not yet supported) 







*****

[[category.confluence]] 
[[category.developer]] 

