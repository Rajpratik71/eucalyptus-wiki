## Feature Overview

* Tagging

User can add up to 10 tags to the following resources: images, instances, volumes, snapshots, security groups. Resources can be filtered using search bar by the key and value of their tag. The tags can be added/deleted/edited by selecting the resource and then clicking "Manage tags" in the "More actions" menu. Tags can be viewed in the "Tags" expando by clicking on the resource and then clicking the Tags tab.

* Naming resources

If a resource is given a name Myresource during creation, the name is treated as a tag with Key = Name (Case sensitive!) Value = Myresource. 

The name will be displayed in place of resource ID wherever appropriate in dialogs and landing pages to make finding a specific resource easier. Resource ID can be viewed by hovering over resource name. An existing resource can be given a name at any point by creating a tag with Key = Name (Case sensitive!) and Value = the name you would like to assign.  


* Search bar 

The search bar allows filtering resource lists using on one or more filters. 
All possible filter options on the search bar are viewed on click.
For some lists, you can select one or more filters by pressing the enter key or clicking on the search bar and selecting
an available filter from the drop down list. Selecting some filters will expose a menu with a list of valid filter values
that you can select by pressing the enter key or clicking.


For example: 

You can filter the list of instances by selecting the filtering criteria from the filtering drop-down list boxes.Selecting a filter will expose a menu with a list of valid filter values
that you can select by pressing the enter key or clicking.

* VM types

All the VM types are now supported in the launch instance wizard. You can customize the available instance types that are listed in the console for your cloud. To do this:
Modify the 

>[instance_type] 

section of the 

>console.ini 

configuration file. 

Each instance type has a property for
number of CPUs, memory (in megabytes), and disk size (in gigabytes). The default configuration file that is installed
with the console is pre-populated with common instance types:
 
  
    [instance_type]
    m1.small.cpu: 1
    m1.small.mem: 512
    m1.small.disk: 5
    c1.medium.cpu: 2
    ...
This configuration setting should be modifed to match if the default instance types are changed in
the CLC configuration.

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

* Changing password

User can change his password from the User Console by clicking at "user@account" on the top right and then  selecting the "Change Password" option. 

#### User level operation and tooling

* The Eucalyptus Console configuration settings are stored in the console.ini file.
For Centos and RHEL installations from packages, this file is located in

>/etc/eucalyptus-console/console.ini

You should always start (or restart) the console when you make changes to the console configuration.

You can start the console using the following command:
>service eucalyptus-console start

You can restart the console using the following command:
>service eucalyptus-console restart

## Debugging 

* When installed from packages, User Console log is written in: 

>/var/log/messages

### Console does not show any data?

One possible reason for this is that data from CLC does not reach the User Console. 
From the log you can determine if there are any problems connecting to CLC, connection timeouts. 

#### Use case

During stress testing create volume churn would cause the User Console to become unresponsive, showing no data. The cause: overloaded CLC would provide slow response causing connection timeouts in the User Console.    
Currently only the data polling intervall can be configured in console.ini. Timeout is set to 30 seconds, to increase timeout, in /usr/share/eucalyptus-console/static/js/support.js look for:

    $.ajaxSetup({
       type: "POST",
       timeout: 30000
     });


Timeout is set in milliseconds. 

* Restarting console:

1. Restart the console process, or 
>service eucalyptus-console restart

2. clear cache in the browser

3. refresh browser

* Firebug is a good tool for debugging in Firefox.




## Useful links

Demo for new features of User Console can be found here http://vimeo.com/68189848 .

## Gotchas

* Search bar works correctly when filtering by one subject. Filtering by multiple subjects, for instance Status and Tag, is scheduled for 3.3.1.

* In multi cluster mode, on dashboard, filtering by availability zone does not work, the fix is scheduled for 3.3.1.



*****
[[category.Training]]