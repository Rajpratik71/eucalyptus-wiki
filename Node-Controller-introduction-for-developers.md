Execution flow on the Node Controller
* incoming requests triggered by actions that are invoked in response to Cluster Controller (CC) commands (routed via do\*() functions to implementations either in handlers_default.c or handlers_kvm.c) .
* [node/handlers.c](https://github.com/eucalyptus/eucalyptus/blob/master/node/handlers.c):monitoring_thread() function, which loops indefinitely, alternating work and sleep
* [util/sensor.c](https://github.com/eucalyptus/eucalyptus/blob/master/util/sensor.c):sensor_bottom_half() function, which loops indefinitely, alternating work and sleep
* [util/stats/stats.c](https://github.com/eucalyptus/eucalyptus/blob/master/util/stats/stats.c):run_stats() function, which loops indefinitely, alternating work and sleep

    

    

 **Axis2C** Action that NC performs and data-structures that are passed to/from CC are defined in [eucalyptus_nc.wsdl](https://github.com/eucalyptus/eucalyptus/blob/master/wsdl/eucalyptus_nc.wsdl). The [Axis2C](http://axis.apache.org/axis2/c/core/index.html) is used to convert this wsdl to C structures and stubs. Stubs generations is invoked from make command before C files compilation. This blog [post](http://wso2.com/library/3534/) has a good introduction to the development with Axis2C. If a new message needs to be created the safest bet is to copy-paste existing one and changing its name. While adding a new field into existing message it is advised for consistency to add it into end of all data structures/conversion functions. The [util/adb-helpers.h](https://github.com/eucalyptus/eucalyptus/blob/master/util/adb-helpers.h) defines some conversion helpers from/to ADB to C structures used by NC and CC.

 **Initialization** It occurs when the first request arrives at the NC (and  _not_  when NC is started, which only means the HTTPD process is listening for incoming requests), because any of the do\*() functions in handlers.c begin with


```
if (init())

 return (EUCA_ERROR);
```
A lot of things happen in that node/handlers.c:init() function. The highlights are:


* in the retry-able first half of the function (this code will get re-executed upon each incoming request until initialization succeeds):
    * setting signal handlers for SIGUSR1 and SIGALRM
    * reading in the configuration (will be re-read later if it changes), from /etc/eucalyptus/eucalyptus.conf by default
    * setting up logging (can be changed later dynamically)
    * initializing NC hooks (external executables that will get called by NC at certain points, allowing some customization of the installation, including installation of bug work-arounds)
    * loading of crypto certs
    * verification of external dependencies (NC shells out to a bunch of tools)
    * util/sensor.c:sensor_bottom_half() is started via sensor_init() invocation

    
* in the non-retry-able second half of the function (any failure in this code permanently disables the NC, requiring a restart of the service):
    * initialization of semaphores
    * fault subsystem initialization
    * EBS subsystem initialization
    * verification of hypervisor usability
    * discovery and setting of resource limits
    * max_mem is physical memory in MB as reported by hypervisor, unless changed by configuration
    * max_cores is the number of cores available to VMs as reported by hypervisors, unless changed by configuration

    
    * optional initialization of disk backing store, discovery and setting of disk limit
    * if work and cache directory aren't present, they are created, for hosting instance metadata and blobstores
    * if disk limits for work and cache are specified, those limits are used and blobstores are created or resized accordingly
    * if limits aren't specified, then they are either
    * based on the size of existing blobstores or
    * chosen based on available disk space, with 33% going to work and 66% going to cache, with blobstores created accordingly.

    

    
    * instances running on the hypervisor are evaluated and, if they were started by an earlier execution of the NC, they are "adopted":
    * their metadata is loaded into memory and
    * NC starts reporting instances via doDescribeInstances()
    * NC accounts for their resources via doDescribeResource()

    
    * backing store integrity is verified (at this point disk state from non-running instances will get purged)
    * resource names like IP, IQN are discovered
    * node/handlers.c:monitoring_thread() is started in a detached pthread
    * util/stats/stats.c:run_stats() is started in a detached pthread

    

 **Concurrency model**  of NC is a mix of


* pthreads - used for
    * request handling (however, Apache for NC is configured to only invoke one request at a time)
    * the three continuously running threads: monitoring_thread, sensor_thread, stats_thread, as well as
    * for some long-running operations
    * startup_thread
    * terminating_thread
    * createImage_thread
    * bundling_thread
    * startstop_thread
    * rebooting_thread
    * migrating_thread
    * (if you grep the code for pthread_create there will be a few more, but those are used for stress-tests relying on concurrent threads)

    

    


* processes - in a few isolated cases, we fork off a process to do something that isn't safe to do in a thread (we're working around a buggy dependency here)

In contrast, CC does not use pthreads but rather process-based concurrency (it forks off its monitoring, sensor, and stats "threads") with shared-memory regions across these processes. Therefore, the code that is shared by NC and CC was written to accommodate the two models of concurrency and IPC. For example:


* blobstore "library" (storage/blobstore.c) uses an implementation of file locks (see open_and_lock()) that uses both pthreads read-write locks (pthread_rwlock_trywrlock(&(path_lock->lock))) and Posix file locks (fcntl(fd, F_SETLK, flock_whole_file(&l, l_type))) so that mutual exclusion can be achieved both by processes and pthreads using the resource. (Back when Eucalyptus supported VMware, blobstore library was used on the CC hosts by euca_imager processes that could be running concurrently. As of 4.\* series, blobstore library is only used on the NC and potentially in the imaging worker instances.)
* util/sensor.c:sensor_init() function takes a semaphore and a memory region parameter, which signal the use of process-based concurrency and shared-memory IPC. When those parameters are not set, the initialization function creates a pthread.

DebuggingThe main tool is to NCclient that allows to invoke NC operations locally. As a bare minimum you need to define LD_LIBRARY_PATH with something like that


```
export AXIS2C_HOME=/usr/lib64/axis2c
export LD_LIBRARY_PATH=$AXIS2C_HOME/lib:$AXIS2C_HOME/modules/rampart/
```
After that NCclient can be invoked with ./NCclient from your node ridrectory. [This page](https://github.com/eucalyptus/eucalyptus/wiki/Debugging-Eucalyptus-C-language-components) has a few good tips how to debug Eucalyptus C components including NC.

LoggingNode Controller logs data to $EUCALYPTUS/var/log/eucalyptus/nc.log file. This is rotated logger. Logging is done via [util/log.c ](https://github.com/eucalyptus/eucalyptus/blob/master/util/log.c)

ConfigurationNode Controller reads configuration from $EUCALYPTUS/etc/eucalyptus/eucalyptus.conf. Possible configuration properties are defined in [util/eucalyptus.h](https://github.com/eucalyptus/eucalyptus/blob/master/util/eucalyptus.h). The configuration file is monitored inmonitoring_thread so changes to logging and ESB parameters are picked up on a fly and don't require NC process restart.









*****

[[category.confluence]] 
[[category.developer]]
