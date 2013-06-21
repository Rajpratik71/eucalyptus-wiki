One of the common problems with a new Eucalyptus cloud is that VM instances are not reachable via network. There are a lot of reasons why this could be the case, and this FAQ attempts to cover some of the most common problems and how to solve them.

### Security Groups (MANAGED, MANAGED-NOVLAN)

In Managed and Managed (No VLAN) networking modes, you have to grant access before any network access will reach the VMs:

```
euca-authorize -P tcp -p 22 -s 0.0.0.0/0 default
euca-authorize -P icmp -s 0.0.0.0/0 default
```
The output of `euca-describe-groups` should look like this:

```
GROUP   900692683087    default default group
PERMISSION      900692683087    default ALLOWS  tcp     22      22      FROM    CIDR    0.0.0.0/0
PERMISSION      900692683087    default ALLOWS  icmp    0       0       FROM    CIDR    0.0.0.0/0
```
### External DHCP (MANAGED, MANAGED-NOVLAN, STATIC)

In Managed, Managed (No VLAN), and Static networking modes, Eucalyptus runs its own DHCP server. It only answers requests for specific MAC addresses of VMs that Eucalyptus knows about. But if there is another DHCP server answering requests on your Eucalyptus network, it is likely that that DHCP server will answer first and your Eucalyptus VMs will get an unexpected address from your external DHCP server.

To prevent this, you can configure your external DHCP server to ignore all MAC addresses that begin with **D0:0D**. (Yes, "d00d". Ha ha.) Note that Eucalyptus DHCP only responds to MAC addresses that begin with D0:0D.

This is difficult to detect but you can put an iptables logging rule on your node controller to track DHCP traffic:

```
iptables -A FORWARD -p UDP --sport 67:68 --dport 67:68 -j LOG --log-level 0
```
(TODO: add an example of output highlighting DHCP traffic here)

### Bridge Configuration on NCs (MANAGED-NOVLAN)

In the Managed (No VLAN) networking mode, an Ethernet bridge needs to be configured for the VM interfaces to attach to. This is described in detail in our Installation Guide, but if this is not configured correctly, instances won't be reachable. You can check the bridge configuration on the NC by running:
```
brctl show
```
You should see something like this:

```
[root@CENTOS-x86-64 eucalyptus]# brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.fec6bfecf715       no              vif5.0
                                                        vif0.0
                                                        eth0
```
The important part is that br0 exists, and that eth0 is in the "interfaces" list. Double-check your networking configuration if yours looks different.

### VLAN-tagged packets not passed by switch (MANAGED)

In Managed mode, Eucalyptus tags instance traffic with its own chosen VLAN tags. If your switch ports aren't trunked or configured to forward a range of tags, the traffic won't flow. See your Installation Guide for some manual troubleshooting steps to test VLAN-tagged interfaces and make sure that your switch configuration is correct.

*****

[[category.docs]]
[[category.networking]]
[[category.training]]