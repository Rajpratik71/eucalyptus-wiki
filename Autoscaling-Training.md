## Feature Overview
Auto Scaling allows you to scale your capacity up or down automatically according to conditions you define. With Auto Scaling, you can ensure that the number of instances you’re using increases seamlessly during demand spikes to maintain performance, and decreases automatically during demand lulls. Auto Scaling is particularly well suited for applications that experience hourly, daily, or weekly variability in usage. Auto Scaling is enabled by CloudWatch and/or scaling can be done with manual intervention.

#### Component level responsibilities
##### Involvement of each Eucalyptus component is wrt to the feature.
Auto scaling is a new service in Eucalyptus. It is tightly integrated with Cloud Watch and ELB. Alarms and policies can be created such that when cloud watch detects any of the set criteria scaling can occur either up or down.  For example, scaling can happen at predetermined times, dynamically based on CPU or network usage or manually by executing a scaling policy.
##### Are there any new components being added
The new component service is called AutoScaling. At this time the service is colocated with the CLC
##### What is the flow of a user request
At its core auto scaling relies on 2 fundamental objects: a launch configuration and an autoscaling group. the very basis for autoscaling is the launch configuration and is required for autoscaling. 

A launch configuration is the blue print for what an autoscaling group will launch when it needs to scale.  At a minimum a launch config defines the emi, the vm type and name of the launch config. Once a launch configuration is created it can be used as the base for an autoscaling group.

Among other things, an autoscaling group defines an availability zone,  minimum number of instances, maximum number of instances, desired capacity, the launch configuration to use when launching new instances and name.

Autoscaling policies describe different modes of scaling and can be executed manually or triggered based on cloud watch metrics. Different types of scaling policies are **ChangeInCapacity** increases or decreases current capacity by a given number of instances, **PercentChangeInCapacity** changes capacity by a percentage of current capacity. For instance PercentChangeInCapacity of 0.5 when there were 4 instances in the group would terminate 2 instances such that the result would be 2 running instances in the group. **ExactCapacity** will change the capacity to an exact number of instances. 

Scaling can be done by increasing or decreasing the desired capacity of the autoscaling group, by executing a policy manually or alarmed based trigger of policy execution. 
 
##### What actions/processes are happening in the background at all times? 
At all times an autoscaling group is monitoring the health of its instances and if enabled instances are submitting metrics to CloudWatch. If at any time an instance in an autoscaling group becomes unhealthy it will be replaced. Also if there is any scaling action then instances will be launched or terminated based on the scaling type.  Scaling can be configured to allow for **cooldown** times, amount of time, in seconds, after a scaling activity completes before any further trigger-related scaling activities may start. Another important feature of autoscaling is **grace period**. A grace period is the number of seconds to wait before starting health checks on newly-created instances.

#### User level operation and tooling
##### How do end users interact with the feature?
Users can interact with AutoScaling in a number of ways.  Euca2ools 3, EucaLobo, boto and AWS Java SDK to name a few.  The new Euca2ools 3 delivers a whole suite of commandline tooling for autoscaling.  Teh new autoscaling commands begin with the prefic "euscale-" pronounced "you scale"

#### A few typical use cases for the feature end-to-end
1. Create a launch configuration:
> euscale-create-launch-config -i emi-24E73962 -t t1.micro --group securtiy-group-1 --key don-key My-LC
2. Create an autoscaling group (no instances will be launched in this example **yet**):
> euscale-create-auto-scaling-group -l My-LC --min-size 0 --max-size 5 -z PARTI00 My-ASG
3. Set desired capacity of the ASG (this will launch instances)
> euscale-set-desired-capacity -c 2 My-ASG
4. You can see the scaling activity
>  [root@h-17 ~]# euscale-describe-scaling-activities
ACTIVITY	94633c90-0784-431f-8178-7030f47af331	2013-06-11T00:10:27.193Z	My-ASG	Successful

## Administrative Tasks
* How will the administrators configure the feature?
* How will admins monitor/create/delete resources from other accounts?

## Debugging through log messages
* Show in one of the use cases from above the log messages that are expected to be seen across the various components and how one can understand where an issue is stemming from.

## Gotchas
* This section should show any caveats or known bugs that will trip up users in the field.

*****
[[category.Training]]