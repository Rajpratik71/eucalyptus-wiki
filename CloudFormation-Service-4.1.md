This document details theCloudFormation (CF) service implementation (API) in Eucalyptus 4.1.

* [Scope](#scope)
  * [API](#api)
  * [Resource Services](#resource-services)
  * [IAM Support](#iam-support)
  * [Limits](#limits)
  * [Helper Scripts](#helper-scripts)
* [Installation / Configuration](#installation-/-configuration)
  * [Registration](#registration)
  * [Cloud Properties](#cloud-properties)
* [Administration](#administration)
  * [Resource Management](#resource-management)
  * [CloudFormation Debugging](#cloudformation-debugging)
* [API Details](#api-details)
  * [Actions](#actions)
  * [Template Functionality](#template-functionality)
  * [Parameters](#parameters)
  * [Resources](#resources)
  * [Outputs](#outputs)
  * [Functions and PseudoParameters](#functions-and-pseudoparameters)



## Scope
This section gives a high-level summary of the scope of theCloudFormation service implementation as of the 4.1 release.


### API
API areas that are not implemented for 4.1 are:


* Cost estimation
* Signalling resources
* Stack updating

Paging of responses is not implemented.

While Set/GetStackPolicy will record the policy and retrieve it, it is not used by the stack internally.


### Resource Services
The 4.1 release includes support for resources from the following services:


* Auto Scaling
* CloudWatch
* EC2 (including VPC)
* ELB
* IAM
* S3
* CloudFormation (Nested Stacks, and WaitCondition\[Handle]s)


### IAM Support
CloudFormationactions can be used in policy and will be enforced.

CloudFormationresource ARNs are supported.


### Limits
Cloud properties can be used to configure limits. A quota key is available for restricting the number of stacks that can be used per account.


*  _cloudformation:quota-stacknumber_ 


### Helper Scripts
Helper scripts such as  _cfn-init_  are not supported for 4.1.0


## Installation / Configuration

### Registration
CloudFormation is part of the UFS/API service group though it can also be separately registered. Manual registration may be required for an upgrade from 4.0.X. Manual registration requires SWF be registered as well.

Manual registration can be done via the following command:



| euca_conf --register-service -T CloudFormation -H 10.111.5.158 -N cfn | 
|  --- | 

Where 10.111.5.118 should be replaced by the host where the CloudFormation service resides.


### Cloud Properties
The following cloud properties are used withCloudFormation in 4.1:



| Property | Default Value | Description | 
|  --- |  --- |  --- | 
| cloudformation.autoscaling_group_deleted_max_delete_retry_secs | 300 | Timeout (in seconds) for deleting the autoscaling group after reducing instance count to 0 during delete of AWS::AutoScaling::AutoScalingGroup | 
| cloudformation.autoscaling_group_zero_instances_max_delete_retry_secs | 300 | Timeout (in seconds) for reducing instance count to 0 during delete of AWS::AutoScaling::AutoScalingGroup | 
| cloudformation.instance_attach_volume_max_create_retry_secs | 300 | Timeout (in seconds) for attaching a volume to an instance during creation of AWS::EC2::Instance | 
| cloudformation.instance_running_max_create_retry_secs | 300 | Timeout (in seconds) for an instance to be 'running' during creation of AWS::EC2::Instance | 
| cloudformation.instance_terminated_max_delete_retry_secs | 300 | Timeout (in seconds) for an instance to be 'running' during creation of AWS::EC2::Instance | 
| cloudformation.max_attributes_per_mapping | 30 | Limit. The maximum number of attributes within a 'Mapping' section of a template. | 
| cloudformation.max_mappings_per_template | 100 | Limit. The maximum number of 'Mapping' elements within a 'Mappings' section of the template. | 
| cloudformation.max_outputs_per_template | 60 | Limit. The maximum number of 'Outputs' within a template. | 
| cloudformation.max_parameters_per_template | 60 | Limit. The maximum number of 'Parameters' within a template. | 
| cloudformation.max_resources_per_template | 200 | Limit. The maximum number of 'Resources' within a template. | 
| cloudformation.region |  | Value returned by the {"Ref":"AWS::Region"} call, and used in ARNs created by cloudformation. Should be set to the region name for the cloud, and match region section of ARNs Default is empty string. | 
| cloudformation.request_template_body_max_length_bytes | 51200 | Limit. Maximum length of a template whose body is embedded within the initial request (using the --template-body parameter in euca2ools) | 
| cloudformation.request_template_url_max_content_length_bytes | 460800 | Limit. Maximum length of the content of the template whose URL is passed in in the initial request (using the --template-url parameter is euca2ools) | 
| cloudformation.security_group_max_delete_retry_secs | 300 | Timeout (in seconds) to delete a security group during delete of AWS::EC2::SecurityGroup | 
| cloudformation.swf_activity_worker_config |  | A JSON value used to configure the ActivityWorkers in the internal SWF workflows. | 
| cloudformation.swf_client_config | {"ConnectionTimeout": 10000, "MaxConnections": 100} | A JSON value used to configure the SWF client in the internal SWF workflows. | 
| cloudformation.swf_domain | CloudFormationDomain | Name of the 'Domain' used by the internal SWF workflows | 
| cloudformation.swf_tasklist | CloudFormationTaskList | Name of the 'TaskList' used for workflows in the internal SWF workflows. | 
| cloudformation.swf_workflow_worker_config | { "DomainRetentionPeriodInDays": 1, "PollThreadCount": 8 } | A JSON value used to configure the WorkflowWorkers in the internal SWF workflows. | 
| cloudformation.url_domain_whitelist | \*[s3.amazonaws.com](http://s3.amazonaws.com) | Comma delimited list of hosts that are allowed for values of --template-url objects. \* and ? wildcards are supported. (? is used as a single character wildcard, not query strings). Supported protocols are http and https. http:// and https:// prefixes can be used in the hosts to limit which protocols can be used.Examples:\* (any http or https request to any host)http://\* (any http request to any host)https://\* (any https request to any host)\*.google.com (any http or https request to any host that ends in ".google.com"[http://www.\*.com](http://www.*.com) (any http request to any host that start with 'www' and ends with '.com'\*.com,\*.net (any http or https request to any .net or .com host)Note: Eucalyptus S3 urls will always be allowed. | 
| cloudformation.volume_attachment_max_create_retry_secs | 300 | Timeout (in seconds) for a volume to attach to an instance during creation of AWS::EC2::VolumeAttachment | 
| cloudformation.volume_available_max_create_retry_secs | 300 | Timeout (in seconds) for an instance to be 'available' during creation of AWS::EC2::Volume | 
| cloudformation.volume_deleted_max_delete_retry_secs | 300 | Timeout (in seconds) for an instance to be deleted during delete of AWS::EC2::Volume | 
| cloudformation.volume_detachment_max_delete_retry_secs | 300 | Timeout (in seconds) for a volume to detach to an instance during delete of AWS::EC2::VolumeAttachment | 
| cloudformation.volume_snapshot_complete_max_delete_retry_secs | 300 | Timeout (in seconds) for a snapshot to be created if DeletionPolicy is 'Snapshot' for a volume during delete of AWS::EC2::Volume | 
| cloudformation.wait_condition_bucket_prefix | cf-waitcondition | Name of the S3 Bucket name prefixes used by CloudFormation for wait conditions. (These buckets are owned by the eucalyptus-cloudformation account) | 


## Administration

### Resource Management
The resource administrator can list and delete stacks and describe most resources.



| Action | Notes | 
|  --- |  --- | 
| DeleteStack | Deleting a non-existent stack returns no error. | 
| DescribeStackEvents |  | 
| DescribeStackResource |  | 
| DescribeStackResources |  | 
| DescribeStacks | Use "verbose" as stack name to list from other accounts | 
| GetStackPolicy | Will returh the policy, but the policy is not used. | 
| GetTemplate |  | 
| ListStackResources |  | 
| ListStacks |  | 
| SetStackPolicy | Will set the policy, but the policy is not used. | 

Where a name or stack identifier is permitted a stack identifier must be used if the resource is owned by another account.


### CloudFormation Debugging
Certain template errors may occur and be reported back synchronously during service calls (such as CreateStack), such as template syntax errors. You may see errors of the form below:



| euform-create-stack --template-file example.template example euform-create-stack: error (ValidationError): Unexpected end-of-input: expected close marker for OBJECT (from \[Source: [java.io](http://java.io).StringReader@56b3916d; line: 1, column: 0]) at \[Source: [java.io](http://java.io).StringReader@56b3916d; line: 38, column: 849] | 
|  --- | 

These errors can be diagnosed by careful examination of the template.



Other errors may occur during stack create or delete, anything from too many resources, or unable to delete resources based on underlying dependencies. Use  _euform-describe-stacks_  to determine the status of a stack. By default if a stack is in CREATE_FAILED state, rollback will be attempted soon after. Stack status values of concern are: CREATE_FAILED, ROLLBACK_COMPLETE, ROLLBACK_IN_PROGRESS, ROLLBACK_FAILED in the create case, and  _DELETE_FAILED_  in the delete case. 

Here is a simple example of creating an instance using a non-existent image id. Running the stack to completion gives the following output from  _euform-describe-stacks_ 



| euform-describe-stacksSTACK mystack CREATE_FAILED The following resource(s) failed to create: \[MyInstance].   2015-01-20T19:54:08.269Z | 
|  --- | 

After about a minute, the stack will roll back and you would see the following output instead.



| euform-describe-stacksSTACK mystack ROLLBACK_COMPLETE    2015-01-20T19:54:08.269Z | 
|  --- | 

Either way, the best way to diagnose this issue is to run the  _euform-describe-stack-events_  command. Look for  _CREATE_FAILED_  events.



| euform-describe-stack-events mystackEVENT mystack 07e39520-f2d1-41de-9d86-829b1858d4f5 AWS::CloudFormation::Stack mystack arn:aws:cloudformation::221642028586:stack/mystack/b5f7f6fc-b619-4c47-ba51-5b994964fa4c 2015-01-20T19:54:53.716Z ROLLBACK_COMPLETE EVENT mystack MyInstance-DELETE_COMPLETE-1421783693175 AWS::EC2::Instance MyInstance  2015-01-20T19:54:53.175Z DELETE_COMPLETE EVENT mystack MyInstance-DELETE_IN_PROGRESS-1421783691283 AWS::EC2::Instance MyInstance  2015-01-20T19:54:51.283Z DELETE_IN_PROGRESS EVENT mystack c6463c29-3999-49ba-a35e-2ef5cbac4737 AWS::CloudFormation::Stack mystack arn:aws:cloudformation::221642028586:stack/mystack/b5f7f6fc-b619-4c47-ba51-5b994964fa4c 2015-01-20T19:54:50.233Z ROLLBACK_IN_PROGRESS Create stack failed. Rollback requested by user.EVENT mystack 62f33aa3-df8c-4feb-a70a-8c8488dbaeaf AWS::CloudFormation::Stack mystack arn:aws:cloudformation::221642028586:stack/mystack/b5f7f6fc-b619-4c47-ba51-5b994964fa4c 2015-01-20T19:54:12.537Z CREATE_FAILED The following resource(s) failed to create: \[MyInstance].EVENT mystack MyInstance-CREATE_FAILED-1421783651194 AWS::EC2::Instance MyInstance  2015-01-20T19:54:11.194Z CREATE_FAILED Failed to lookup image named: emi-abcdefgEVENT mystack MyInstance-CREATE_IN_PROGRESS-1421783650246 AWS::EC2::Instance MyInstance  2015-01-20T19:54:10.246Z CREATE_IN_PROGRESS EVENT mystack f1aec74e-6355-42c6-a430-973f5a3765c8 AWS::CloudFormation::Stack mystack arn:aws:cloudformation::221642028586:stack/mystack/b5f7f6fc-b619-4c47-ba51-5b994964fa4c 2015-01-20T19:54:08.847Z CREATE_IN_PROGRESS User Initiated | 
|  --- | 

Note the  _CREATE_FAILED_  event for MyInstance, you can see the root failure "Failed to lookup image named: emi-abcdefg".

Using  _euform-describe-stack-events_  and looking for  _CREATE_FAILED_  or  _DELETE_FAILED_  events is the best way to diagnose stack failures.



In other circumstances, things might go wrong that happen outside a create stack event or similar, or more detail might be required. Checking the  _cloud-output.log_ file for errors is the best bet then. Since CloudFormation uses the Netflix Glisten library, and no other service currently does, and since CloudFormation prints a stack trace on most errors, looking for the phrase "netflix" or "glisten" in the log might narrow down search time. Here is the result of the above error in the log.



| 2015-01-20 11:54:12 DEBUG 000001593 CreateStackWorkflowImpl | com.amazonaws.services.simpleworkflow.flow.ActivityTaskFailedException: com.eucalyptus.compute.ClientComputeException:Failed to lookup image named: emi-abcdefg for activityId="4" of activityType={Name: StackActivity.performCreateStep,Version: 1.0}com.amazonaws.services.simpleworkflow.flow.ActivityTaskFailedException: com.eucalyptus.compute.ClientComputeException:Failed to lookup image named: emi-abcdefg for activityId="4" of activityType={Name: StackActivity.performCreateStep,Version: 1.0} at com.amazonaws.services.simpleworkflow.flow.worker.GenericActivityClientImpl.handleActivityTaskFailed(GenericActivityClientImpl.java:192) at com.amazonaws.services.simpleworkflow.flow.worker.AsyncDecider.processEvent(AsyncDecider.java:188) at com.amazonaws.services.simpleworkflow.flow.worker.AsyncDecider.decide(AsyncDecider.java:496) at com.amazonaws.services.simpleworkflow.flow.worker.AsyncDecisionTaskHandler.handleDecisionTask(AsyncDecisionTaskHandler.java:50) at com.amazonaws.services.simpleworkflow.flow.worker.DecisionTaskPoller.pollAndProcessSingleTask(DecisionTaskPoller.java:201) at com.amazonaws.services.simpleworkflow.flow.worker.GenericWorker$PollServiceTask.run(GenericWorker.java:94) at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145) at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615) at java.lang.Thread.run(Thread.java:745) at --- continuation ---.(:0) at com.amazonaws.services.simpleworkflow.flow.worker.GenericActivityClientImpl.scheduleActivityTask(GenericActivityClientImpl.java:103) at com.amazonaws.services.simpleworkflow.flow.DynamicActivitiesClientImpl$2.doTry(DynamicActivitiesClientImpl.java:120) at --- continuation ---.(:0) at com.amazonaws.services.simpleworkflow.flow.DynamicActivitiesClientImpl.scheduleActivity(DynamicActivitiesClientImpl.java:101) at com.amazonaws.services.simpleworkflow.flow.DynamicActivitiesClientImpl$1.doExecute(DynamicActivitiesClientImpl.java:91) at --- continuation ---.(:0) at com.amazonaws.services.simpleworkflow.flow.core.Functor.<init>(Functor.java:22) at com.amazonaws.services.simpleworkflow.flow.DynamicActivitiesClientImpl$1.<init>(DynamicActivitiesClientImpl.java:82) at com.amazonaws.services.simpleworkflow.flow.DynamicActivitiesClientImpl.scheduleActivity(DynamicActivitiesClientImpl.java:82) at sun.reflect.GeneratedMethodAccessor1579.invoke(Unknown Source) at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) at java.lang.reflect.Method.invoke(Method.java:606) at org.codehaus.groovy.runtime.callsite.PojoMetaMethodSite$PojoCachedMethodSite.invoke(PojoMetaMethodSite.java:189) at org.codehaus.groovy.runtime.callsite.PojoMetaMethodSite.call(PojoMetaMethodSite.java:53) at com.netflix.glisten.impl.swf.AsyncCaller.methodMissing(AsyncCaller.groovy:61)... | 
|  --- | 

Note: These stack traces occur at DEBUG.

Finally, there may be times there is an issue with the underlying SWF implementation. This may cause things like stacks to make no progress during delete or create.

CloudFormation uses SWF for stack workflows. You can use the AWS CLI to check the underlying SWF resources for a stack:


```bash
# aws --endpoint-url http://simpleworkflow.b-38.autoqa.qa1.eucalyptus-systems.com:8773/ swf list-closed-workflow-executions --domain CloudFormationDomain --type-filter name=CreateStackWorkflow.createStack --tag-filter tag=StackName:EUCA10020VPCStack
{
    "executionInfos": [
        {
            "closeTimestamp": 1421115791.243,
            "workflowType": {
                "version": "1.0",
                "name": "CreateStackWorkflow.createStack"
            },
            "startTimestamp": 1421115768.674,
            "closeStatus": "COMPLETED",
            "executionStatus": "CLOSED",
            "execution": {
                "workflowId": "3dc50170-c488-4fe0-ada5-a4712de25cb0",
                "runId": "253b0186-5007-473f-9feb-c947989f5b48"
            },
            "cancelRequested": false,
            "tagList": [
                "StackId:arn:aws:cloudformation::638394063390:stack/EUCA10020VPCStack/487d2374-41d8-4b81-b943-867ae02e10d3",
                "StackName:EUCA10020VPCStack",
                "AccountId:638394063390",
                "AccountName:eucalyptus"
            ]
        }
    ]
}
#
#
# aws --endpoint-url http://simpleworkflow.b-38.autoqa.qa1.eucalyptus-systems.com:8773/ swf get-workflow-execution-history --domain CloudFormationDomain --execution runId=253b0186-5007-473f-9feb-c947989f5b48,workflowId=3dc50170-c488-4fe0-ada5-a4712de25cb0
{
    "events": [
        {
            "eventId": 1,
            "eventType": "WorkflowExecutionStarted",
            "workflowExecutionStartedEventAttributes": {
                "taskList": {
                    "name": "CloudFormationTaskList"
                },
                "parentInitiatedEventId": 0,
                "taskStartToCloseTimeout": "30",
                "childPolicy": "TERMINATE",
                "executionStartToCloseTimeout": "10800",
                "input": "[\"[Ljava.lang.Object;\",[\"arn:aws:cloudformation::638394063390:stack/EUCA10020VPCStack/487d2374-41d8-4b81-b943-867ae02e10d3\",\"638394063390\",\"{\\\"nodes\\\":\\\"[\\\\\\\"VPC\\\\\\\",\\\\\\\"InternetGateway\\\\\\\",\\\\\\\"Subnet\\\\\\\",\\\\\\\"AttachGateway\\\\\\\",\\\\\\\"IPAddress\\\\\\\",\\\\\\\"RouteTable\\\\\\\",\\\\\\\"Route\\\\\\\",\\\\\\\"NetworkAcl\\\\\\\",\\\\\\\"NetworkAclEntry\\\\\\\",\\\\\\\"DhcpOptions\\\\\\\",\\\\\\\"SubnetNetworkAclAssociation\\\\\\\",\\\\\\\"SubnetRouteTableAssociation\\\\\\\",\\\\\\\"VPCDHCPOptionsAssociation\\\\\\\",\\\\\\\"SecurityGroup\\\\\\\",\\\\\\\"NetworkInterface\\\\\\\"]\\\",\\\"dependencies\\\":\\\"{\\\\\\\"AttachGateway\\\\\\\":[\\\\\\\"IPAddress\\\\\\\"],\\\\\\\"DhcpOptions\\\\\\\":[\\\\\\\"VPCDHCPOptionsAssociation\\\\\\\"],\\\\\\\"InternetGateway\\\\\\\":[\\\\\\\"AttachGateway\\\\\\\",\\\\\\\"Route\\\\\\\"],\\\\\\\"NetworkAcl\\\\\\\":[\\\\\\\"NetworkAclEntry\\\\\\\",\\\\\\\"SubnetNetworkAclAssociation\\\\\\\"],\\\\\\\"RouteTable\\\\\\\":[\\\\\\\"Route\\\\\\\",\\\\\\\"SubnetRouteTableAssociation\\\\\\\"],\\\\\\\"SecurityGroup\\\\\\\":[\\\\\\\"NetworkInterface\\\\\\\"],\\\\\\\"Subnet\\\\\\\":[\\\\\\\"NetworkInterface\\\\\\\",\\\\\\\"SubnetNetworkAclAssociation\\\\\\\",\\\\\\\"SubnetRouteTableAssociation\\\\\\\"],\\\\\\\"VPC\\\\\\\":[\\\\\\\"AttachGateway\\\\\\\",\\\\\\\"NetworkAcl\\\\\\\",\\\\\\\"RouteTable\\\\\\\",\\\\\\\"SecurityGroup\\\\\\\",\\\\\\\"Subnet\\\\\\\",\\\\\\\"VPCDHCPOptionsAssociation\\\\\\\"]}\\\"}\",\"AID0DJHR6JLDSMKV3ICFY\",\"ROLLBACK\"]] ",
                "workflowType": {
                    "version": "1.0",
                    "name": "CreateStackWorkflow.createStack"
                },
                "tagList": [
                    "StackId:arn:aws:cloudformation::638394063390:stack/EUCA10020VPCStack/487d2374-41d8-4b81-b943-867ae02e10d3",
                    "StackName:EUCA10020VPCStack",
                    "AccountId:638394063390",
                    "AccountName:eucalyptus"
                ]
            },
            "eventTimestamp": 1421115768.674
        },
        {
            "eventId": 2,
            "eventType": "DecisionTaskScheduled",
            "decisionTaskScheduledEventAttributes": {
                "startToCloseTimeout": "30",
                "taskList": {
                    "name": "CloudFormationTaskList"
                }
            },
            "eventTimestamp": 1421115768.674
        },
        {
            "eventId": 3,
            "eventType": "DecisionTaskStarted",
            "eventTimestamp": 1421115768.73,
            "decisionTaskStartedEventAttributes": {
                "scheduledEventId": 2,
                "identity": "14071@b-38.qa1.eucalyptus-systems.com"
            }
        },
        {
            "eventId": 4,
            "eventType": "DecisionTaskCompleted",
            "decisionTaskCompletedEventAttributes": {
                "startedEventId": 3,
                "scheduledEventId": 2
            },
            "eventTimestamp": 1421115769.0139999
        },
        {
            "eventId": 5,
            "eventType": "ActivityTaskScheduled",
            "activityTaskScheduledEventAttributes": {
                "input": "[\"[Ljava.lang.Object;\",[\"arn:aws:cloudformation::638394063390:stack/EUCA10020VPCStack/487d2374-41d8-4b81-b943-867ae02e10d3\",\"638394063390\",\"CREATE_IN_PROGRESS\",\"User Initiated\"]] ",
                "activityId": "1",
                "activityType": {
                    "version": "1.0",
                    "name": "StackActivity.createGlobalStackEvent"
                },
                "decisionTaskCompletedEventId": 4
            },
            "eventTimestamp": 1421115769.0139999
        },
....
```
Note that SWF resources for CloudFormation are owned by the CloudFormation service, not by the account that owns the stack


## API Details

### Actions
CloudFormation actions are implemented as follows in 4.1:



| Action | Implemented | Notes | 
|  --- |  --- |  --- | 
| CancelUpdateStack | Y |  | 
| CreateStack | Y |  | 
| DeleteStack | Y |  | 
| DescribeStackEvents | Y |  | 
| DescribeStackResource | Y |  | 
| DescribeStackResources | Y |  | 
| DescribeStacks | Y |  | 
| EstimateTemplateCost |  |  | 
| GetStackPolicy | Y |  | 
| GetTemplate | Y |  | 
| GetTemplateSummary |  | Not stubbed | 
| ListStackResources | Y |  | 
| ListStacks | Y |  | 
| SetStackPolicy | Y |  | 
| SignalResource |  | Not stubbed | 
| UpdateStack | Y |  | 
| ValidateTemplate | Y |  | 


### Template Functionality
What follows in the next few sections, namely the listing of all supported values are done because there is only one API version of CloudFormation at AWS and new functionality is continually being added. The lists here are given to show the current level of support at Eucalyptus in as much detail as possible.

The following sections are supported within templates:


* AWSTemplateFormatVersion
* Description
* Parameters
* Mappings
* Conditions
* Resources
* Outputs

AWSTemplateFormatVersion only supports a value of '2010-09-09'.


### Parameters
The following attributes are supported for parameters:


* Type
* Default
* NoEcho
* AllowedValues
* AllowedPattern
* MaxLength
* MinLength
* MaxValue
* MinValue
* Description
* ConstraintDescription

AWS-specific types for template parameters are not supported, the parameter types that can be used are:


* String
* Number
* CommaDelimitedList


### Resources
The following fields are supported for resources:


* Type
* Properties
* DeletionPolicy
* Description
* DependsOn
* Metadata
* UpdatePolicy
* Condition

UpdatePolicy is syntactically supported, but nothing is done with the values.

DeletionPolicy values are:


* Delete
* Retain
* Snapshot (only supported for AWS::EC2::Volume at the moment)

For Resource Types: all resources, properties, and attributes in the following table are the subset of the AWS CloudFormation values that are supported by Eucalyptus. Value types and meanings are the same as they are at AWS. Required properties are in  **bold** . Note: Some properties values that are passed in to the back end service may not be supported there.



| Resource | Properties | Attributes | Notes | 
|  --- |  --- |  --- |  --- | 
| AWS::AutoScaling::AutoScalingGroup | <ul><li> **AvailabilityZones** </li><li>Cooldown</li><li>DesiredCapacity</li><li>HealthCheckGracePeriod</li><li>HealthCheckType</li><li>InstanceId</li><li>LaunchConfigurationName</li><li>LoadBalancerNames</li><li> **MaxSize** </li><li> **MinSize** </li><li>NotificationConfiguration</li><li>Tags</li><li>TerminationPolicies</li><li>VPCZoneIdentifier</li></ul> |  | HealthCheckType is syntactically supported but never referenced.InstanceId is syntactically supported but not allowed by the back end, making LaunchConfigurationName a de-facto required field. | 
| AWS::AutoScaling::LaunchConfiguration | <ul><li>AssociatePublicIpAddress</li><li>BlockDeviceMappings</li><li>EbsOptimized</li><li>IamInstanceProfile</li><li> **ImageId** </li><li>InstanceId</li><li>InstanceMonitoring</li><li> **InstanceType** </li><li>KernelId</li><li>KeyName</li><li>RamDiskId</li><li>SecurityGroups</li><li>SpotPrice</li><li>UserData</li></ul> |  | AssociatePublicIpAddress is syntactically supported but never referenced. | 
| AWS::AutoScaling::ScalingPolicy | <ul><li> **AdjustmentType** </li><li> **AutoScalingGroupName** </li><li>Cooldown</li><li> **ScalingAdjustment** </li></ul> |  |  | 
| AWS::CloudFormation::Stack | <ul><li>NotificationARNs</li><li>Parameters</li><li> **TemplateURL** </li><li>TimeoutInMinutes</li></ul> | <ul><li>Outputs. _NestedStackOutput_ </li></ul>Where _NestedStackOutput_  is an output from the nested stack. | This resource represents a stack embedded within the calling stack.NotificationARNs are recorded, but not used internally.TemplateURL follows the same rules as in the CreateStack action. | 
| AWS::CloudFormation::WaitCondition | <ul><li>Count</li><li> **Handle** </li><li>Timeout</li></ul> | <ul><li>Data</li></ul> |  | 
| AWS::CloudFormation::WaitConditionHandle |  |  |  | 
| AWS::CloudWatch::Alarm | <ul><li>ActionsEnabled</li><li>AlarmActions</li><li>AlarmDescription</li><li>AlarmName</li><li> **ComparisonOperator** </li><li>Dimensions</li><li> **EvaluationPeriods** </li><li>InsufficientDataActions</li><li> **MetricName** </li><li> **Namespace** </li><li>OKActions</li><li> **Period** </li><li> **Statistic** </li><li> **Threshold** </li><li>Unit</li></ul> |  |  | 
| AWS::EC2::DHCPOptions | <ul><li>DomainName</li><li>DomainNameServers</li><li>NetbiosNameServers</li><li>NetbiosNodeType</li><li>NtpServers</li><li>Tags</li></ul> |  |  | 
| AWS::EC2::EIP | <ul><li>InstanceId</li><li>Domain</li></ul> | <ul><li>AllocationId</li></ul> |  | 
| AWS::EC2::EIPAssociation | <ul><li>AllocationId</li><li>EIP</li><li>InstanceId</li><li>NetworkInterfaceId</li><li>PrivateIpAddress</li></ul> |  | In VPC mode, AllocationId or EIP is required.In EC2-Classic mode, EIP is required.Either NetworkInterfaceId or InstanceId is required. | 
| AWS::EC2::Instance | <ul><li>AvailabilityZone</li><li>BlockDeviceMappings</li><li>DisableApiTermination</li><li>EbsOptimized</li><li>IamInstanceProfile</li><li> **ImageId** </li><li>InstanceInitiatedShutdownBehavior</li><li>InstanceType</li><li>KernelId</li><li>KeyName</li><li>Monitoring</li><li>NetworkInterfaces</li><li>PlacementGroupName</li><li>PrivateIpAddress</li><li>RamdiskId</li><li>SecurityGroupIds</li><li>SecurityGroups</li><li>SourceDestCheck</li><li>SubnetId</li><li>Tags</li><li>Tenancy</li><li>UserData</li><li>Volumes</li></ul> | <ul><li>AvailabilityZone</li><li>PrivateDnsName</li><li>PublicDnsName</li><li>PrivateIp</li><li>PublicIp</li></ul> | InstanceInitiatedShutdownBehavior and SourceDestCheck are syntactically supported but never referenced. | 
| AWS::EC2::InternetGateway | <ul><li>Tags</li></ul> |  |  | 
| AWS::EC2::NetworkAcl | <ul><li> **VpcId** </li><li>Tags</li></ul> |  |  | 
| AWS::EC2::NetworkAclEntry | <ul><li> **CidrBlock** </li><li>Egress</li><li>Icmp</li><li> **NetworkAclId** </li><li>PortRange</li><li> **Protocol** </li><li> **R**  **uleAction** </li><li> **RuleNumber** </li></ul> |  | According to AWS documentation, Egress is required, but it defaults to false if not present in AWS and Eucalyptus maintains that behavior. | 
| AWS::EC2::NetworkInterface | <ul><li>Description</li><li>GroupSet</li><li>PrivateIpAddress</li><li>PrivateIpAddresses</li><li>SecondaryPrivateIpAddressCount</li><li>SourceDestCheck</li><li> **SubnetId** </li><li>Tags</li></ul> | <ul><li>PrimaryPrivateIpAddress</li><li>SecondaryPrivateIpAddresses</li></ul> | Even though properties and attributes referencing secondary ip addresses are supported syntactically, Eucalyptus does not support more than one IP address per network interface, so such values will be empty or ignored.SourceDestCheck is syntactically supported but never referenced. | 
| AWS::EC2::Route | <ul><li> **DestinationCidrBlock** </li><li>GatewayId</li><li>InstanceId</li><li>NetworkInterfaceId</li><li> **RouteTableId** </li><li>VpcPeeringConnectionId</li></ul> |  |  | 
| AWS::EC2::RouteTable | <ul><li> **VpcId** </li><li>Tags</li></ul> |  |  | 
| AWS::EC2::SecurityGroup | <ul><li> **GroupDescription** </li><li>SecurityGroupEgress</li><li>SecurityGroupIngress</li><li>VpcId</li><li>Tags</li></ul> | <ul><li>GroupId</li></ul> |  | 
| AWS::EC2::SecurityGroupIngress | <ul><li>CidrIp</li><li>GroupName</li><li>GroupId</li><li> **IpProtocol** </li><li>SourceSecurityGroupName</li><li>SourceSecurityGroupId</li><li>SourceSecurityGroupOwnerId</li><li>FomPort</li><li>ToPort</li></ul> |  |  | 
| AWS::EC2::SecurityGroupEgress | <ul><li>CidrIp</li><li>DestinationSecurityGroupId</li><li>FromPort</li><li> **GroupId** </li><li> **IpProtocol** </li><li>ToPort</li></ul> |  |  | 
| AWS::EC2::Subnet | <ul><li>AvailabilityZone</li><li> **CidrBlock** </li><li>Tags</li><li> **VpcId** </li></ul> | <ul><li>AvailabilityZone</li></ul> |  | 
| AWS::EC2::SubnetNetworkAclAssociation | <ul><li> **SubnetId** </li><li> **NetworkAclId** </li></ul> |  |  | 
| AWS::EC2::SubnetRouteTableAssociation | <ul><li> **RouteTableId** </li><li> **SubnetId** </li></ul> |  |  | 
| AWS::EC2::Volume | <ul><li> **AvailabilityZone** </li><li>Iops</li><li>Size</li><li>SnapshotId</li><li>Tags</li><li>VolumeType</li></ul> |  |  | 
| AWS::EC2::VolumeAttachment | <ul><li> **Device** </li><li> **InstanceId** </li><li> **VolumeId** </li></ul> |  |  | 
| AWS::EC2::VPC | <ul><li> **CidrBlock** </li><li>EnableDnsSupport</li><li>EnableDnsHostnames</li><li>InstanceTenancy</li><li>Tags</li></ul> |  |  | 
| AWS::EC2::VPCDHCPOptionsAssociation | <ul><li> **DhcpOptionsId** </li><li> **VpcId** </li></ul> |  |  | 
| AWS::EC2::VPCGatewayAttachment | <ul><li> **InternetGatewayId** </li><li>VpcId</li><li>VpnGatewayId</li></ul> |  |  | 
| AWS::ElasticLoadBalancing::LoadBalancer | <ul><li>AccessLoggingPolicy</li><li>AppCookieStickinessPolicy</li><li>AvailabilityZones</li><li>ConnectionDrainingPolicy</li><li>CrossZone</li><li>HealthCheck</li><li>Instances</li><li>LBCookieStickinessPolicy</li><li>LoadBalancerName</li><li> **Listeners** </li><li>Policies</li><li>Scheme</li><li>SecurityGroups</li><li>Subnets</li></ul> | <ul><li>CanonicalHostedZoneName</li><li>CanonicalHostedZoneNameID</li><li>DNSName</li><li>SourceSecurityGroup.GroupName</li><li>SourceSecurityGroup.OwnerAlias</li></ul> | AccessLoggingPolicy, ConnectionDrainingPolicy and CrossZone are syntactically supported but never referenced. | 
| AWS::IAM::AccessKey | <ul><li>Serial</li><li>Status</li><li> **UserName** </li></ul> | <ul><li>SecretAccessKey</li></ul> | AWS documentation states that Status is a required property but examples given in that same document omit it, and testing verifies that. Status appears to be 'Active' by default. | 
| AWS::IAM::Group | <ul><li>Path</li><li>Policies</li></ul> | <ul><li>Arn</li></ul> |  | 
| AWS::IAM::InstanceProfile | <ul><li> **Path** </li><li> **Roles** </li></ul> | <ul><li>Arn</li></ul> |  | 
| AWS::IAM::Policy | <ul><li>Groups</li><li> **PolicyDocument** </li><li> **PolicyName** </li><li>Roles</li><li>Users</li></ul> |  |  | 
| AWS::IAM::Role | <ul><li> **AssumeRolePolicyDocument** </li><li>Policies</li></ul> | <ul><li>Arn</li></ul> |  | 
| AWS::IAM::User | <ul><li>Path</li><li>Groups</li><li>LoginProfile</li><li>Policies</li></ul> | <ul><li>Arn</li></ul> |  | 
| AWS::IAM::UserToGroupAddition | <ul><li> **GroupName** </li><li> **Users** </li></ul> |  |  | 
| AWS::S3::Bucket | <ul><li>AccessControl</li><li>BucketName</li><li>CorsConfiguration</li><li>LifecycleConfiguration</li><li>LoggingConfiguration</li><li>NotificationConfiguration</li><li>Tags</li><li>VersioningConfiguration</li><li>WebsiteConfiguration</li></ul> | <ul><li>DomainName</li><li>WebsiteURL</li></ul> | Even though WebsiteURL is a supported attribute, static web hosting is not supported in buckets. | 


### Outputs
The following parameters are supported for Outputs:


* Description
* Condition
* Value


### Functions and PseudoParameters
The following Cloudformation template functions are supported:


* Ref
* Condition
* Fn::And
* Fn::Equals
* Fn::If
* Fn::Not
* Fn::Or
* Fn::Base64
* Fn::Select
* Fn::Join
* Fn::FindInMap
* Fn::GetAZs (values calculated at stack creation)
* Fn::GetAtt

For the "Ref" function, in addition to parameters and resources, the following pseudo-parameters are supported:


* AWS::AccountId
* AWS::NotificationARNs
* AWS::NoValue
* AWS::Region
* AWS::StackId
* AWS::StackName





*****

[[category.confluence]] 
