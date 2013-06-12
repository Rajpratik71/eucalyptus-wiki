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

Alarms can be created that monitor the CloudWatch data for particular conditions. When an alarm state is triggered it can be used to kick an autoscaling policy that defines a scaling activity for a particular ASG.  The alarm states are evaluated with the following algorithm:

1. If you ask to evaluate for n evaluation periods, we will calculate n + 2
(or possibly up to n + 5, depending on how small the windows are)
2. If a window has no data, it will look in the previous evaluation period (and further back) and copy that data for the sake of evaluation
3. OK = one of the evaluation period has ok data (previous n)
4. ALARM = (previous n windows have ALARM)
5. INSUFFICIENT_DATA = anything else

### Component level responsibilities
* The CloudWatch service is currently colocated with the ENABLED CLC. 

* Custom metrics put into the system are directly entered into the eucalyptus_cloudwatch database in the  custom_metric_data_* tables. No other components are involved at that point.

* System metrics that are added traverse many components:

1. EC2/EBS - Collected at the NCs using the `/usr/share/eucalyptus/getstats.pl` script then aggregated at the CC and the polled by the CloudWatch service and added to the eucalyptus_cloudwatch database in the system_metric_data_* tables.
2. AutoScaling - Put into the CloudWatch service by the AutoScaling service when autoscaling groups are created.
3. ELB - Pushed from the Servo VM into the CloudWatch service using boto. Only pushed when values are non-zero.

* GetStatistics calls are received on the CloudWatch service which then scans the db, aggregates the data and then returns the requested statistics to the user based on the given time bounds and period.

* The only background processes related to CloudWatch are the alarm evaluators which look at the historical data for a metric and decide whether or not to throw an alarm. The alarms are evaluated every minute. 

### User level operation and tooling
* How do end users interact with the feature?
Euca2ools provides a suite of tools for interacting with the CloudWatch service that are under the euwatch-* prefix.
```
[root@clcfrontend ~]# euwatch-
euwatch-delete-alarms               euwatch-describe-alarms-for-metric  euwatch-get-stats                   euwatch-put-metric-alarm
euwatch-describe-alarm-history      euwatch-disable-alarm-actions       euwatch-list-metrics                euwatch-set-alarm-state
euwatch-describe-alarms             euwatch-enable-alarm-actions        euwatch-put-data
[root@clcfrontend ~]# euwatch-
```
It may be valuable to graph CloudWatch metrics in order to determine their validity and accuracy. For this task, EucaLobo is the preferred mechanism. A [blog post](http://testingclouds.wordpress.com) has been written on how to install, configure, and use EucaLobo. 

## Administrative Tasks
* There are two main knobs to turn wrt to CloudWatch:
```
PROPERTY        cloud.monitor.default_poll_interval_mins        5
DESCRIPTION     cloud.monitor.default_poll_interval_mins        How often the reporting system requests information from the cluster controller
PROPERTY        cloud.monitor.history_size      5
DESCRIPTION     cloud.monitor.history_size      The initial history size of metrics to be send from the cc to the clc
```
* By setting the `cloud.monitor.default_poll_interval_mins` property you are setting how often the user will see updates to cloudwatch data. The `cloud.monitor.history_size` property states how many intervals will be reported within the window defined by `cloud.monitor.default_poll_interval_mins`. So for example the defaults of 5/5 will ensure that every 5 minutes CloudWatch data is updated in the db and at that point 5 datapoints will be entered. The datapoints are polled at an even interval across the `cloud.monitor.default_poll_interval_mins`. That means that setting the values to 5/10 will mean that a datapoint will be collected from backend every 30 seconds. Setting the value to 1/2 also provides the same periodicity but ensures that users are able to see their data more rapidly. 

* CloudWatch data is purged when it is 2 weeks old. If the admin would like to purge data manually and the procedure is listed [[here|CloudWatch-FAQ]]

## Debugging through log messages
* When at TRACE level each datapoint entered into the system is logged. This can cause slowness if CloudWatch is being heavily used.

## Gotchas
### ELB 
* Metrics do not get entered unless requests are going through the load balancer as per AWS semantics.

### EC2/EBS
* EC2 Disk metrics are only valid for ephemeral disks. For BFEBS instances root filesystem usage is tracked as and EBS metric.

### ELB
* Metrics for requests show statistics in slightly strange ways:
1. Sum and SampleCount are the total requests in the period
2. Average/Min/Max are always 1 or 0


[[category.Training]]