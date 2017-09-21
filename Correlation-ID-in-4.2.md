

 **GOAL** Using the single command line, I want to find all ERROR logs related to a faulty ELB “xyz”.

 **CORRELATION ID** In 4.1., Correlation ID was introduced to help cloud admins to troubleshoot the clouds using the request ID found in the user’s API response.For instance, if a user’s attempt to run an instance fails, the cloud admins can gather the logs from the distributed services and find the errors that includes the request ID of the failed RunInstance. In the implementation, the format of correlation ID in the internal messages is changed so that the ID is pre-fixed with the request ID:



\[REQUEST-ID]::\[RANDOM-GUID]

The request ID is included in every log messages so that the tools can grep the distributed logs using the ID.

 **EXTENSION IN 4.2** I propose adding the support for correlating resources with internal messages and distributed tasks. This will allow the cloud-admins to troubleshoot the distributed components based on the resource names such as the autoscaling group and the ELB.Here’s the possible scenario that the extension would enable:

1) A cloud user (Alice) creates a new load-balancer.

2) After finding that the loadbalancer is not operating, Alice ask the cloud-admin (Bob) to troubleshoot the problem.

3) Bob would use ‘LogTracker’ tool for troubleshooting ([https://github.com/eucalyptus/logtrackers](https://github.com/eucalyptus/logtrackers)).

 _euca-req-history_  would give Bob the request ID that created the faulty ELB.

 _euca-req-track_  would return ALL of distributed logs that is associated with the request ID (hence the faulty ELB). We may limit the result to only a particular type of logs (ERROR).

 **IMPLEMENTATION** We will use the same format of correlation ID in the messages that ELB/ASG/CloudFormation dispatch to EC2/IAM/S3. The request ID would be initially from the request that created a resource (e.g., CreateLoadBalancer, CreateAutoScalingGroup). Then any internal service calls to manage the resource (e.g., launching an instance to scale up an autoscaling group, launching a servo VM for ELB) would include the request ID. We will have to persist the request ID that created the resource by adding a new column in the database tables (perhaps new column in AbstractPersistent.java)

 **WHAT’S DIFFERENT?** In 4.1., we correlated a user's EC2 request with the internal messages and logs. This was possible because most of the distributed sub-tasks (such as allocating an IP address or launching a VM) are instantiated immediately from the user’s request. We could troubleshoot a problem at a finer granularity in terms of an action than just a resource name (i.e., we could find issues in TerminateInstances, RunInstances, etc).



However it is not exactly the case with the higher-level services managing EC2/S3/IAM resources. For example, an autoscaling group can launch an instance when the autoscaling group is created, when the desired capacity was changed, or when enforcing a scaling policy. It would be not clear which user requests (e.g., CreateAutoscalingGroup, SetDesiredCapacity, etc) would be correlated with the RunVMInstance requests on NC. Instead, we can be certain that the name of logical resources (autoscaling group) can be correlated with the distributed tasks.

In addition, using the name would be convenient when the cloud-user refers the problem (e.g., “I have a problem with my ELB xyz”). One potential issue when correlating with resource name (as opposed to a single user request) is that the search result would be significantly large because it will include all of the logging history related to the resource.Filtering by log type (ERROR only) and sorting by recent time would help the cloud-admin to find the problem quickly.



One would argue that the proposed solution is no difference from searching logs with the resource name (e.g., grep “alice-elb” /var/log/eucalyptus/cloud-debug). However it is clearly different because distributed sub-tasks that is spawned while managing a resource have different names. For example, creating an ELB creates the VM instances (servo VM), security group, IAM roles, etc, and their names have no connection to the ELB that caused them to exist. We tag the resources to help the cloud admins to identify the related resources, but it would be much more convenient if we let them use the resource name directly when troubleshooting the problem (“find all ERROR logs related to ELB xyz” vs “describe-tags and finding ERROR logs for each type of resource”)

*****

[[category.confluence]] 
