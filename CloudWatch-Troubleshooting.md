## Feature Overview
The CloudWatch service provides users a metrics database that can be used in order to track cloud resources (automatically collected by the system) or arbitrary time series data (added manually by the user). CloudWatch data returned from the server is limited to a minimum resolution of 60 seconds. Datapoints can be entered at arbitrary frequencies but when returned to the user all datapoints within the period will be aggregated. When entering a datapoint the user must provide:
* Namespace
* Dimensions (can be empty)
* Start Time
* End Time
* Value

For each combination of Namespace+Dimensions a set of datapoints can be returned from the service with the following statistics pre-computed for the period:
* Average
* Maximum
* Minimum
* Sum
* SampleCount



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