## Feature Overview
Elastic Load Balancing automatically distributes incoming application traffic across multiple instances. It enables you to achieve even greater fault tolerance in your applications, seamlessly providing the amount of load balancing capacity needed in response to incoming application traffic. Elastic Load Balancing detects unhealthy instances within a pool and automatically reroutes traffic to healthy instances until the unhealthy instances have been restored.

#### Component level responsibilities
* Talk about what the involvement of each Eucalyptus component is wrt to the feature. 
* Are there any new components being added? 
* What is the flow of a user request?
* What actions/processes are happening in the background at all times? 

#### User level operation and tooling
* How do end users interact with the feature?
Along with new features comes new Euca2ools.  Euca2ools 3 attempts to mirror all AWS cli functionality for ELB. The elb specific commands have the prefix "eulb-" pronounced "you'll be". New CLI guide will cover all the new eulb commands. 
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




* Show a few typical use cases for the feature end-to-end

## Administrative Tasks
* How will the administrators configure the feature?
* How will admins monitor/create/delete resources from other accounts?

## Debugging through log messages
* Show in one of the use cases from above the log messages that are expected to be seen across the various components and how one can understand where an issue is stemming from.

## Gotchas
* This section should show any caveats or known bugs that will trip up users in the field.

*****
[[category.Training]]