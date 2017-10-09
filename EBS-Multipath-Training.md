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
# multipath_conf='/usr/share/doc/eucalyptus-3.4.0/multipath.conf.example.equallogic'
# multipath_conf='/usr/share/doc/eucalyptus-3.4.0/multipath.conf.example.netapp'
# multipath_conf='/usr/share/doc/eucalyptus-3.4.0/multipath.conf.example.vnx'                
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
#### Debugging General
For general multipath debugging there is lots of good info online for device mapper multipathing. Multipath support requires proper configuration on each component utilizing the feature. If multiple paths are specified for a given partition, than all components of that type will need to have device mapper multipathing setup correctly. The more components in a system to setup the more likely an administrator may misconfigure or overlook an item in the setup process. Below is some common things which have been misconfigured and/or overlooked in the setup process:

1. Is device-mapper-multipath installed on this system? yum info device-mapper-multipath ?
2. Diff /etc/multipath.conf to the example multipath script (for your specific storage backend) provided by Eucalyptus. Is there missing or conflicting config in the diff?
3. Diff /etc/iscsi/iscsid.conf to the example iscsi conf script provided by Eucalyptus. Is there missing or conflicting config in the diff that could be at fault?
4. Did you restart or reload multipathd after changing /etc/multipath.conf? 
5. multipath -ll will provide some amount of information for devices currently multipath'd. In the example below 'mpathaa' is the result of a single volume attached to an NC. The ncpaths property has 2 paths, both are in a good status. 

```
 [root@eucahost-51-75 ~]# multipath -ll
 mpathaa (36006016098b03000eaae95b256d4e211) dm-17 DGC,VRAID
 size=1.0G features='1 queue_if_no_path' hwhandler='1 alua' wp=rw
 |-+- policy='round-robin 0' prio=50 status=active
 | `- 7:0:0:1 sdf 8:80 active ready  running
 `-+- policy='round-robin 0' prio=10 status=enabled
   `- 6:0:0:1 sdd 8:48 active ready  running
```


6.Check iscsiadm -m session -P3 for additional info about the scsi devices in the mpath group you are debugging. From the above example, our volume is using sdf, and sdd for it's 2 paths:

```
[root@eucahost-51-75 ~]# iscsiadm -m session -P3
iSCSI Transport Class version 2.0-870
version 2.0-872.41.el6
Target: iqn.1992-04.com.emc:cx.apm00121200804.a6
	Current Portal: 192.168.25.182:3260,1
	Persistent Portal: 192.168.25.182:3260,1
		**********
		Interface:
		**********
		Iface Name: iface0
		Iface Transport: tcp
		Iface Initiatorname: iqn.1994-05.com.redhat:c788a5412453
		Iface IPaddress: 192.168.51.75
		Iface HWaddress: <empty>
		Iface Netdev: br0
		SID: 1
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 15
		Target Reset Timeout: 30
		LUN Reset Timeout: 10
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: c7a7-10240
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 65536
		FirstBurstLength: 0
		MaxBurstLength: 262144
		ImmediateData: No
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 6	State: running
		scsi6 Channel 00 Id 0 Lun: 0
			Attached scsi disk sdc		State: running
		scsi6 Channel 00 Id 0 Lun: 1
			Attached scsi disk sdd		State: running
Target: iqn.1992-04.com.emc:cx.apm00121200804.b6
	Current Portal: 10.109.25.186:3260,2
	Persistent Portal: 10.109.25.186:3260,2
		**********
		Interface:
		**********
		Iface Name: iface1
		Iface Transport: tcp
		Iface Initiatorname: iqn.1994-05.com.redhat:c788a5412453
		Iface IPaddress: 10.109.51.75
		Iface HWaddress: <empty>
		Iface Netdev: eth1
		SID: 2
		iSCSI Connection State: LOGGED IN
		iSCSI Session State: LOGGED_IN
		Internal iscsid Session State: NO CHANGE
		*********
		Timeouts:
		*********
		Recovery Timeout: 15
		Target Reset Timeout: 30
		LUN Reset Timeout: 10
		Abort Timeout: 15
		*****
		CHAP:
		*****
		username: c7a7-10240
		password: ********
		username_in: <empty>
		password_in: ********
		************************
		Negotiated iSCSI params:
		************************
		HeaderDigest: None
		DataDigest: None
		MaxRecvDataSegmentLength: 262144
		MaxXmitDataSegmentLength: 65536
		FirstBurstLength: 0
		MaxBurstLength: 262144
		ImmediateData: No
		InitialR2T: Yes
		MaxOutstandingR2T: 1
		************************
		Attached SCSI devices:
		************************
		Host Number: 7	State: running
		scsi7 Channel 00 Id 0 Lun: 0
			Attached scsi disk sde		State: running
		scsi7 Channel 00 Id 0 Lun: 1
			Attached scsi disk sdf		State: running
```

7. Check /var/log/eucalyptus/nc.log, /var/log/messages and /var/log/dmesg and/or dmesg for insight into what happened during the attachment process:
Example output from nc.log during volume attach. Take note of the timestamps (11:27:08) for comparing to other logs in the system:
```
2013-06-13 11:27:08 DEBUG 000012427 doAttachVolume           | [i-126041BC][vol-DABC42FE] volume attaching (localDev=/dev/vde)
2013-06-13 11:27:17 DEBUG 000012427 scClientCall             |  done scOps=ExportVolume clientrc=0 opFail=0
2013-06-13 11:27:18 DEBUG 000012427 connect_iscsi_target     | connect script returned: 0, stdout: '/dev/disk/by-id/dm-uuid-mpath-36006016098b03000eaae95b256d4e211', stderr: 'Before connecting:
$VAR1 = {
          'sid' => '1',
          'tpgt' => '1',
          'netdev' => 'br0',
          'lun-0' => 'sdc',
          'hostnumber' => '6',
          'target' => 'iqn.1992-04.com.emc:cx.apm00121200804.a6',
          'pportal' => '192.168.25.182',
          'iface' => 'iface0',
          'portal' => '192.168.25.182'
        };
$VAR1 = {
          'sid' => '2',
          'tpgt' => '2',
          'netdev' => 'eth1',
          'lun-0' => 'sde',
          'hostnumber' => '7',
          'target' => 'iqn.1992-04.com.emc:cx.apm00121200804.b6',
          'pportal' => '10.109.25.186',
          'iface' => 'iface1',
          'portal' => '10.109.25.186'
        };
After connecting:
$VAR1 = {
          'sid' => '1',
          'tpgt' => '1',
          'netdev' => 'br0',
          'lun-0' => 'sdc',
          'lun-1' => 'sdd',
          'hostnumber' => '6',
          'target' => 'iqn.1992-04.com.emc:cx.apm00121200804.a6',
          'pportal' => '192.168.25.182',
          'iface' => 'iface0',
          'portal' => '192.168.25.182'
        };
$VAR1 = {
          'sid' => '2',
          'tpgt' => '2',
          'netdev' => 'eth1',
          'lun-0' => 'sde',
          'lun-1' => 'sdf',
          'hostnumber' => '7',
          'target' => 'iqn.1992-04.com.emc:cx.apm00121200804.b6',
          'pportal' => '10.109.25.186',
          'iface' => 'iface1',
          'portal' => '10.109.25.186'
        };
'
2013-06-13 11:27:18 DEBUG 000012427 doAttachVolume           | [i-126041BC][vol-DABC42FE] attached iSCSI target of host device '/dev/disk/by-id/dm-uuid-mpath-36006016098b03000eaae95b256d4e211'
2013-06-13 11:27:18 DEBUG 000012427 write_xml_file           | [i-126041BC] wrote volume XML to /disk1/storage/eucalyptus/instances/work/KYGU95MZPSCNQDMRCXVP8/i-126041BC/vol-DABC42FE.xml
2013-06-13 11:27:18  INFO 000012427 doAttachVolume           | [i-126041BC][vol-DABC42FE] volume attached as host device '/dev/disk/by-id/dm-uuid-mpath-36006016098b03000eaae95b256d4e211' to guest device 'vde'
```

Example output from /var/log/messages during the attach operation. Note timestamp taken from nc.log (11:27:08):
```
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] READ CAPACITY failed
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] Result: hostbyte=DID_OK driverbyte=DRIVER_SENSE
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] Sense Key : Illegal Request [current]
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] Add. Sense: Logical unit not supported
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] Test WP failed, assume Write Enabled
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] Asking for cache data failed
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] Assuming drive cache: write through
Jun 13 11:27:17 CENTOS-x86-64 kernel: scsi 6:0:0:1: Direct-Access     DGC      VRAID            0532 PQ: 0 ANSI: 4
Jun 13 11:27:17 CENTOS-x86-64 kernel: scsi 6:0:0:1: alua: supports implicit and explicit TPGS
Jun 13 11:27:17 CENTOS-x86-64 kernel: scsi 6:0:0:1: alua: port group 01 rel port 07
Jun 13 11:27:17 CENTOS-x86-64 kernel: scsi 6:0:0:1: alua: rtpg failed with 8000002
Jun 13 11:27:17 CENTOS-x86-64 kernel: scsi 6:0:0:1: alua: port group 01 state N supports tolUsNA
Jun 13 11:27:17 CENTOS-x86-64 kernel: scsi 6:0:0:1: alua: Attached
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:1: Attached scsi generic sg3 type 0
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:1: [sdd] 2097152 512-byte logical blocks: (1.07 GB/1.00 GiB)
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:1: [sdd] Write Protect is off
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:1: [sdd] Write cache: disabled, read cache: enabled, doesn't support DPO or FUA
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] READ CAPACITY failed
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] Result: hostbyte=DID_OK driverbyte=DRIVER_SENSE
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] Sense Key : Illegal Request [current]
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] Add. Sense: Logical unit not supported
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] Test WP failed, assume Write Enabled
Jun 13 11:27:17 CENTOS-x86-64 kernel: sdd:
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] Asking for cache data failed
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] Assuming drive cache: write through
Jun 13 11:27:17 CENTOS-x86-64 kernel: scsi 7:0:0:1: Direct-Access     DGC      VRAID            0532 PQ: 0 ANSI: 4
Jun 13 11:27:17 CENTOS-x86-64 kernel: scsi 7:0:0:1: alua: supports implicit and explicit TPGS
Jun 13 11:27:17 CENTOS-x86-64 kernel: scsi 7:0:0:1: alua: port group 02 rel port 11
Jun 13 11:27:17 CENTOS-x86-64 kernel: scsi 7:0:0:1: alua: rtpg failed with 8000002
Jun 13 11:27:17 CENTOS-x86-64 kernel: scsi 7:0:0:1: alua: port group 02 state A supports tolUsNA
Jun 13 11:27:17 CENTOS-x86-64 kernel: scsi 7:0:0:1: alua: Attached
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:1: Attached scsi generic sg5 type 0
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:1: [sdf] 2097152 512-byte logical blocks: (1.07 GB/1.00 GiB)
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:1: [sdf] Write Protect is off
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:1: [sdf] Write cache: disabled, read cache: enabled, doesn't support DPO or FUA
Jun 13 11:27:17 CENTOS-x86-64 kernel: sdf:
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] READ CAPACITY failed
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] Result: hostbyte=DID_OK driverbyte=DRIVER_SENSE
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] Sense Key : Illegal Request [current]
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] Add. Sense: Logical unit not supported
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] Test WP failed, assume Write Enabled
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] Asking for cache data failed
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:0: [sdc] Assuming drive cache: write through
Jun 13 11:27:17 CENTOS-x86-64 kernel: unknown partition table
Jun 13 11:27:17 CENTOS-x86-64 kernel: unknown partition table
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:1: [sdf] Attached SCSI disk
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 6:0:0:1: [sdd] Attached SCSI disk
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] READ CAPACITY failed
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] Result: hostbyte=DID_OK driverbyte=DRIVER_SENSE
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] Sense Key : Illegal Request [current]
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] Add. Sense: Logical unit not supported
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] Test WP failed, assume Write Enabled
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] Asking for cache data failed
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:0: [sde] Assuming drive cache: write through
Jun 13 11:27:17 CENTOS-x86-64 multipathd: sdf: add path (uevent)
Jun 13 11:27:17 CENTOS-x86-64 multipathd: mpathaa: load table [0 2097152 multipath 1 queue_if_no_path 1 alua 1 1 round-robin 0 1 1 8:80 1]
Jun 13 11:27:17 CENTOS-x86-64 multipathd: mpathaa: event checker started
Jun 13 11:27:17 CENTOS-x86-64 multipathd: sdf path added to devmap mpathaa
Jun 13 11:27:17 CENTOS-x86-64 multipathd: sdd: add path (uevent)
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:1: alua: port group 02 state A supports tolUsNA
Jun 13 11:27:17 CENTOS-x86-64 multipathd: mpathaa: load table [0 2097152 multipath 1 queue_if_no_path 1 alua 2 1 round-robin 0 1 1 8:80 1 round-robin 0 1 1 8:48 1]
Jun 13 11:27:17 CENTOS-x86-64 multipathd: sdd path added to devmap mpathaa
Jun 13 11:27:17 CENTOS-x86-64 kernel: sd 7:0:0:1: alua: port group 02 state A supports tolUsNA

```


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
[[category.ebs]]
[[category.storage]]