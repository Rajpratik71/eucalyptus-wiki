## Feature Overview
The CloudWatch service provides users a metrics database that can be used in order to track cloud resources (automatically collected by the system) or arbitrary time series data (added manually by the user). CloudWatch data returned from the server is limited to a minimum resolution of 60 seconds. Datapoints can be entered at arbitrary frequencies but when returned to the user all datapoints within the period will be aggregated. When entering a datapoint with the PutMetricData call the user must provide:
* Namespace
* Dimensions (can be empty)
* Start Time
* End Time
* Value

For each combination of Namespace+Dimensions a set of datapoints can be returned from the service with the following statistics pre-computed for the period using a GetMetricStatistics call:
* Average
* Maximum
* Minimum
* Sum
* SampleCount

The CloudWatch service is basically a way to enter data at arbitrary intervals and extract statistics about the data for a given time range with a user defined granularity.

The following metrics are collected automatically:
* EBS
* ELB
* AutoScaling

Monitoring EC2 instances can be enabled at runtime of an instance or post facto. Unlike AWS, Eucalyptus CloudWatch is disabled by default rather than giving metrics at less granularity by default and enabling "detailed monitoring" when enabled.
```
At runtime:
euca-run-instance emi-E6053B4B --monitor
Post Facto:
euca-monitor-instances i-E84B431F
euca-unmonitor-instances i-E84B431F
```

Alarms can be created that monitor the CloudWatch data for particular conditions. When an alarm state is triggered it can be used to kick an autoscaling policy that defines a scaling activity for a particular ASG.

#### Component level responsibilities
* Talk about what the involvement of each Eucalyptus component is wrt to the feature. 
The CloudWatch service is currently colocated with the ENABLED CLC. 

* What is the flow of a user request? 
Custom metrics put into the system are directly entered into the eucalyptus_cloudwatch database in the custom_metric_data_* tables. No other components are involved at that point.

System metrics that are added traverse many components:

1. EC2/EBS - Collected at the NCs using the `/usr/share/eucalyptus/getstats.pl` script then aggregated at the CC and the polled by the CloudWatch service and added to the eucalyptus_cloudwatch database in the system_metric_data_* tables.
2. AutoScaling - Put into the CloudWatch service by the AutoScaling service when autoscaling groups are created.
3. ELB - Pushed from the Servo VM into the CloudWatch service using boto. Only pushed when values are non-zero.

GetStatistics calls are received on the CloudWatch service which then scans the db, aggregates the data and then returns the requested statistics to the user based on the given time bounds and period.

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
* ELB metrics do not get entered unless requests are going through the load balancer as per AWS semantics.
* EC2 Disk metrics are only valid for ephemeral disks. For BFEBS instances root filesystem usage is tracked as and EBS metric.

[[category.Training]]