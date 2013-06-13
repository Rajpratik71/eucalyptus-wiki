## Feature Overview
Eucalyptus has the ability to utilize multipathing for it's iscsi connections. Sets of redundant physical resources define multiple logical paths between the Eucalyptus components and the storage device(s). The multiple redundant paths help insure higher availability for I/O and administration of the storage device(s) in use by Eucalyptus. 

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
PROPERTY	PARTI00.storage.sanhost	10.109.1.26,10.109.1.27
DESCRIPTION	PARTI00.storage.sanhost	Hostname for SAN device.
```

iface0:192.168.25.182,iface1:10.109.25.186"


#### User level operation and tooling
* How do end users interact with the feature?
* Show a few typical use cases for the feature end-to-end

## Administrative Tasks
* How will the administrators configure the feature?
* How will admins monitor/create/delete resources from other accounts?

## Debugging through log messages
* Show in one of the use cases from above the log messages that are expected to be seen across the various components and how one can understand where an issue is stemming from.

## Gotchas
* This section should show any caveats or known bugs that will trip up users in the field.

[[category.Training]]