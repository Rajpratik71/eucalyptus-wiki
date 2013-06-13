## Feature Overview
In the 3.3.0 release additional block device mapping support was added for EBS backed instances. EBS backed instances are often referred to as 'boot from EBS' (bfebs) instances, or instances with an EBS root device.
In the scope of this wiki, a 'Block Device Mapping' (BDM) is used to specify the block devices to be attached to EBS backed instances at instance launch time.  

#### Using a block device mapping
The block device mapping for a given instance can be specified in two ways. 
* A block device map can be associated with a specific EMI when registering an EMI. In this case instances launched from this EMI will inherit this block device mapping, and block devices specified in the mapping will be created and attached to each instance run from this EMI. 
* A block device map can be specified for a reservation during the run instances request. By default instances will use the BDM provided by the EMI they are run from. Changes and/or additions to this BDM can be specified during the run request. Device attributes provided at run time will override device attributes in the BDM provided by the EMI.  Changes and/or additions to the BDM at run time will only take affect for instances in that specific reservation and will not change the EMIs BDM for future reservations. Note: Changes to the root device are limited to the size and 'delete on terminate' attributes. 

#### Block Device Mapping attributes 
A block device mapping can determine the following:
* name of the block devices (ie: /dev/sda)
* size of the device (in GB) 
* type of device (ephemeral or ebs).
* source of the device (ie snapshot to create the device from, empty ebs volume, or ephemeral disk)
* delete on terminate - whether or not to delete the EBS volume backing the device when the instance terminates. 
* no device, exclude a device from a block device mapping being overridden. 


#### User level operation and tooling
* End users will interact with this feature when creating new EBS backed EMIs, and running instances from existing EBS backed EMIs. The user can customize Block Device Mappings in each operation. 

#### Examples:
For this section, euca2ools examples will be shown as '-b' arguments to be provided during the 'euca-run-instances' and 'euca-register' requests. The boto examples will be shown as the BlockDeviceMapping() object to be passed as the kwarg 'block_device_map' to ec2.register_image() and image run() methods. 

Euca2ools syntax:

```
-b device_name=snapshot-id:size:delete_on_terminate
-b "/dev/sda=snap-ABC1234:<volume size, this may be different than the snap size>:<delete on terminate: true:false>"
```
##### Example #1 - Registering an EMI/Image using a bootable EBS root device (BFEBS) from snapshot snap-ABC1234:

* Euca2ools Example:

```
euca-register --root-device-name /dev/sda -b "/dev/sda=snap-ABC1234::true
```

* Boto Example:
```
from boto.ec2.blockdevicemapping import BlockDeviceMapping, BlockDeviceType
block_device_map = BlockDeviceMapping()
block_dev_type = BlockDeviceType()
block_dev_type.delete_on_termination = True
block_dev_type.snapshot_id = snapshot_id
block_device_map['/dev/sda'] = block_dev_type
image_id = boto.ec2.register_image(name='MyNewImage', block_device_map=block_device_map,         root_device_name='/dev/sda')
```

##### Example #2 - Creating a block device mapping for an ephemeral disk

* Euca2ools:
```
-b '/dev/sdb=ephemeral0'
```

* Boto:
```
block_device_map = BlockDeviceMapping()
block_dev_type = BlockDeviceType()
block_dev_type.ephemeral_name='ephemeral0'
block_device_map['/dev/sdb'] = block_dev_type
```

##### Example #3 - Creating a block device mapping for an EBS block dev created from an existing snapshot.

Note: For EBS block devices created from an existing snapshot, the size will default to the size of the original snapshot/volume. The size can be overridden to be greater than the default size, but not smaller. The delete on terminate flag can be set for EBS block devices as well. For this example assume the original snapshot is 1G in size and the block device mapping will request a 2G volume. The block device mapping will also request the backing volume is 'not' deleted upon instance termination. 

* Euca2ools:
```
-b '/dev/sdb=snap-XXXXYYY:2:false'
```

* Boto:
```
block_device_map = BlockDeviceMapping()
block_dev_type = BlockDeviceType()
block_dev_type.snapshot_id='snap-XXXXYYY'
block_dev_type.delete_on_termination=False
block_dev_type.size=2
block_device_map['/dev/sdb'] = block_dev_type
```

##### Example #4 - Creating a block device mapping for an empty EBS block dev.

Note: For this example the block device mapping will request an empty volume of size 5GB. The block device mapping will also request the backing volume 'is' deleted upon instance termination. 

* Euca2ools:
```
-b '/dev/sdb=:5:true'
```

* Boto:
```
block_device_map = BlockDeviceMapping()
block_dev_type = BlockDeviceType()
block_dev_type.delete_on_termination=True
block_dev_type.size=5
block_device_map['/dev/sdb'] = block_dev_type
```
#### Example #5 Combining multiple block mapping types in request

* Euca2ools
```
euca-register --root-device-name '/dev/sda' -b '/dev/sda=snap-F1DC3D01::True' -b '/dev/sdb=ephemeral0'  -b '/dev/sdc=snap-A41141AA:3:True' -b '/dev/sdd=:1:True' --name 'test2'

```

## Administrative Tasks
### Configuration to be aware of:
* Will make use of the clouds SC for EBS, and Walrus for snapshot capabilities. 
* Users will need proper permissions to EBS and snapshot resources via IAM

### Properties which may affect this feature 
**(see 'euca-describe-properties | grep stor' ):**
* storage.maxtotalvolumesizeingb (which may limit the sum of all EBS volumes created)
* storage.maxvolumesizeingb (which may limit the size of a given EBS volume)
* walrus.storagemaxtotalsnapshotsizeingb (which may limit the sum of all snapshots created)

## Debugging 
#### register/run command is failing and returning an error? 
* Most errors at the CLI level are easy to debug and are either due to incorrect syntax, permissions to execute (IAM rules), lack of resources (see storage/walrus properties listed above and or 'euca-describe-availability-zones --verbose to see what limit is hit).  

#### EBS volumes are not present on guest? 
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



## Gotchas
* As of Eucalyptus 3.3.0 ephemeral disks will not be displayed in describe instance and/or image requests
* As of Eucalyptus 3.3.0 only a single ephemeral device can be requested for an instance and/or image. This device size will determined by the vm/instance type requested at run time. 
* As of Eucalyptus 3.3.0 multiple block device mappings are only supported on EBS backed instances. 
* As of Eucalyptus 3.3.0 multiple block device mappings are not supported for vmware

[[category.Training]]