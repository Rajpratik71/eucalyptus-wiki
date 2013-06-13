## Feature Overview
Eucalyptus has the ability to utilize multipathing for it's iscsi connections. Sets of redundant physical resources define multiple logical paths between the Eucalyptus components and the storage device(s). The multiple redundant paths help insure higher availability for I/O and administration of the storage device(s) in use by Eucalyptus. 

#### Component level responsibilities
The multipathing functions primarily reside upon the NC and SC. However the CLC stores properties related to path attributes which define which paths both the SC(s) and NC(s) are to use. 

##### Cluster/Partition/Zone level configuration
* 


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