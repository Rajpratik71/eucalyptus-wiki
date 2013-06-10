## Feature Overview
Elastic Load Balancing automatically distributes incoming application traffic across multiple instances. It enables you to achieve even greater fault tolerance in your applications, seamlessly providing the amount of load balancing capacity needed in response to incoming application traffic. Elastic Load Balancing detects unhealthy instances within a pool and automatically reroutes traffic to healthy instances until the unhealthy instances have been restored.

#### Component level responsibilities
##### Involvement of each Eucalyptus component is wrt to the feature. 
* ELB is built using our implementation of autoscaling.  As such, each ELB is actually backed by a launch config and autoscaling group.

##### Flow of a user request
* Typically a user will: 
1. create a load balancer (this includes the health check and a default listener)
1. register 1 or more instances to the load balancer
1. the load balancer will be the front facing host taking care of spreading load to all registered healthy instances

##### Actions/processes are happening in the background at all times
* At all times autoscaling service will be checking up with the ELB vm as defined by it's autoscaling group to ensure it is healthy and the correct number of servo VM instances are running. If at any time the servo vm becomes unhealty as determined by it's autoscling group a new servo vm will be launched to replace it.
* ELB will also be monitoring the health of instances registered to it. If at any time an instance is unhealthy requests will no longer be routed to that instance.
* By default monitoring is enabled for ELB. There will be ELB metrics submission to cloud watch through the life of the ELB. (This does not mean that instances registered to the ELB have monitoring enabled. They will have to have monitoring turned on if you want EC2 metrics from the instances) 

#### User level operation and tooling
##### How do end users interact with the feature?
Along with new features comes new Euca2ools.  Euca2ools 3 attempts to mirror all AWS cli functionality for ELB. The elb specific commands have the prefix "eulb-" pronounced "you'll be". New CLI guide will cover all the new eulb commands. You can also use EucaLobo to interact with the service.

**NOTE**: Euca2ools 3 contains some commands that are not yet applicable to Eucalyptus these include:
   * eulb-attach-lb-to-subnets [NA, VPC only]
   * eulb-create-app-cookie-stickiness-policy [NA, not supported in 3.3.0]
   * eulb-create-lb-cookie-stickiness-policy [NA, not supported in 3.3.0]
   * eulb-create-lb-policy [NA, not supported in 3.3.0]
   * eulb-delete-lb-policy [NA, not supported in 3.3.0]
   * eulb-describe-lb-policies [NA, not supported in 3.3.0]
   * eulb-describe-lb-policy-types [NA, not supported in 3.3.0]
   * eulb-detach-lb-from-subnets [NA, VPC only not supported in 3.3.0]
   * eulb-set-lb-listener-ssl-cert [NA, no SSL support in 3.3.0]
   * eulb-set-lb-policies-for-backend-server [NA, not supported in 3.3.0]
   * eulb-set-lb-policies-of-listener [NA, not supported in 3.3.0]

##### Show a few typical use cases for the feature end-to-end
A typical use case would be to register several identical web server instances to a load balancer.  In this use case the world would use the load balancers DNS name to reach the site and the load balancer would take care of sending traffic to the different registered instances.
 
POC setup:
* 3 instances running an apache webserver. All belong to security group that allows tcp traffic over port 80
* A load balancer is created with instance health checks done on http port 80 and listening/forwarding http port 80-80
* The 3 instances are registered to the load balancer.
* Once the instances are "InService" requests to the load balancer will round robin around the registered healthy instances.

## Administrative Tasks
### How will the administrators configure the feature?
* The first thing an admin must do is add an ELB servo VM and set "loadbalancing.loadbalancer_emi" property (done automatically when installing the servo vm via [recommended technique](https://github.com/eucalyptus/eucalyptus/wiki/ELB-Internals-Training)). Until this property is set the Load Balancing service will be in not ready state.
* ELB supports IAM and quotas.  Administrators will need to give "allow" permissions for ELB for users who will use ELB. 
### How will admins monitor/create/delete resources from other accounts?
* A default describe lbs as cloud admin will not show all ELB's, use "eulb-describe-lbs verbose".
* The installed ELB emi will be private to the cloud admin and **does not** need to be changed to public for other users to use ELB
* ELB is implemented using autoscaling. As such, when a load balancer is created an associated launch config and auto scaling group are created to manage the launching of the ELB emi and ensuring the load balancer will be recreated in case the lb servo vm is terminated or otherwise becomes unhealthy.
* There are a number of euca properties specific to ELB such as the LB servo emi name, type and number of vms to launch etc.  The only one required to be configured is the emi to use. This is done automatically when using the recommend ELB servo vm installation method.
* When Load balancing across multiple zones there will be an ELB servo vm launched in each zone.

## Debugging through log messages
* Show in one of the use cases from above the log messages that are expected to be seen across the various components and how one can understand where an issue is stemming from.

## Gotchas
* This section should show any caveats or known bugs that will trip up users in the field.

*****
[[category.Training]]