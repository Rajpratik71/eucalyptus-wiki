## Feature Overview
Eucalyptus has the ability to utilize multipathing for it's iscsi connections. Sets of redundant physical resources define multiple logical paths between the Eucalyptus components and the storage device(s). The multiple redundant paths provide additional fault tolerance and higher availability for I/O and administration of the storage device(s) in use by Eucalyptus. 

#### Component level responsibilities
*The multipathing functions primarily reside upon the NC and SC. Eucalyptus leverage Linux's device mapper  multipath functions for the iscsi device(s) used with EBS volumes on the NCs and Snapshot creation on the SCs. *The SC also utilizes multiple interfaces/paths for it's SAN administration. 
*CLC stores properties related to path attributes which define which paths both the SC(s) and NC(s) are to use. 

##### Cluster/Partition/Zone level configuration
There are several properties that can be set at the partition level (aka zone, cluster). (To see storage related properties and definitions: euca-describe-properties --verbose | grep stor)
Example of the relevant partition level properties (assuming a partition named 'PARTI00':

###### **property storage.ncpaths**:
This property provides one or more paths for the NC to use for ISCSI traffic. This property can be the address of a single SAN SP, or a list of addresses for each SP which can service the multipathed ISCSI device. The list can be in the format; 
```
storage.ncpaths='<sp1_addr>,<sp2_addr>,<sp3_addr>'

```

or a specific 'iface' who's mapping is defined on each NC can be used in the property such as: 

```
storage.ncpaths=iface0:<sp1_addr>,iface1:<sp2_addr>,iface2:<sp3_addr>
```

euca-describe-properties --verbose output:
```
PROPERTY	PARTI00.storage.ncpaths	10.107.1.200,10.107.1.201,10.108.1.200,10.108.1.201
DESCRIPTION	PARTI00.storage.ncpaths	iSCSI Paths for NC. 
```


###### **property storage.scpaths**:
This property provides one or more paths for the SC to use for ISCSI traffic (used during snapshot creation). This property can be the address of a single SAN SP, or a list of addresses for each SP which can service the multipathed ISCSI device. The list can be in the format; 
```
storage.scpaths='<sp1_addr>,<sp2_addr>,<sp3_addr>'
```  
or a specific 'iface' who's mapping is defined on each SC can be used in the property such as: 
```
storage.scpaths=iface0:<sp1_addr>,iface1:<sp2_addr>,iface2:<sp3_addr>
```

```
PROPERTY	PARTI00.storage.scpaths	10.107.1.200,10.107.1.201,10.108.1.200,10.108.1.201
DESCRIPTION	PARTI00.storage.scpaths	iSCSI Paths for SC
```

###### **property storage.sanhost**:
The sanhost property defines the interface(s) for SAN management/administrative tasks. A redundant list of host entries can be given if the SAN supports multiple management interfaces. This allows Eucalyptus to failover if one interface becomes un-reachable. 
```
PROPERTY	PARTI00.storage.sanhost	10.109.1.26,10.109.1.27
DESCRIPTION	PARTI00.storage.sanhost	Hostname for SAN device.
```

#### Component Level Configuration and setup 
** (Node and Storage Controllers) **
Below is a set of commands run on the remote component(s) to give an example of how we might setup a new SC or NC to utilize device mapper multipathing. 

```
 ### copy iscsid.conf
find /  -name iscsid.conf.example | grep euca | xargs -I '{}' cp '{}' /etc/iscsi/iscsid.conf
		
### logout of iscsi sessions to allow iscsi restart
iscsiadm -m session -u

### start and/or restart iscsid
iscsid; service iscsid restart

### YUM INSTALL "device-mapper-multipath" package
yum -y install device-mapper-multipath

### ENABLE MULTIPATH
mpathconf --enable

### COPY multipath.conf to /etc/multipath.conf                  
find / -name ".$multipath_conf." | grep eucalyptus | xargs -I '{}' cp '{}' /etc/multipath.conf

### START multipathd SERVICE
service multipathd start

### COPY udev rules                                             
cp -f /root/euca_builder/eee/clc/modules/storage-common/udev/12-dm-permissions.rules /etc/udev/rules.d/
```
** eucalyptus.conf 'optional' configuration per SC/NC. **
In QA we may use the following 'storage.ncpaths' and/or 'storage.scpaths' properties. These specify an 'ifaceX value which is then defined in 'eucalyptus.conf' on each NC and/or SC. Specifying a specific interface is optional, and both the 'ifaceX:' syntax in the paths properties as well as the configuration in eucalyptus.conf is only needed as possible way to choose a specific interface for a specific path. 
```
EXAMPLE:
Partition level properties set and viewed via euca-modify-property and euca-describe-properties:
storage.scpaths=iface0:192.168.1.100,iface1:192.168.1.101
storage.ncpaths=iface0:192.168.1.100,iface1:192.168.1.101
```

'If' using the iface syntax in the paths properties, the following line would be added to each NC SC using those path properties. Eucalyptus does not need to be restarted for this configuration addition/change. 
```
STORAGE_INTERFACES="iface0=br0,iface1=eth1"
```

## Administrative Tasks
* How will the administrators configure the feature?
* How will admins monitor/create/delete resources from other accounts?

## Debugging through log messages
* Show in one of the use cases from above the log messages that are expected to be seen across the various components and how one can understand where an issue is stemming from.

## Gotchas
* This section should show any caveats or known bugs that will trip up users in the field.

[[category.Training]]