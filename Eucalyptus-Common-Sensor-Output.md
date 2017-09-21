
## Overview
Sensors run **within**  eucalyptus processes all output data to a common filesystem tree such that to get the sensor output, all one must do is read a file. This enables easy integration with existing systems as well as manual debugging by administrators.

The common location for all sensor output from internal Eucalyptus sensors is:

 _$EUCALYPTUS/var/run/eucalyptus/status_ 



Within the directory, each sensor will output its readings to a specific file with a specific path indicated by the sensor name itself.

Sensor Data File FormatEach sensor data file is a json-formatted document containing a common set of elements as well as a per-sensor custom set of data elements.

For Example:


```
{ "sensor":"euca.components.testservice.msgs.stats",
  "timestamp":1415269845,
  "ttl":60,
  "tags":[
    "period: 60"
  ],
  "values":{
    "enabled":true,
    "fakemessage":{
      "failure_count":0,
      "mean":500.000000,
      "min":500.000000,
      "max":500.000000,
      "count":1,
      "success_count":1
    },
    "fakemessageDescribe":{
      "failure_count":0,
      "mean":250.000000,
      "min":250.000000,
      "max":250.000000,
      "count":1,
      "success_count":1
    },
    "fakemessageRun":{
      "failure_count":0,
      "mean":200.000000,
      "min":200.000000,
      "max":200.000000,
      "count":1,
      "success_count":1
    }
  }
}


```
The common elements to all sensor output files are:


*  _sensor -_ The name of the sensor that also indicates the path to this file (e.g. euca.components.cc.state)
*  _timestamp_ -The timestamp, in epoch seconds, when the sensor was run and the data collected
*  _ttl -_ The time-to-live in seconds of the data values. The data in this sensor output is not considered any time after _timestamp+ttl_ epoch seconds (typically 60 seconds after the sensor is run)
*  _values_ map - The sensor output values/data collected by this sensor
* and _tags_ array - Tags to help classify or filter the sensor data in external systems

The members of the _tags_  array and _values_  set are specific to each sensor.


## Sensor Data Security
Each sensor data file is owned by _eucalyptus_ user and the _eucalyptus-status_ group with "rw-r—" or 640 permissions so that only the _eucalyptus_  user can modify and only members of the group _eucalyptus-status_  group can read the data. Thus, to allow another system, e.g. nagios, to consume the data you must add the user that runs the agent on each host to the _eucalyptus-status_  group.







*****

[[category.monitoring]] 
[[category.confluence]] 
