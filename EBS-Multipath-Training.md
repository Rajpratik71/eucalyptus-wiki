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
storage.ncpaths='iface0:<sp1_addr>,iface1:<sp2_addr>,iface2:<sp3_addr>'
```

euca-describe-properties --verbose output:
```
PROPERTY	PARTI00.storage.ncpaths	10.107.1.200,10.107.1.201,10.108.1.200,10.108.1.201
DESCRIPTION	PARTI00.storage.ncpaths	iSCSI Paths for NC. 
```

###### **property storage.scpaths**:
This property provides one or more paths for the SC to use for ISCSI traffic (used during snapshot creation). This property can be the address of a single SAN SP, or a list of addresses for each SP which can service the multipathed ISCSI device. The list can be in the formats:

```
storage.scpaths='<sp1_addr>,<sp2_addr>,<sp3_addr>' 

# or a specific 'iface' who's mapping is defined on each SC can be used in the property such as: 

storage.scpaths='iface0:<sp1_addr>,iface1:<sp2_addr>,iface2:<sp3_addr>'
```

Example:
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
Note: Copy with caution. 
The configuration examples are provided as a reference, 
overwriting the existing configuration completely may not have a desirable outcome for services outside Eucalyptus. 
'diff'ing the existing file and the example and deciding what config makes sense to use is probably a better approach for a live system.
cp /usr/share/doc/eucalyptus-3.3.0/iscsid.conf.example /etc/iscsi/iscsid.conf

Automated QA uses something like the following for source and package builds.
#find /  -name iscsid.conf.example | grep euca | xargs -I '{}' cp '{}' /etc/iscsi/iscsid.conf
		
### logout of iscsi sessions to allow iscsi restart
iscsiadm -m session -u

### start and/or restart iscsid
iscsid; service iscsid restart

### YUM INSTALL "device-mapper-multipath" package
yum -y install device-mapper-multipath

### ENABLE MULTIPATH
mpathconf --enable

### COPY multipath.conf to /etc/multipath.conf  
Note: Copy with caution. 
The configuration examples are provided as a reference, 
overwriting the existing configuration completely may not have a desirable outcome for services outside Eucalyptus. 
'diff'ing the existing file and the example and deciding what config makes sense to use is probably a better approach for a live system.
# Where $multipath_conf would likely be one of the following example configurations provided by eucalyptus:
# /usr/share/doc/eucalyptus-3.4.0/multipath.conf.example.equallogic
# /usr/share/doc/eucalyptus-3.4.0/multipath.conf.example.netapp
# /usr/share/doc/eucalyptus-3.4.0/multipath.conf.example.vnx                
cp $multipath_conf /etc/multipath.conf

Automated QA uses something like the following for source and package builds. 
# find / -name $multipath_conf | grep eucalyptus | xargs -I '{}' cp '{}' /etc/multipath.conf

### START multipathd SERVICE
service multipathd start

### COPY udev rules
Note: This should not need to be done for package installs, for src builds the rules can be copied from the source itself...                                             
cp -f /root/euca_builder/eee/clc/modules/storage-common/udev/12-dm-permissions.rules /etc/udev/rules.d/
```

##### eucalyptus.conf 'optional' configuration per SC/NC. 
If the 'storage.ncpaths' and/or 'storage.scpaths' properties specify an 'ifaceX value, that iface value is then defined in 'eucalyptus.conf' on each NC and/or SC. Specifying a specific interface is optional so both the 'ifaceX:' syntax in the paths properties as well as the configuration in eucalyptus.conf is only needed as possible way to choose a specific interface for a specific path. Forcing a specific interface can also be done through normal networking configuration.  

****EXAMPLE: 

Partition level properties set and viewed via euca-modify-property and euca-describe-properties:
```
storage.scpaths=iface0:192.168.1.100,iface1:192.168.1.101
storage.ncpaths=iface0:192.168.1.100,iface1:192.168.1.101
```

'If' using the iface syntax in the paths properties, the following line would be added to each NC SC using those path properties. Eucalyptus does not need to be restarted for this configuration addition/change. 
```
STORAGE_INTERFACES="iface0=br0,iface1=eth1"
```

## Administrative Tasks
See partition properties from above for multipath specific configuration. The feature should be transparent to other eucalyptus properties and usage. 

## Debugging 

#### Debug#1 - Failing to create volume: 
* Follow CLI level errors if any. These may provide information as to incorrect syntax, exceeding property set limits for resources, etc..
* If a volume id was provided at the CLI level, grep the logs on the SC for the volume id. 
``` 
grep -l <volume id> /var/log/eucalyptus/* 
```

Look for errors in the creation process. If the volume id is not present on the SC, or no logs are reported during the time of creation regarding the creation, grep the CLC logs for the volume id and look for errors on the CLC. 
* If errors are not found and the error is repeatable, it may help to increase log levels on the SC/CLC to log level --debug, etc. 

#### Debug#2 - Failing to attach a volume 
* First confirm the block device mapping for the guest shows the device you're looking for, for example device '/dev/sdb' below is backed by volume 'vol-XYZ12345':
```
euca-describe-instances i-19E9429E
RESERVATION	r-3B81431E	963852387532	default
INSTANCE	i-19E9429E	emi-B2D13F7B	0.0.0.0	0.0.0.0	pending	vic	0	 t1.micro	2013-04-18T00:09:43.881Z	PARTI01	eki-A60E38D2 monitoring-disabled	0.0.0.0	0.0.0.0	 ebs
BLOCKDEVICE	/dev/sda	vol-ABCD1234	2013-04-18T00:09:44.291Z	true
BLOCKDEVICE	/dev/sdb	vol-XYZ12345	2013-04-18T00:09:44.291Z	true
```

* euca-describe-volume backing the device to confirm proper in-use, attached state 
* Use euca-describe-nodes to see which node is hosting the instance. 
* Start from the NC and work backwards 'grep'ing through eucalyptus logs to see how far the attachment request made it through the system. Starting with the NC, then SC, then CLC - 'cat /var/log/eucalyptus/nc.log | grep vol-XYZ12345' or search entire dir 'grep -l vol-XYZ12345 /var/log/eucalyptus/*' . 
* If the volume id is present on the NC check the logs for errors, if there are no errors in the logs, check '/var/log/messages and dmesg on the guest, confirm the guest has the proper PCI, virtio drivers, etc..
* If volume id is not on NC, search SC logs to see if volume id is present: 'grep -l vol-XYZ12345 /var/log/eucalyptus/*'. If using HA, make sure you are searching on the SC which was enabled at the time of request. If so look for errors in logs related to creating or exporting the volume. 
* If volume id is not on NC or SC, search SC logs to see if volume id is present: 'grep -l vol-XYZ12345 /var/log/eucalyptus/*'. If using HA, make sure you are searching on the CLC which was enabled at the time of request. If so look for errors in logs related to creating this volume. 
* If volume id is not on NC, SC, or CLC the request may not have made it to the cloud. Check for cli syntax, proper credentials, and URLs (make sure you're talking to the correct system). 
* Increasing log levels to --debug or greater may provide more insight into where operations are failing in the system. 

#### Debug#3 - Failing to run and/or start an EBS backed instance (bfebs)
* Debugging this case will be a combination of the creation and attachment debugging steps noted above. 
* In order to follow the above debugging steps, retrieve the volume-id(s) by displaying the block device mapping in the describe-instances request:
```
euca-describe-instances i-19E9429E
RESERVATION	r-3B81431E	963852387532	default
INSTANCE	i-19E9429E	emi-B2D13F7B	0.0.0.0	0.0.0.0	pending	vic	0	 t1.micro	2013-04-18T00:09:43.881Z	PARTI01	eki-A60E38D2 monitoring-disabled	0.0.0.0	0.0.0.0	 ebs
BLOCKDEVICE	/dev/sda	vol-ABCD1234	2013-04-18T00:09:44.291Z	true
BLOCKDEVICE	/dev/sdb	vol-XYZ12345	2013-04-18T00:09:44.291Z	true

```




## Gotchas
*As of 3.3.0- This is not supported on Overlay and DAS storage backends.

[[category.Training]]