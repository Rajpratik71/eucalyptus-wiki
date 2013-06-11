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
`euca-run-instance emi-32512341 --monitor
euca-monitor-instances i-1321c132`

#### Component level responsibilities
* Talk about what the involvement of each Eucalyptus component is wrt to the feature. 
* Are there any new components being added? 
* What is the flow of a user request? 
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
* This section should show any caveats or known bugs that will trip up users in the field.

[[category.Training]]