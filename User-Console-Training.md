## Feature Overview
*Tagging
User can add up to 10 tags to the following resources: images, instances, volumes, snapshots, security groups. Resources can be filtered using search bar by the key and value of their tag. The tags can be added/deleted/edited by selecting the resource and then clicking "Manage tags" in the "More actions" menu. Tags can be viewed in the "Tags" expando by clicking on the resource and selecting "Tags".

*Dynamic search bar

#### Component level responsibilities
* Talk about what the involvement of each Eucalyptus component is wrt to the feature. 
* Are there any new components being added? 
* What is the flow of a user request? 
* What actions/processes are happening in the background at all times? 

#### User level operation and tooling
* How do end users interact with the feature?
* Show a few typical use cases for the feature end-to-end

## Administrative Tasks
* How will the administrators configure the feature?
* How will admins monitor/create/delete resources from other accounts?

## Debugging through log messages
* Show in one of the use cases from above the log messages that are expected to be seen across the various components and how one can understand where an issue is stemming from.

## Gotchas

* Search bar works correctly when filtering by one subject. Filtering by multiple subjects, for instance Status and Tag, is scheduled for 3.3.1.

* In multi cluster mode, on dashboard, filtering by availability zone does not work, the fix is scheduled for 3.3.1.



*****
[[category.Training]]