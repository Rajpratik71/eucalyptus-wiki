
# Amazon VPC features implementation status as of 2017-01 (Eucalyptus <span style="color: rgb(51,153,102);">4.2</span>/<span style="color: rgb(51,102,255);">4.3</span>/<span style="color: rgb(0,0,255);">4.4</span>/<span style="color: rgb(0,204,255);">5.0</span>).
<span style="color: rgb(255,0,0);">Features not supported in Eucalyptus</span><span style="color: rgb(51,153,102);">Implemented in eucanetd VPCMIDO 4.2</span><span style="color: rgb(51,102,255);">Implemented in eucanetd VPCMIDO 4.3 (no new VPC features planed for 4.4)</span><span style="color: rgb(0,204,255);">Implemented in eucanetd VP</span><span style="color: rgb(0,204,255);">CMID</span><span style="color: rgb(0,204,255);">O 5.0</span><span style="color: rgb(0,0,0);">Features possibly implemented in frontend services or not responsibility of eucanetd</span><span style="color: rgb(51,153,102);">Virtual Private Cloud (VPC)</span>A VPC is a virtual network dedicated to an account. VPCs can be created, configured, and deleted. VPCs are controlled and<span style="color: rgb(0,0,0);"> implemented </span>mostly in Eucalyptus frontend. VPCs without running instances do not use resources inEucalyptus backend. Basicfeatures of VPC are implemented in Eucalyptus.


* a default VPCis created for each account
    * always a /16CIDR block
    * instances created without VPC specification are created in the default VPC

    
* Availability Zones (implemented in the frontend)
* <span style="color: rgb(51,102,255);">attaching and detaching an Internet gateway</span>
* <span style="color: rgb(255,0,0);">endpoints for S3 (not supported in Eucalyptus)</span>
* <span style="color: rgb(255,0,0);">DNS hostnames(not supported in Eucalyptus) - instances receive (or not) DNS hostnames</span>
* <span style="color: rgb(255,0,0);">Hardware tenancy (not supported in Eucalyptus) - whether or not instances run in dedicated hardware</span>
* VPC CIDR block should be >= /28 and <= /16
* a VPC can span multiple AZs
* to delete a VPC, all instances in the VPC must be terminated.
* on VPC delete, all its components are deleted
    * <span style="color: rgb(255,0,0);">VPN connections are not deleted (not supported in Eucalyptus)</span>
    * if a default VPC is deleted, it can only be restored by contacting AWS support

    
* <span style="color: rgb(255,0,0);">a flow log can be created on a VPC to capture traffic flowing to and from interfaces on the VPC (not supported in Eucalyptus)</span>
* a VPC can have tags associated
* <span style="color: rgb(255,0,0);">a VPC can have DHCP options set</span>
* a default Security Group is associated with a VPC on creation
    * the default security group allows inboundtraffic from instances assigned to the same security group and all outbound traffic
    * instances created without specifying a security group are automatically associated with the VPC default security group

    

<span style="color: rgb(255,0,0);">IPv6 support</span>
* <span style="color: rgb(255,0,0);">IPv6 CIDR block can optionally be assigned to a VPC</span>
    * <span style="color: rgb(255,0,0);">when creating a VPC</span>
    * <span style="color: rgb(255,0,0);">associating an IPv6 CIDR block on an extant VPC</span>

    
* <span style="color: rgb(255,0,0);">VPC IPv6 CIDR blocks have fixed prefix length of /56</span>
* <span style="color: rgb(255,0,0);">Users cannot choose the range of addresses or the IPv6 block size</span>
* <span style="color: rgb(255,0,0);">VPC subnet IPv6 CIDR blocks have fixed prefix length of /64</span>
* <span style="color: rgb(255,0,0);">IPv6 CIDR blocks can be disassociated from a subnet</span>
* <span style="color: rgb(255,0,0);">IPv6 CIDR blocks can be disassociated from a VPC</span>
* <span style="color: rgb(255,0,0);">No elastic IPv6 addresses (IPv6 addresses are all public)</span>
* <span style="color: rgb(255,0,0);">VPC VPN connections, customer gateways, NAT devices and VPC endpoints are not supported</span>
* <span style="color: rgb(255,0,0);">Instance types M3 and G2 are not supported</span>
* <span style="color: rgb(255,0,0);">Within a VPC, IPv6 operation of the following is similar (same) of IPv6:</span>
    * <span style="color: rgb(255,0,0);">Security Groups</span>
    * <span style="color: rgb(255,0,0);">Route Tables</span>
    * <span style="color: rgb(255,0,0);">Network ACLs</span>
    * <span style="color: rgb(255,0,0);">VPC Peering</span>
    * <span style="color: rgb(255,0,0);">Internet Gateway</span>
    * <span style="color: rgb(255,0,0);">Direct Connect</span>
    * <span style="color: rgb(255,0,0);">VPC Flow Logs</span>
    * <span style="color: rgb(255,0,0);">DNS resolution</span>

    

<span style="color: rgb(51,153,102);">VPC subnet or subnet</span>A subnet is a range (Classless Inter-Domain Routing - CIDR - block) of IP addresses within a VPCIP address range. Subnets are controlled and implemented mostly in Eucalyptus frontend. Subnets without running instances do not use resources in Eucalyptus backend. Basic features of subnet are implemented in Eucalyptus.


* for each default VPC, a default subnet is created ineach Availability Zone
    * always a /20 CIDR block
    * instances created without subnet specification are created in the default subnet
    * when a new AZ becomes available, a default subnet is added to all default VPCs that have not been modified
    * if a default subnet is deleted, it can only be restored by contacting AWS support

    
* non-default subnets are private when created (i.e., public IP address is not associated with instances launched in non-default subnets)
* subnet's public IP address attribute - whether or not public IP addresses are assigned to instances on creation
* type of subnets:
    * <span style="color: rgb(51,102,255);">public subnet: traffic is routed to an Internet gateway</span>
    * <span style="color: rgb(51,102,255);">private subnet:does not have aroute to an Internet gateway</span>
    * <span style="color: rgb(255,0,0);">VPN-only subnet: does not have a route to an Internet gateway but do have a route to a virtual private gateway (not supported in Eucalyptus)</span>
    * in Eucalyptus, the distinction between public and private networks is whether or not instances can have public/elastic IPs

    
* reserved IPs (example of a /24 subnet):
    * .0 - network address
    * .1 -used by the VPC router
    * <span style="color: rgb(255,0,0);">.2 - AWS provided DNS (not supported in Eucalyptus)</span>
    * .3 - reserved forfuture use
    * .255 - broadcast address (AWS does not support broadcast)

    
* a subnet cannot span multiple AZs
* subnets within a VPC cannot have overlapping CIDR blocks
* to delete a subnet, all instances in the subnet must be terminated
* <span style="color: rgb(255,0,0);"><span style="color: rgb(0,204,255);">a network ACL can be associated with a subnet</span></span>
* <span style="color: rgb(255,0,0);">a flow log can be created on a subnet to capture traffic flowing to and from interfaces on the subnet (not supported in Eucalyptus)</span>
* a subnet can have tags associated

<span style="color: rgb(255,0,0);">IPsec Virtual Private Network (VPN)connection</span><span style="color: rgb(255,0,0);">Amazon VPC allows users toextend their VPCs to an external network through the use of IPsec VPN. VPN connection requires:</span>


* <span style="color: rgb(255,0,0);">virtual private gateway attached to VPC (not implemented in Eucalyptus)</span>
* <span style="color: rgb(255,0,0);">customer gateway (external to Eucalyptus, fully controlled by users)</span>
* <span style="color: rgb(255,0,0);">overlaps of CIDR blocks of VPC and on-premise CIDR blocks are not allowed - i.e., instances in VPC cannot access on-premise resources with IPs in the VPC CIDR block range.</span>

<span style="color: rgb(255,0,0);">VPC peering</span><span style="color: rgb(255,0,0);">VPC peering allows one to establish connection between instances in different VPCs</span>


* <span style="color: rgb(255,0,0);">VPCs can be owned by the same of different accounts</span>
* <span style="color: rgb(255,0,0);">Overlapping CIDR blocks are not allowed.</span>
* <span style="color: rgb(255,0,0);">peering VPCs in different regions not supported</span>
* <span style="color: rgb(255,0,0);">given a pair of VPCs, only one peering is allowed</span>
* <span style="color: rgb(255,0,0);">MTU across VPC peering connection is 1500 bytes</span>
* <span style="color: rgb(255,0,0);">Security Group of peer VPCs cannot be referenced</span>
* <span style="color: rgb(255,0,0);">VPCs are required to be within a single region</span>
* <span style="color: rgb(255,0,0);">peering relationships are not transitive: peering pairs must be explicitly established</span>
* <span style="color: rgb(255,0,0);">requester VPC sends a request to peer VPC; peer VPC must accept to activate the peering</span>
* <span style="color: rgb(255,0,0);">route tables must be manually adjusted</span>

<span style="color: rgb(51,153,102);">Route Table</span><span style="color: rgb(51,102,255);">Each VPC subnet can be associated with a Main Route Table (automatically created), or a Custom Route Table (user-defined).</span>


* <span style="color: rgb(51,102,255);">route tables define rules to direct traffic from subnets</span>
* <span style="color: rgb(51,102,255);">a subnet can only be associated with one route table</span>
* <span style="color: rgb(51,102,255);">one route table can be associated with multiple subnets</span>
* <span style="color: rgb(51,102,255);">Main Route Table can be modified</span>
* <span style="color: rgb(51,102,255);">Main Route Table cannot be deleted</span>
* <span style="color: rgb(51,102,255);">a custom route table can only be deleted if there are no subnets associated</span>
* <span style="color: rgb(0,0,0);">Main Route Table can be replaced with a custom route table (so that it becomes the default route table when new subnets are created)</span>
* <span style="color: rgb(51,102,255);">All route tables contains a local route that enables communication within VPC. This route cannot be deleted.</span>
* <span style="color: rgb(255,0,0);">Route targets can be:</span>
    * <span style="color: rgb(51,102,255);">Internet Gateway ID</span>
    * <span style="color: rgb(255,0,0);">Virtual Private Gateway ID</span>
    * <span style="color: rgb(51,102,255);">NAT Gateway ID</span>
    * <span style="color: rgb(51,102,255);">NAT Instance ID (the instance should have exactly one ENI)</span>
    * <span style="color: rgb(51,102,255);">ENI ID</span>
    * <span style="color: rgb(255,0,0);">VPC peering ID</span>
    * <span style="color: rgb(255,0,0);">VPC endpoint ID</span>

    
* <span style="color: rgb(51,102,255);">Routes through user-created Internet gateways, virtual private gateways, NAT instance/gateway, peering connection, or a VPC endpoint needs to be explicitly added by users</span>
* <span style="color: rgb(255,0,0);">Route propagation can be enabled/disabled</span>

<span style="color: rgb(51,153,102);">Security Group</span>Security Groups control inbound and outbound traffic at the instance (and/or interface)level.


* <span style="color: rgb(51,153,102);">instances can be associated with one or more SGs</span>
    * <span style="color: rgb(51,153,102);">SGs are associated with interfaces. Instance SG refers to the SG associated with the primary interface</span>

    
* instances created without SG being specified, belongs to the default SG of the VPC
* SG rules are not applied to reserved IPs and link-local addresses (169.254.0.0/16)
* <span style="color: rgb(51,153,102);">supports allow rules only</span>
* <span style="color: rgb(51,153,102);">stateful: return traffic automatically allowed</span>
* <span style="color: rgb(51,153,102);">all rules are evaluated before decision</span>
* <span style="color: rgb(51,153,102);">applies to instances that are members to the SG</span>
* SG rules can only reference SGs within a VPC
* <span style="color: rgb(51,153,102);">all protocols with a number can be specified in the rule</span>
* a SG can be only deleted if there is no interface assigned to it

<span style="color: rgb(0,204,255);">Access Control List(ACL)</span><span style="color: rgb(0,51,102);">Access Control Lists control inbound and outbound traffic at the subnet level.</span>


* <span style="color: rgb(0,0,0);">default ACL is associated with a subnet on creation</span>
* <span style="color: rgb(0,204,255);">ACLrules are not applied to reserved IPs and link-local addresses (169.254.0.0/16)</span>
* <span style="color: rgb(0,204,255);">supports allow and deny rules</span>
* <span style="color: rgb(0,204,255);">stateless: return traffic must be explicitly allowed by rules</span>
* <span style="color: rgb(0,204,255);">rules are processed in number order</span>
    * <span style="color: rgb(0,204,255);">highest number for a rule is 32766</span>

    
* <span style="color: rgb(0,204,255);">ACL rules automaticallyapplied toall instances in the associated subnet</span>

<span style="color: rgb(255,0,0);">Flow Log</span><span style="color: rgb(255,0,0);">Allows users to monitor accepted/rejected IP traffic going to/from instances.</span>


* <span style="color: rgb(255,0,0);">can monitor a VPC, a subnet, or an individual instance (interface)</span>
* <span style="color: rgb(255,0,0);">Flow Log data is published to CloudWatch Logs</span>
* <span style="color: rgb(255,0,0);">Can only enable flow logs to VPCs in an account (even if an external VPC is peered)</span>
* <span style="color: rgb(255,0,0);">Flow logs cannot be tagged</span>
* <span style="color: rgb(255,0,0);">Flow logs cannot be changed (needs to be deleted and created anew)</span>
* <span style="color: rgb(255,0,0);">Flow log actions do not support resource-level permissions</span>


## VPC-specific IAM policies
Access to VPC resources can becontrolled through IAM policies

<span style="color: rgb(51,153,102);">Private IP Address</span>
* When an instance is createda primary private IP is specified or automatically assigned
* <span style="color: rgb(255,0,0);">Secondary private IP addresses cam be assigned to interfaces (not supported in Eucalyptus)</span>
* <span style="color: rgb(255,0,0);">Secondary private IP addresses can be reassigned from one interface to another (not supported in Eucalyptus)</span>

<span style="color: rgb(51,153,102);">Public IP Address</span>
* a public IP address can be assigned to an instance on creation


* public IP addresses are released on instance termination
* public IP addresses cannot be manually associated or disassociated

<span style="color: rgb(51,153,102);">Elastic IP Address</span>
* a static public IP
* belongs to a network interface
* when associating an elastic IP to a primary interface with public IP, the public IP is released. When disassociating an elastic IPfrom an interface that had a public IP address, a new public IP address getsassigned
* elastic IPs can be moved among VPCs


## Elastic Network Interface (ENI)

* Attributes of ENI
    * <span style="color: rgb(51,102,255);">primary private IP address</span>
    * <span style="color: rgb(255,0,0);">one or more secondary private IP address (not supported in Eucalyptus)</span>
    * <span style="color: rgb(51,102,255);">one Elastic IP per private IP address</span>
    * one public IP address on the primary ENI - public IP is assigned only if the ENI is created with the instance; if an extant ENI is specified as primary on instance creation, no public IP gets assigned
    * <span style="color: rgb(51,102,255);">one or more security groups</span>
    * <span style="color: rgb(51,102,255);">MAC address</span>
    * <span style="color: rgb(51,102,255);">source/destination check flag</span>
    * description

    
* ENIs can be:
    * created
    * attached to an instance - hot attach (running instance), warm attach (stopped instance), cold attach (instance creation)
    * detached from an instance

    
* <span style="color: rgb(51,102,255);">ENI attributes follow the ENI as it moves from one instance to another</span>
* primary ENI cannot be detached
* ENIs attached to an instance must reside in the same AZ
* Secondary ENIs may require users to manually <span style="color: rgb(68,68,68);">bring up the second interfaces, configure the private IP addresses, and modify the guest OS route table accordingly</span>

<span style="color: rgb(255,0,0);">ec2-net-utils</span>
* <span style="color: rgb(255,0,0);">set of scripts that automate the configuration of ENIs</span>
* <span style="color: rgb(255,0,0);">compatible with Amazon Linux</span>

<span style="color: rgb(255,0,0);">ClassicLink</span><span style="color: rgb(255,0,0);">Enables EC2-classic instances (on 10.0.0.0/8 CIDR) to communicate with VPC instances using private addresses (not supported in Eucalyptus)</span>

<span style="color: rgb(255,0,0);">VPC Endpoint</span>
* <span style="color: rgb(255,0,0);">enables a private connection between VPC instances and AWS service (not supported in Eucalyptus)</span>
* <span style="color: rgb(255,0,0);">multiple endpoint routes to different services in a route tableare allowed</span>
* <span style="color: rgb(255,0,0);">multiple endpoint routes to the same service in different route tables are allowed</span>
* <span style="color: rgb(255,0,0);">multiple endpoints to the same server in a single route table is not allowed</span>
* <span style="color: rgb(255,0,0);">explicit add, modify, and delete of an endpoint route using route table APIs not allowed</span>
* <span style="color: rgb(255,0,0);">endpoint routes are automatically added when associating a route table with an endpoint</span>

<span style="color: rgb(51,102,255);">Internet Gateway</span>
* <span style="color: rgb(51,102,255);">allows communication between instances and the Internet</span>
* <span style="color: rgb(51,102,255);">associating an Internet Gateway to a subnet is not sufficient to provide Internet access to instances - public IP assignment or elastic IP association is required</span>

<span style="color: rgb(255,0,0);">Egress-Only Internet Gateway</span>
* <span style="color: rgb(255,0,0);">allows outbound communication between instances and the Internet using IPv6 addressing</span>
* <span style="color: rgb(255,0,0);">prevents hosts in the Internet to initiate connections with instances</span>

<span style="color: rgb(51,102,255);">NAT Gateway</span>
* <span style="color: rgb(0,0,0);">recommended by AWS over NAT instance</span>
* <span style="color: rgb(255,0,0);"><span style="color: rgb(0,0,0);">public subnet and an</span><span style="color: rgb(51,102,255);">Elastic IP must be specified (Eucalyptus backend allows private subnet)</span></span>
* <span style="color: rgb(255,0,0);">implemented with redundancy in an AZ</span>
* <span style="color: rgb(0,0,0);">users need to update route tables to point to NAT gateway(s)</span>
* <span style="color: rgb(51,102,255);">a NAT Gateway can be used by instances in different subnets and AZs</span>
* <span style="color: rgb(255,0,0);">supports bursts of up to 10 Gbps (depends on deployment hardware and/or topology)</span>
* <span style="color: rgb(0,0,0);">Elastic IP association cannot be changed</span>
* <span style="color: rgb(51,102,255);">NAT gateway supports TCP, UDP, and ICMP</span>
* <span style="color: rgb(0,0,0);">Security Groups cannot be associated</span>
* <span style="color: rgb(0,204,255);">Network ACL of the subnet that the NAT Gateway resides is applied</span>
* <span style="color: rgb(51,102,255);">ports 1024-65535 is used</span>
* <span style="color: rgb(0,0,0);">ENI of NAT gateway can be viewed but not changed</span>
* <span style="color: rgb(0,0,0);">cannot be accessed by ClassicLink</span>
* <span style="color: rgb(0,0,0);">cannot send traffic over VPC endpoints, VPN connections, AWS Direct Connect, and VPC peering</span>
* <span style="color: rgb(0,0,0);">no support for IPsec</span>
* <span style="color: rgb(51,102,255);">support up to 65000 simultaneous connections</span>
* <span style="color: rgb(0,0,0);">IAM users needs permission to create, describe, and delete NAT gateways</span>
* <span style="color: rgb(0,0,0);">no support for resource-level permissions for any of ec2:\*NatGateway\*</span>

<span style="color: rgb(51,102,255);">NAT Instance (all VPC features needed are in place, instance image is not provided)</span>
* <span style="color: rgb(255,0,0);">Amazon provided Amazon Linux AMIs configured to run as NAT instances</span>
* <span style="color: rgb(255,0,0);">iptables IP masquerading is used inside the instance</span>
* <span style="color: rgb(255,0,0);">can be accessed, modified, and saved as user's AMI</span>
* <span style="color: rgb(51,102,255);">SrcDestCheck attribute must be disabled</span>
* <span style="color: rgb(0,0,0);">port forwarding supported - requires user configuration inside the guest</span>


## Dynamic Host Configuration Protocol (DHCP) Option Sets

* customize the options filed of DHCP messages


    * <span style="color: rgb(255,0,0);">domain-name-servers - up to four comma separated DNS servers</span>
    * <span style="color: rgb(255,0,0);">domain-name - space separated domain names (few guest OS accept multiple domain names)</span>
    * <span style="color: rgb(255,0,0);">ntp-servers - up to four NTP server IPs</span>
    * <span style="color: rgb(255,0,0);">netbios-name-servers - up to four NetBIOS name server IPs</span>
    * <span style="color: rgb(255,0,0);">netbios-node-type - 1, 2, 4, or 8 (2 is recommended)</span>

    
* DHCP options cannot be changed after creation
* propagation of newly associated DHCP options depends onDHCP lease expiration

<span style="color: rgb(255,0,0);">Amazon DNS Server</span>
* <span style="color: rgb(255,0,0);">AmazonProvidedDNS maps to DNS server on the base of the VPC network range plus 2 (e.g., 10.0.0.2 on 10.0.0.0/16)</span>
* <span style="color: rgb(255,0,0);">AmazonProvidedDNS relies on Amazon Route53</span>
* <span style="color: rgb(255,0,0);">169.254.169.253 can be used by instances that allows DNS servers in link-local range (Windows 2008 disallows)</span>







*****

[[category.confluence]] 
