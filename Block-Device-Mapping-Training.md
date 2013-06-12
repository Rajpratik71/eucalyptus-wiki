## Feature Overview
In the 3.3.0 release additional block device mapping support was added specific to EBS backed instances. EBS backed instances are often referred to as 'boot from EBS' or bfebs instances, or instances with an EBS root device.
A 'Block Device Mapping' (BDM) is used to specify the block devices to be attached to an instance run from a specific emi/image. The block device mapping for a given instance can be specified in two ways. 
* A block device map can be associated with a specific EMI when registering an EMI. In this case instances launched from this EMI will inherit this block device mapping, and block devices specified in the mapping will be created and attached to each instance run from this EMI. 
* A block device map can be specified for a reservation during the run instances request. By default instances will use the BDM provided by the EMI. Changes and/or additions to the EMI's BDM can be specified during the run request. Changes to the root device are limited to the size and 'delete on terminate' attributes. 
 
#### Component level responsibilities


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