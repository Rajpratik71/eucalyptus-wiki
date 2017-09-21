
## Overview
The "Eucalyptus Stats" feature of Eucalyptus provides internal system state to consumers outside the Eucalyptus processes to facilitate health checks and operational insight into a Eucalyptus deployment.


## Sensors
For eucalyptus-cloud services (those running in a JVM), the sensors run on a 60-second interval, so new results are output every 60 seconds.

Sensor tree:


* euca
    * components
    * autoscalingbackend
    *  **state– The State and check status of the component** 

    
    * bootstrap
    *  **state** 

    
    * cloudwatchbackend
    *  **state** 

    
    * cluster
    *  **state** 

    
    * component
    *  **state** 

    
    * configuration
    *  **state** 

    
    * db
    *  **state** 

    
    * dns
    *  **state** 

    
    * eucalyptus
    *  **state** 

    
    * imagingbackend
    *  **state** 

    
    * jetty
    *  **state** 

    
    * ldap
    *  **state** 

    
    * loadbalancingbackend
    *  **state** 

    
    * node
    *  **state** 

    
    * notifications
    * pollednotifications
    * properties
    * reporting
    *  **message_contexts– The number of currently active message contexts (messages being processed by this jvm) at the time the sensor runs** 

    
    * db
    *  **connection_pools– The active connections and pool sizes for each db connection pool in the JVM (typically just two: database_events and eucalyptus_shared)** 

    
    * jvm
    * memory
    *  **pools– Usage info about each memory pool used by the JVM (eden space, code cache, perm gen, etc)** 
    *  **general– Basic memory stats on Heap usage and non-heap usage** 
    *  **gc - Garbage collector info like count, sweep time, etc** 

    
    * threads
    *  **state - Basic JVM thread info about num threads, total started, locked threads, peak thread count, etc.** 

    

    

    

For eucalyptus-cc and eucalyptus-nc services, the sensors run on a 60-second interval so new results are output every 60 seconds.


* euca
    * components
    * cc
    * msgs
    *  **stats– Per-message-type counters for message handling latency (in milliseconds) and statistics– min, max, count, avg** 

    
    *  **state– The State and check status of the component** 

    

    

    
* Service state (ENABLED|DISABLED).
* Message processing latency statistics on a per-message-type basis
    * For each message type (DescribeInstances, RunInstance, etc), the number, min, max, and avg latency to process each request over the last sensor interval
    * For each message type the number of sucessful vs failed requests

    


## Configuring Stats
For eucalyptus-cloud services: In the /etc/eucalyptus/eucalyptus.conf file, add  _-Deuca.enable_stats=true_ to the CLOUD_OPTS line. A service restart is required.

For eucalyptus-nc/cc services: In the /etc/eucalyptus/eucalyptus.conf file, add  _ENABLED_SENSORS="messages service_state"_  to the file as a single line, no restart required.









*****

[[category.monitoring]] 
[[category.confluence]] 
