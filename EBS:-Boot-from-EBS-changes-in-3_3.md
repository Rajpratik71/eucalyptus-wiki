Eucalyptus 3.3 introduces improvements in EBS backed image registration and instances. Some of the high level changes are explained here.

> These changes have been implemented for KVM only. 

## Root block device is /dev/sda and not /dev/sda1 

Eucalyptus does not support booting from partitions (/dev/sda1). Eucalyptus can only boot EMIs from disks (/dev/sd devices) because of hypervisor limitations. Prior to 3.3 though CLC allowed disk partitions in the incoming requests, the user input was ignored and /dev/sda was always treated as root block device. This behavior has been changed in 3.3, all block device mappings in EMI registration and launch instance requests are checked for disk partitions. Upgrade from 3.2.2 to 3.3 modifies EBS backed EMIs and instances to reflect /dev/sda as the root block device so that pre upgrade behavior is preserved.

Block device mapping cannot be a disk partition:  

    $ euca-register -n bfebsimage --root-device-name /dev/sda1 -b /dev/sda1=snap-EA7E3CED
    RegisterImageType: Device name cannot be a partition

    $ euca-run-instances emi-4931419B -b /dev/sda1=snap-EA7E3CED
    RunInstancesType: Device name cannot be a partition

##  Allow multiple EBS block device mappings

EBS backed EMIs and instances support multiple block devices in Eucalyptus 3.3. Refer to Eucalyptus User Guide for various types of block device mappings: <Link to doc>. Block device mappings can be specified as part of creating an EMI so that the mapping can be used by all instances launched from the EMI. Alternatively, block device mappings can be specified at launch instance time and can override the ones in the EMI definition or create new mappings. Overriding EMI mapping at launch instance time could mean one or more of the following:

* Change the type of mapping between EBS and ephemeral devices.
* Omit a block device mapping on the launched instance
* Add/remove backing snapshot for the EBS volume mapped to the device
* Change the size of the EBS volume mapped to the device
* Change the 'delete on termination' setting for the EBS volume
* Change the backing ephemeral disk (id/number?) for the device.

The root block device mapping cannot be entirely overridden at launch instance time. Allowed operations on root device mapping include increasing the volume size and changing the 'delete on termination' setting.

EMI registration with multiple block device mappings:  

    $ euca-register -n bfebsimage --root-device-name /dev/sda -b /dev/sda=snap-EA7E3CED::true -b /dev/sdb=:4:false -b /dev/sdc=ephemeral0
    IMAGE	emi-4931419B

Override EMI device mappings at launch instance time:
 
    $ euca-run-instances emi-4931419B -b /dev/sdb=snap-3A6D3E34:3:true -b /dev/sdc=none
    RESERVATION    r-3629423D	578722639974	default
    INSTANCE       i-27F840AB	emi-4931419B	0.0.0.0	0.0.0.0	pending		0		m1.small

Limited override options on root block device mapping:

    $ euca-run-instances emi-4931419B -b /dev/sda=snap-3A6D3E34
    RunInstancesType: Snapshot ID cannot be modified for the root device. Source snapshot from the image registration will be used for creating the root device

    $ euca-run-instances emi-EA1B3E86 -b /dev/sda=ephemeral0
    RunInstancesType: Logical type cannot be modified for the root device. Source snapshot from the image registration will be used for creating the root device

## No more default ephemeral disk at /dev/sdb

Starting 3.3, EBS backed instances will not expose ephemeral storage by default (in compliance with AWS API). An ephemeral disk can be requested by using a block device mapping either during EMI registration or launch instance time. Only one ephemeral disk can be exposed per instance as of now. This will be fixed in future releases along with support for additional VM types. The upgrade process ensures that resources existing prior to the upgrade continue to exhibit the same behavior i.e. EBS backed EMIs and instances registered before the upgrade will provide ephemeral disk at /dev/sdb without explicitly requesting for one. 

Requesting an ephemeral disk:   
    
    $ euca-register -n bfebsimage --root-device-name /dev/sda -b /dev/sda=snap-3A6D3E34 -b /dev/sdb=ephemeral0
    IMAGE	emi-4982740C

Limiting the total number of ephemeral disks per instance to 1

    $ euca-register -n bfebsimage --root-device-name /dev/sda -b /dev/sda=snap-3A6D3E34 -b /dev/sdb=ephemeral0 -b /dev/sdc=ephemeral1
    RegisterImageType: Only one ephemeral device is supported. More than one ephemeral device mappings found


## Size checks

Block device mappings define the criteria for creating EBS volumes that are attached to the instance. If an EBS mapping specifies both the backing snapshot and the size, Eucalyptus checks to see if the size is greater than or equal to snapshot size before processing the request. The request is rejected if the volume size specified in the mapping is smaller than the snapshot size 

Block device size cannot be smaller than snapshot size:

    $ euca-register -n bfebsimage --root-device-name /dev/sda -b /dev/sda=snap-3A6D3E34:1
    RegisterImageType: Size of the volume cannot be smaller than the source snapshot for /dev/sda

    $ euca-run-instances emi-EA1B3E86 -b /dev/sdb=snap-3A6D3E34:1
    RunInstancesType: Size of the volume cannot be smaller than the source snapshot for /dev/sdb

## Metadata service changes

Metadata service will list all EBS and ephemeral mappings for the instance as defined by launch instance request. The actual device names on the VM instance may or may not exactly match those in the block device mappings (due to KVM limitations?)

Register a EBS backed image and launch an instance using the EMI. Add additional block device mappings at launch time:

    $ euca-register -n bfebsimage --root-device-name /dev/sda -b /dev/sda=snap-A89E3DB2IMAGE	
    emi-0CEC3FFC

    $ euca-run-instances emi-0CEC3FFC -b /dev/sdb=:2:false -b /dev/sdc=ephemeral0
    RESERVATION	    r-B1F443DE	   842251859415	   default
    INSTANCE	    i-EBB040A9	   emi-0CEC3FFC	   0.0.0.0   0.0.0.0	pending	   0    m1.small	

Login to the VM instance and query the metadata for block device mappings

    # curl http://169.254.169.254/latest/meta-data/block-device-mapping/
    emi
    root
    ebs1
    ebs2
    ephemeral0

    # curl http://169.254.169.254/latest/meta-data/block-device-mapping/emi
    /dev/sda
    # curl http://169.254.169.254/latest/meta-data/block-device-mapping/root
    /dev/sda
    # curl http://169.254.169.254/latest/meta-data/block-device-mapping/ebs1
    /dev/sda
    # curl http://169.254.169.254/latest/meta-data/block-device-mapping/ebs2
    /dev/sdb
    # curl http://169.254.169.254/latest/meta-data/block-device-mapping/ephemeral0
    /dev/sdc

## Jira References

### Story
[EUCA-4786](https://eucalyptus.atlassian.net/browse/EUCA-4786) Fully support EBS-based EC2 block device mappings in run-instance and register-instance

### Bugs
[EUCA-4243](https://eucalyptus.atlassian.net/browse/EUCA-4243) User input lost when registering image with multiple device mappings  
[EUCA-3254](https://eucalyptus.atlassian.net/browse/EUCA-3254) RunInstances api call needs to support multiple block device mappings  
[EUCA-4244](https://eucalyptus.atlassian.net/browse/EUCA-4244) Should support images with multiple device mappings  
[EUCA-4047](https://eucalyptus.atlassian.net/browse/EUCA-4047) clc allows bfebs volume size less than the snapshot  
[EUCA-5686](https://eucalyptus.atlassian.net/browse/EUCA-5686) RunInstances should enforce minimum boot from EBS snapshot sizes  
[EUCA-3954](https://eucalyptus.atlassian.net/browse/EUCA-3954) Metadata service doesn't show correct block device mapping for BFEBS instance when block device mapping is set with euca-register  
[EUCA-4081](https://eucalyptus.atlassian.net/browse/EUCA-4081) metadata service incorrect for BFEBS instance  
[EUCA-4932](https://eucalyptus.atlassian.net/browse/EUCA-4932) Failed to launch instance after deleting snapshot associated with an image as non-root ebs mapping  
[EUCA-3271](https://eucalyptus.atlassian.net/browse/EUCA-3271) EBS Backed instance always have an ephemeral disk
[TOOLS-180](https://eucalyptus.atlassian.net/browse/TOOLS-180) euca-register/euca-run-instance: query parameter to suppress block device mapping is not populated  
[TOOLS-181](https://eucalyptus.atlassian.net/browse/TOOLS-181) euca-describe-images does not list all the device mappings  
[TOOLS-182](https://eucalyptus.atlassian.net/browse/TOOLS-182) euca-register request with both --snapshot and -b options tends to muck up the list of device mappings  
[TOOLS-185](https://eucalyptus.atlassian.net/browse/TOOLS-185) Multiple block device mappings for the same device are silently dropped  



***
[[category.ebs]]