

This document details theSimple Workflow (SWF) service implementation (API) in Eucalyptus 4.1.

* [Scope](#scope)
  * [API](#api)
  * [IAM Support](#iam-support)
  * [Limits](#limits)
* [Installation / Configuration](#installation-/-configuration)
  * [Registration](#registration)
  * [Enabling SWF](#enabling-swf)
  * [SWF Cloud Properties](#swf-cloud-properties)
* [Administration](#administration)
  * [Resource Cleanup](#resource-cleanup)
* [API Details](#api-details)
  * [Actions](#actions)



## Scope
This section gives a high-level summary of the scope of the SWF service implementation in the 4.1 release.


### API
All API actions are implemented though not all functionality is present. The  _RespondDecisionTaskCompleted_  action covers a significant amount of functionality and does not implement:


* Continuing as a new workflow
* Starting a child workflow

Paging of responses is not implemented.


### IAM Support
SWF actions can be used in policy and will be enforced.

SWF resource ARNs are not supported.

SWF condition keys are not supported.


### Limits
Cloud properties can be used to configure resource limits for most resource types. A quota key is available for restricting the number of domains that can be used.


## Installation / Configuration

### Registration
Although SWF is a tech preview in 4.1 it is registered along with other UFS/API services as it is required for CloudFormation.


### Enabling SWF
By default the SWF service is registered but it only enabled for use by the system / cloud administrators. To enable use of SWF by regular users the  _services.simpleworkflow.systemonly_  cloud property must be set to  _false_ .


### SWF Cloud Properties
The following cloud properties are used with SWF in 4.1:



| Property | Default Value | Description | 
|  --- |  --- |  --- | 
| services.simpleworkflow.activitytypesperdomain | 10000 | Limit | 
| services.simpleworkflow.deprecatedactivitytyperetentionduration | 30d | Period of time to retain deprecated activity types | 
| services.simpleworkflow.deprecateddomainretentionduration | 1d | Period of time to retain deprecated domains | 
| services.simpleworkflow.deprecatedworkflowtyperetentionduration | 1d | Period of time to retain deprecated workflow types | 
| services.simpleworkflow.openactivitytasksperworkflowexecution | 1000 | Limit | 
| services.simpleworkflow.opentimersperworkflowexecution | 1000 | Limit | 
| services.simpleworkflow.openworkflowexecutionsperdomain | 100000 | Limit | 
| services.simpleworkflow.systemonly | true | Service available for internal/administrator use only. | 
| services.simpleworkflow.workflowexecutionduration | 365d | Maximum workflow execution time. | 
| services.simpleworkflow.workflowexecutionhistorysize | 25000 | Limit | 
| services.simpleworkflow.workflowexecutionretentionduration | 90d | Maximum workflow execution history retention time. | 
| services.simpleworkflow.workflowtypesperdomain | 10000 | Limit | 


## Administration

### Resource Cleanup
Unlike AWS/SWF deprecated resources are not retained indefinitely. Cloud properties can be configured to control when deprecated resources will be purged.


## API Details

### Actions
The following SWF actions are implemented in 4.1:



| Action | Notes | 
|  --- |  --- | 
| CountClosedWorkflowExecutions |  | 
| CountOpenWorkflowExecutions |  | 
| CountPendingActivityTasks |  | 
| CountPendingDecisionTasks |  | 
| DeprecateActivityType |  | 
| DeprecateDomain |  | 
| DeprecateWorkflowType |  | 
| DescribeActivityType |  | 
| DescribeDomain |  | 
| DescribeWorkflowExecution |  | 
| DescribeWorkflowType |  | 
| GetWorkflowExecutionHistory |  | 
| ListActivityTypes |  | 
| ListClosedWorkflowExecutions |  | 
| ListDomains |  | 
| ListOpenWorkflowExecutions |  | 
| ListWorkflowTypes |  | 
| PollForActivityTask |  | 
| PollForDecisionTask |  | 
| RecordActivityTaskHeartbeat |  | 
| RegisterActivityType |  | 
| RegisterDomain |  | 
| RegisterWorkflowType |  | 
| RequestCancelWorkflowExecution |  | 
| RespondActivityTaskCanceled |  | 
| RespondActivityTaskCompleted |  | 
| RespondActivityTaskFailed |  | 
| RespondDecisionTaskCompleted | Partially implemented | 
| SignalWorkflowExecution |  | 
| StartWorkflowExecution |  | 
| TerminateWorkflowExecution |  | 



*****

[[category.confluence]] 
