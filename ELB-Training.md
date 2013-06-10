## Feature Overview
Elastic Load Balancing automatically distributes incoming application traffic across multiple instances. It enables you to achieve even greater fault tolerance in your applications, seamlessly providing the amount of load balancing capacity needed in response to incoming application traffic. Elastic Load Balancing detects unhealthy instances within a pool and automatically reroutes traffic to healthy instances until the unhealthy instances have been restored.

#### Component level responsibilities
##### Involvement of each Eucalyptus component is wrt to the feature. 
* ELB is built using our implementation of autoscaling.  As such, each ELB is actually backed by a launch config and autoscaling group.

##### Typical flow of a user request
1. create a load balancer (this includes the health check and a default listener)
> eulb-create-lb -z PARTI00 -l "lb-port=22, protocol=TCP, instance-port=22, instance-protocol=TCP" My-LB
1. register 1 or more instances to the load balancer
> eulb-register-instances-with-lb --instances i-1DF9440E My-LB
1. the load balancer will be the front facing host taking care of spreading load to all registered healthy instances

##### Actions/processes are happening in the background at all times
* At all times autoscaling service will be checking up with the ELB vm as defined by it's autoscaling group to ensure it is healthy and the correct number of servo VM instances are running. If at any time the servo vm becomes unhealty as determined by it's autoscling group a new servo vm will be launched to replace it.
* ELB will also be monitoring the health of instances registered to it. If at any time an instance is unhealthy requests will no longer be routed to that instance.
* By default monitoring is enabled for ELB. There will be ELB metrics submission to cloud watch through the life of the ELB. (This does not mean that instances registered to the ELB have monitoring enabled. They will have to have monitoring turned on if you want EC2 metrics from the instances)
* When an ELB is created it also gets its own security group. This security group will automatically get allow rules any time a listener is added to an ELB. Similarly if a listener is removed the rule for that protocol and port is removed as well.

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
* If the servo VM is not launching check that there are cloud resources available
* You can check scaling activities to see if it was/is an issue with scaling "euscale-describe-scaling-activities" If scaling was unsuccessful it should keep retrying.
* If the servo VM is running but instances are not reaching InService or go to out of service check that the security group associated with the instance allows comms on the health check protocol and port
* If requests are not reaching registered instances you can log onto the servo VM and check that Haproxy is running and servo script is running
** load-balancer-servo and haproxy process are running?
** /var/log/load-balancer-servo/servo.log
** /var/lib/load-balancer-servo/servo-haproxy.conf 

## Gotchas
* ELB servo VM property must be set
* Users attempting to use ELB must have proper allow policy for ELB
* Instances registered to an ELB must have an ingress rule defined for the the health check protocol and port that the ELB is using.  Same goes for the listener. If an instances security group does not allow TCP on port 80 then requests to an ELB with an HTTP80->80 listener will fail because it cannot reach the instance.
* All load balancers launch config and autoscaling group belong to cloud admin and **NOT** the user. Thus, as the user you cannot see the launch config, autoscaling group or VM servo instance associated with your load balancer.
* You can define how many servo VMs to launch per ELB.  This setting takes effect for all subsequently created ELBs
* As a cloud admin you can scale out an already running load balancer as you have access to the autoscaling group. Just change desired capacity to the number of servo VMs you desire. 

## CLI Examples
* eulb-apply-security-groups-to-lb
[root@eucahost-51-29 ~]# eulb-apply-security-groups-to-lb -g my-group LoadStar
SECURITY_GROUPS

* eulb-configure-healthcheck
[root@eucahost-51-20 ~]# eulb-configure-healthcheck --healthy-threshold=30 --interval=60 -t HTTP:80/index.html --timeout=120 --unhealthy-threshold 360 TripleBeam
HEALTH_CHECK     HTTP:80/index.html     60     120     30     360

* eulb-create-lb
[root@eucahost-51-29 ~]# eulb-create-lb -z PARTI00 -l "lb-port=80, protocol=HTTP, instance-port=80, instance-protocol=HTTP" LoadStar
DNS_NAME     LoadStar-972528928292.lb.localhost

* eulb-create-lb-listeners
[root@eucahost-51-20 ~]# eulb-create-lb-listeners -l "lb-port=22, protocol=TCP, instance-port=22, instance-protocol=TCP" LoadStar

* eulb-delete-lb
[root@c-30 ~]# eulb-delete-lb LoadStar

* eulb-delete-lb-listeners
[root@eucahost-51-20 ~]# eulb-delete-lb-listeners -l 22 TripleBeam

* eulb-deregister-instances-from-lb
[root@eucahost-51-20 ~]# eulb-deregister-instances-from-lb --instances i-50683D12 TripleBeam

* eulb-describe-instance-health
[root@c-07 ~]# eulb-describe-instance-health LoadStar
INSTANCE     i-59A041B8     InService
INSTANCE     i-DEB03DDF     InService
INSTANCE     i-40CC3EC1     InService
INSTANCE     i-71DD408F     InService
INSTANCE     i-B0733FFF     InService

*eulb-describe-lbs
[root@eucahost-51-29 ~]# eulb-describe-lbs
LOAD_BALANCER     LoadStar     LoadStar-972528928292.lb.localhost     2013-05-15T23:31:32.806Z
[root@eucahost-51-29 ~]# eulb-describe-lbs --show-long
LOAD_BALANCER     LoadStar     LoadStar-972528928292.lb.localhost               {interval=30,target=TCP:80,timeout=5,healthy-threshold=3,unhealthy-threshold=2}     PARTI00               i-1DF9440E     {protocol=HTTP,lb-port=80,instance-protocol=HTTP,instance-port=80}                         {owner-alias=972528928292,group-name=euca-internal-972528928292-LoadStar}          2013-05-15T23:31:32.806Z
* eulb-disable-zones-for-lb
[root@c-07 ~]# eulb-disable-zones-for-lb -z PARTI00 LoadStar-1-AZ
AVAILABILITY_ZONES     PARTI01
* eulb-enable-zones-for-lb
[root@c-07 ~]# eulb-enable-zones-for-lb -z PARTI00 LoadStar-1-AZ
AVAILABILITY_ZONES     PARTI01, PARTI00
* eulb-register-instances-with-lb
[root@eucahost-51-29 ~]# eulb-register-instances-with-lb --instances i-1DF9440E LoadStar
INSTANCE     i-1DF9440E

*****
[[category.Training]]