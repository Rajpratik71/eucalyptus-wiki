## Feature Overview

* Tagging

User can add up to 10 tags to the following resources: images, instances, volumes, snapshots, security groups. Resources can be filtered using search bar by the key and value of their tag. The tags can be added/deleted/edited by selecting the resource and then clicking "Manage tags" in the "More actions" menu. Tags can be viewed in the "Tags" expando by clicking on the resource and selecting "Tags".

* Dynamic search bar 

On all landing pages there is search bar that allows finding resources based on one or more filters. 
All possible filter options on the search bar a viewed on click.
Selecting a filter will expose a menu with a list of valid filter values
that you can select by pressing the enter key or clicking

* Changing password

User can change his password from the User Console by clicking at "user@account" on the top right and then  selecting the "Change Password" option. 

* Expandos

After clicking on a resource such as instance, image, volume, snapshot, security group, the area beneath the resource expands to a tabbed interface.

For example: 

You can expand an instance entry in the list to see details about the instance, including associated volumes and tags.

1. Click the name of the instance in the list of instances.
The area beneath the instance name will expand to a tabbed interface.
2. Click the General tab to see a panel of detailed information about the instance, include the volume associated with
the instance.
3. Click the Volumes tab to see a list of volumes attached to the instance.
4. Click the Tags tab to see a list of tags associated with the instance.

You can expand a volume entry in the list to see details about the volume, including associated tags and instances.

You can expand a security group in the list to see details about the security group, including tags and rules associated
with the security group.
etc.
## Gotchas

* Search bar works correctly when filtering by one subject. Filtering by multiple subjects, for instance Status and Tag, is scheduled for 3.3.1.

* In multi cluster mode, on dashboard, filtering by availability zone does not work, the fix is scheduled for 3.3.1.



*****
[[category.Training]]