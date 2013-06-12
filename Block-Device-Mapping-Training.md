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
* source of the device (ie snapshot to create the device from, or empty ebs volume)
* delete on terminate - whether or not to delete the EBS volume backing the device. 
* no device, exclude a device from a block device mapping being overridden. 


#### Examples:
For this section, euca2ools examples will be shown as '-b' arguments to be provided during the 'euca-run-instances' and 'euca-register' requests. The boto examples will be shown as the BlockDeviceMapping() object to be passed as the kwarg 'block_device_map' to ec2.register_image() and image run() methods. 
Euca2ools syntax:
-b device_name:size:delete_on_terminate
-b "/dev/sda=snap-ABC1234:<volume size, this may be different than the snap size>:<delete on terminate: True:False>"

Boto import:
from boto.ec2.blockdevicemapping import BlockDeviceMapping, BlockDeviceType

* Example #1 - Registering an EMI/Image using a bootable EBS root device (BFEBS) from snapshot snap-ABC1234:
Euca2ools Example:
euca-register --root-device-name /dev/sda -b "/dev/sda=snap-ABC1234::True

Boto Example:
block_device_map = BlockDeviceMapping()
block_dev_type = BlockDeviceType()
block_dev_type.delete_on_termination = True
block_dev_type.snapshot_id = snapshot_id
block_device_map['/dev/sda'] = block_dev_type
image_id = self.ec2.register_image(name='MyNewImage,
block_device_map=block_device_map, root_device_name='/dev/sda')
image_id = self.ec2.register_image(name=MyNewImage, block_device_map=block_device_map, root_device_name='/dev/sda')









 
#### Component level responsibilities


#### User level operation and tooling
* End users will interact with this feature when creating new EBS backed EMIs, and running instances from existing EBS backed EMIs. The user can customize Block Device Mappings in each operation. 

* Show a few typical use cases for the feature end-to-end

## Administrative Tasks
* How will the administrators configure the feature?
* How will admins monitor/create/delete resources from other accounts?

## Debugging through log messages
* Show in one of the use cases from above the log messages that are expected to be seen across the various components and how one can understand where an issue is stemming from.

## Gotchas
* This section should show any caveats or known bugs that will trip up users in the field.

[[category.Training]]