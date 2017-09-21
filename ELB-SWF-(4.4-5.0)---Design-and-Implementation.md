
## Overview
The document presents the design and implementation details of refactored ELB that uses SWF as a workflow orchestration engine.


## FlowFramework - aspectJ weaving
ELB service uses FlowFramework for Java that is included in AWS sdk. In a nutshell, FlowFramework transforms workflows and activities written in Java into SWF decision/activity tasks. It also allows a client to manage the workflow's lifecycle. This usageis different from the implementation of CloudFormation that uses glisten as a SDK to interact with SWF service. FlowFramework defines a handful of annotation, such as <workflow> and <activities>, by which ELB service implements the workflows and the distributed activity tasks. For example, the followings are the interface definition and class implementation of the workflow that creates a load-balancer:


```
Workflow

@WorkflowRegistrationOptions(defaultExecutionStartToCloseTimeoutSeconds = 120, defaultTaskStartToCloseTimeoutSeconds = 60)

public interface CreateLoadBalancerWorkflow {

@Execute(name = "CreateLoadBalancer", version = "1.0")

void createLoadBalancer(final String accountId, final String loadbalancer, final String\[] availabilityZones);

}
```

```
public class CreateLoadBalancerWorkflowImplimplements CreateLoadBalancerWorkflow {
```
....


```
 public void createLoadBalancer(final String accountId, final String loadbalancer, final String\[] availabilityZones) {

admission = doAdmission(

client.createLbAdmissionControl(accountId, loadbalancer, availabilityZones));

roleName = client.iamRoleSetup(accountId, loadbalancer, admission);

instanceProfile = 

client.instanceProfileSetup(Promise.asPromise(accountId), Promise.asPromise(loadbalancer), roleName);

iamPolicy = 

client.iamPolicySetup(Promise.asPromise(accountId), Promise.asPromise(loadbalancer), roleName);

securityGroup = 

client.securityGroupSetup(accountId, loadbalancer, admission);

tag = client.createLbTagCreator(Promise.asPromise(accountId), Promise.asPromise(loadbalancer), getSecurityGroupId(securityGroup));

    }
```
}



In addition, the following is the interface definition and the class implementation of the activities that's invoked by the workflow above.


```
@Activities(version="1.0")

@ActivityRegistrationOptions(

    defaultTaskHeartbeatTimeoutSeconds = FlowConstants.NONE, defaultTaskScheduleToCloseTimeoutSeconds = 180,

    defaultTaskScheduleToStartTimeoutSeconds = 120, defaultTaskStartToCloseTimeoutSeconds = 60)

public interface LoadBalancingActivities {

 boolean createLbAdmissionControl(String accountNumber, String lbName, String\[] zones) throws LoadBalancingActivityException;

    String iamRoleSetup(String accountNumber, String lbName) throws LoadBalancingActivityException;

    String instanceProfileSetup(String accountNumber, String lbName, String roleName) throws LoadBalancingActivityException;

    String iamPolicySetup(String accountNumber, String lbName, String roleName) throws LoadBalancingActivityException;

    SecurityGroupSetupActivityResult securityGroupSetup(String accountNumber, String lbName) throws LoadBalancingActivityException;
```
}


```
public class LoadBalancingActivitiesImplimplements LoadBalancingActivities {
```

```
 @Override

 public boolean createLbAdmissionControl(final String accountNumber, final String lbName, final String\[] zones) throws LoadBalancingActivityException {
```
    .....

}



Before compilation, the annotations for FlowFramework should be pre-processed and the various Java stubs corresponding to workflow clients should be created. For example, the following is the code snippet that starts the new workflow to create a loadbalancer. The type ' _CreateLoadBalancerWorkflowClientExternal_ ' is created after the annotation is preprocessed using AWS build tools. 


```
final CreateLoadBalancerWorkflowClientExternal workflow =

    WorkflowClients.getCreateLbWorkflow();

workflow.createLoadBalancer(accountId, loadbalancer, availabilityZones.toArray(new String\[availabilityZones.size()]));
```
Pre-processing the workflow definitions is done with 'aws-swf-build-tools'. The example build.xml for pre-processing can be found in 'eucalyptus/clc/modules/loadbalancing/src/main/java/com/eucalyptus/loadbalancing/workflow'. (To run it, do 'ant compile' and it will generate client stubs in "generated" subdirectory).



AWS flow-framework for Java uses AspectJ weaving in order to transform the workflow implementation (in Java) to SWF decision/activity tasks. For example, in the workflow above, the method invocations in each line corresponds to SWF activity tasks and executing them sequentially (and in parallel in other workflows) is done by flow-framework using AspectJ weaving. We currently uses load-time weaving that requires adding 'javaagent=\[path to aspectjweaver.jar]' when JVM is started.


## Worker Configuration and Bootstrap
When ELB service is started, SWF workers for activity and decision tasks are started. WorkflowClient defined in the module 'simpleworkflow-common' is used and it is given the configuration for the workers. The configuration is defined as properties and its default values are as follows:


```
SWF_WORKFLOW_WORKER_CONFIG = "{ \"DomainRetentionPeriodInDays\": 1, \"PollThreadCount\": 4, \"MaximumPollRateIntervalMilliseconds\": 50 }";
```

```
SWF_ACTIVITY_WORKER_CONFIG = "{\"PollThreadCount\": 4, \"TaskExecutorThreadPoolSize\": 16, \"MaximumPollRateIntervalMilliseconds\": 50 }";
```
In the configuration, PollThreadCount determines the number of activity pollers that initiates a long-polling to SWF service. Since each long-poll currently holds a thread out of a limited thread-pool (both in client and service side thread-pool), setting this value too high can cause a significant performance issue.

During the bootstrap, task implementations annotated with <Loadbalancing.class> as the component class is loaded and registered to flow-framework. After the workers are started, they start polling for tasks continuously from the SWF service. The task polled has a tasklist = 'LoadbalancerTasks' in order to specify that the tasks should be scheduled to loadbalancing service (that is hosted on UFS).

The configuration and bootstrap of workers running inside the VMs are presented later when we discuss VM Workers.


## Workflows
ELB service's workflows can be classified into a few types:


*  **Resource management** 

    When ELB service receives the API call to create a loadbalancer, listener, or to enable an availability zone, the service eventually creates a set of resources to fulfill the service. For example, creating an ELB result in the creation of an autoscaling group, security rule, IAM roles, and more. The role of workflows in this type is to coordinate the creation or deletion of such resources across many services. When a workflow completes, the artifacts are the set of resources that are linked together to provide the service. The resource-creation tasks in the workflow often has a corresponding roll-back task. If a task failsduring the workflow execution, the workflow rolls back the tasks that has been executed up to the point of failure. The table presents the workflows in this type and the artifacts they create.



| Workflow | Artifacts | 
|  --- |  --- | 
| CreateLoadBalancerWorkflow | IAM role, IAM instance profile, IAM policy, Security Group, Ec2 Tag → LoadBalancer | 
| DeleteLoadBalancerWorkflow | (Delete the artifacts above) | 
| CreateLoadBalancerListenerWorkflow | IAM role policy (to allow downloading server cert. for HTTPS listener), Security group rule → LoadBalancerListener | 
| DeleteLoadBalancerListenerWorkflow | (Delete the artifacts above) | 
| EnableAvailabilityZoneWorkflow | Launch config, Autoscaling group | 
| DisableAvailabilityZoneWorkflow | (Delete the artifacts above) | 
| ModifyLoadBalancerAttributesWorkflow | IAM policy (to allow access logging to S3 bucket) | 




*  **State management** 

    Because ELB is implemented using distributed VMs that contains Haproxy as a software loadbalancer, the changes in the loadbalancer's state must be propagated between the ELB service (interfacing persistent DB) and the VMs. For example, if a listener is added to the loadbalancer, the new specification about the loadbalancer should be passed down to the VM so that haproxy can be re-configured with the new listener. The table illustrates the workflows and the summary of their work.



| Workflow | Summary | 
|  --- |  --- | 
| UpdateLoadBalancerWorkflow (1 per LB) | Propagates an ELB's specification containing listeners, instances, health check, and SSL certificate information, to the VMs. The workflow runs whenever there's a change in the loadbalancer (e.g., adding a listener). | 
| InstanceStatusWorkflow (1 per LB) | Each ELB VM polls the endpoint as defined in the ELB's health check. This workflow runs continuously and updates the VM's health-check result periodically. | 
| CloudWatchPutMetricWorkflow (1 per LB) | Similar to the instance status, each ELB VM continously collects the cloud-watch metrics from the ELB traffic. The workflow runs continously and collects those metrics periodically from the VMs. | 
| LoadBalancingServiceHealthCheckWorkflow (1 in the service) | Continuously monitor the ELB Vm's status (i.e., running/terminated) to keep it up to date with the state maintained in the service. For example, if an ELB VM is accidentally terminated, the change is updated to the service so that the service can replace it with a new VM. This workflow also do some clean-up tasks after a loadbalancer is deleted. | 


*  **Properties** 

    Updating ELB properties require changes to the resources. For example, changing the instance-type of the ELB VM requires changes to the launch configuration and the autoscaling groups for the existing loadbalancers. There is only one workflow in this type.



| Workflow | Summary | 
|  --- |  --- | 
| ModifyServicePropertiesWorkflow | Update the launch configuration and autoscaling group when one of the properties is changed:services.loadbalancing.worker.image services.loadbalancing.worker.init_script services.loadbalancing.worker.app_cookie_duration services.loadbalancing.worker.instance_type services.loadbalancing.worker.keyname services.loadbalancing.worker.ntp_server services.loadbalancing.vm_per_zone | 




## Task scheduling
Some workflows require scheduling a task to a specific ELB VM. For example, when UpdateLoadBalancerWorkflow runs, the workflow should schedule a specification about the loadbalancer to the VM that belongs to the loadbalancer. In another example, whenCloudWatchPutMetricWorkflow collects the cloud-watch metrics, it should collect the metrics from the VM that belongs to a specific loadbalancer. We use **Tasklist** to schedule an activity task to the specific VM. When SWF activity worker is started in the VM, the activity worker is given the instance ID as the tasklist. Then the worker continuously poll for activity tasks that match the instance ID as the tasklist. When a workflow should schedule a task to a VM, it sets the tasklist of the task to the instance ID.


## Workflow Lifecycle
The workflows in the type of <resource-creation> and <properties> are started when the originating API is called and they should close in one of the SWF-defined state (completed, failed, terminated, cancelled, or timed out).

Among the state management workflows,LoadBalancingServiceHealthCheckWorkflow starts when the ELB service is started and it runs indefinitely until the service stops. Because it monitors the other state management workflows and re-start them as necessary, it can be considered a parent workflow of all other workflows.

The rest of the state-magement workflows (InstanceStatusWorkflow, UpdateLoadBalancerWorkflow, CloudwatchPutMetricWorkflow) are started when a new loadbalancer is created and they run indefinitely until the loadbalancer is deleted. They run activities in every polling interval defined in the workflow implementation (30sec - 1 minute but subject to change).

Because continuous workflows can generate a large amount of history, the workflow closes after a certain number of polling has been performed and they start again using ContinueAsNewWorkflowExecution decision. The new workflow will use the same workflow ID and a unique run ID and the old workflow will close withWorkflowExecutionContinuedAsNew event.

The code snippet with the comments below illustrates the task scheduling and the continuous workflow execution.


```
public class CloudWatchPutMetricWorkflowImpl implements CloudWatchPutMetricWorkflow {

final LoadBalancingVmActivitiesClient vmClient = 

new LoadBalancingVmActivitiesClientImpl();

final LoadBalancingActivitiesClient client =

new LoadBalancingActivitiesClientImpl();

final CloudWatchPutMetricWorkflowSelfClient selfClient =

new CloudWatchPutMetricWorkflowSelfClientImpl(); // the client for invoking continueAsNewWorkflowExecution

private int MAX_PUT_PER_WORKFLOW = 10;  // number of polling per workflow

private final int PUT_PERIOD_SEC = 30;  // wait period between polling

@Override public void putCloudWatchMetric(final String accountId, final String loadbalancer) {    

task = new TryCatchFinally() {

@Override

 protected void doTry() throws Throwable {

       putCloudWatchMetricPeriodic(0);

      }



@Override

 protected void doFinally() throws Throwable {

if(exception.isReady() && exception.get())

return;

else if (task.isCancelRequested())

return;

else {

selfClient.getSchedulingOptions().setTagList(

              Lists.newArrayList(String.format("account:%s", accountId), 

                  String.format("loadbalancer:%s", loadbalancer)));  // run the workflow again with continueAsNewWorkflowExecution

selfClient.putCloudWatchMetric(accountId, loadbalancer);

        }  

      }

    };

  }



@Asynchronous 

 private void putCloudWatchMetricPeriodic(final int count, Promise<?>... waitFor) {

if (count >= MAX_PUT_PER_WORKFLOW) { // number of polling exceeds the limit

return;

    }

final Promise<List<String>> servoInstances = client.lookupServoInstances(this.accountId, this.loadbalancer);

    doPutMetric(count, servoInstances);

  }



@Asynchronous

 private void doPutMetric(final int count, final Promise<List<String>> servoInstances) {

final Map<String, Promise<String>> metrics = Maps.newHashMap();

final List<String> instances = servoInstances.get();

for(final String instanceId : instances) {

final ActivitySchedulingOptions scheduler =

new ActivitySchedulingOptions();

      scheduler.setTaskList(instanceId); // For scheduling, instance ID is set as the tasklist.

      scheduler.setScheduleToCloseTimeoutSeconds(10L);      metrics.put(instanceId, vmClient.getCloudWatchMetrics(scheduler)); // Activities will be performed on the designated VMs

    }



final Promise<Map<String, String>> metricMap = Promises.mapOfPromisesToPromise(metrics);

client.putCloudWatchMetrics(Promise.asPromise(this.accountId), Promise.asPromise(this.loadbalancer), metricMap); 

    // Activity is run to merge the metrics gathered from the VMs 

 client.putCloudWatchInstanceHealth(this.accountId, this.loadbalancer);
```

```
final WorkflowClock clock = contextProvider.getDecisionContext().getWorkflowClock();

final Promise<Void> timer = clock.createTimer(PUT_PERIOD_SEC);

    putCloudWatchMetricPeriodic(count+1, timer); 

    // the next recursive call is delayed until the timer expires. The timer is a decision task in SWF.

  }

}
```

## VM workers

### Credential & Permission
ELB VM uses a short-lived role credential that is created under the ELB system account. ELB service grants the limited permission to invoke SWF APIs that is required to run the activity worker. More details can be found in the [[arch analysis|ELB-SWF-ARCHITECTURAL-ANALYSIS]].


### Worker configuration & bootstrap
Currently the configuration for a VM activity worker is hard-coded (this may change in the future). The default configuration is found in WorkflowStandaloneClient.java under simpleworkflow-common module. They are as follows:


```
private String domain = "LoadbalancingDomain";

private int clientConnectionTimeout = 30000;

private int clientMaxConnections = 100;

private int domainRetentionPeriodInDays = 1;

private int pollThreadCount = 1;
```
Among the configurations, pollThreadCount requires the attention because increasing the count increases the number of pollers across all ELBs in the system. As we discussed above,  number of pollers can directly affect the scalability of SWF. Because a VM is assigned only a few activities in a minute (as part of state-management workflow), only 1 poll thread should be enough.

The worker is a standalone Java program (WorkflowStandaloneClient.java) that is executed during the bootstrapping of load-balancer-servo ([https://github.com/eucalyptus/load-balancer-servo/blob/dev-spark-swf/servo/swf_worker.py#L88](https://github.com/eucalyptus/load-balancer-servo/blob/dev-spark-swf/servo/swf_worker.py#L88)). The implementation of VM worker is in the 'loadbalancing' module (the same module for ELB service). The program is given the path to the loadbalancing.jar and the worker implementation is loaded and registered to flow-framework and flow-framework starts polling tasks from SWF service.

ActivitiesThere are three types of activity task that is run in the VM worker. Each activity task is merely forwarding each command to the load-balancer-servo (Python) process. For interprocess communication between the worker (in Java) and the load-balanacer-servo (in Python), we use Redis as a local IPC channel. Redis server is run automatically when the VM starts and it listens only on 127.0.0.1.


*  **setLoadBalancer** 

    Receives the specification about the loadbalancer including listeners, SSL certificates, backend instances, and health-check. Publish the specification (XML doc) to the Redis channel on localhost. load-balancer-servo reconfigures haproxy using the new spec.
*  **getInstanceStatus** 

    Receives the task without any data. Publish the command "GetCloudwatchMetrics" to the Redis channel, wait until load-balancer-servo publishes the result, and return the result.
*  **getCloudwatchMetrics** Similarly, receives the task without any data. Publish the command "GetInstanceStatus" to the Redis channel, wait until load-balancer-servo publishes the result, followed by returning the result to SWF.


## Scale Consideration
In the current implementation, the number of pollers (activity task + decision task) can be determined as follows:

Parameters:


* U: Number of UFS
* L: Number of loadbalancers
* V: Number of VMs per loadbalancer ( configurable via services.loadbalancing.worker.vm_per_zone )

Number of decision task pollers = 4\*U

Number of activity task pollers = 4\*U + L\*V

In other words, there are 4 decision and 4 activity task pollers for each UFS and each ELB VM runs 1 activity task poller.



The number of workflows that is running continuously in the system can be determined as follows:

Number of workflows = 1 + 3\*L (1 forLoadBalancingServiceHealthCheckWorkflow, and the rest of state-management workflow per loadbalancer)


## References
[[ELB/SWF ARCHITECTURAL ANALYSIS|ELB-SWF-ARCHITECTURAL-ANALYSIS]]













*****

[[category.confluence]] 
