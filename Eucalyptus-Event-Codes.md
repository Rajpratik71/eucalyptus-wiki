
## Document Status:Draft

## Overview
Event codes are meant to categorize the types of failures that may happen in a Eucalyptus Cloud. From this page, Cloud Admins should be able to find more information about the nature of an event they are investigating as well as links to documentation and possibly steps to repair the issue that caused the event to be emitted.


## Event Codes


| Event Code | Code Name | Event Code Description | Event Code Documentation and Possible Fixes | Eucalyptus Version(s) Introduced | 
|  --- |  --- |  --- |  --- |  --- | 
| 10 | SAN Not Responding | Information was sent to the SAN you have configured in your cloud, but the SAN never responded to the request | <ul><li>may be an intermittent failure</li><li>check connectivity from the originHost SC and the configured SAN</li></ul> | 4.2.0 | 
| 11 | SAN Operation Failure | A request was sent to the SAN, but the response from the SAN indicated that the operation failed at the SAN | <ul><li>may need to investigate SAN configuration</li><li>potentially an intermittent failure</li></ul> | 4.2.0 | 
| 1001 | Invalid eucalyptus.conf | Invalid eucalyptus.conf file | <ul><li>check that the eucalyptus.conf file has a valid configuration for the version of Eucalyptus you are using</li></ul> |  | 
| 1003 | Low Disk Space | At some point, a component within Eucalyptus attempted to perform an operation and the operating system halted the operation because there was insufficient disk space. | <ul><li>Delete unnecessary files on partition that hosts Eucalyptus data

</li><li>Allocate more disk space on the partition that hosts Eucalyptus

</li><li>Install more storage and allocate it to the partition that hosts Eucalyptus

</li></ul> |  | 
| 1004 | Java needs more memory | Java is running very low on heap space memory. Java does not automatically allocate large amounts of memory, even when a system has sufficient memory. | <ul><li>Ensure that the CLOUD_OPTS variable set in eucalyptus.conf includes a value for -Xmx. Set or increase this value as necessary: by default, it should be at least 1GB or 1/4 of the physical memory, whichever is smaller. This value can be expressed in bytes, or as '1G'.

</li><li>Restart the cloud application.

</li><li>If this condition continues despite a high value for -Xmx, it may be necessary to add more physical memory to the machine.

</li></ul> |  | 
| 1005 | Java needs more memory | Although this is similar to event 1004 (Heap Space), it is slightly different. Java allocates memory for two distinct purposes. Heap space is allocated for performing operations and storing variable, etc. Java uses "permgen space" for internal purposes. | <ul><li>Ensure that the CLOUD_OPTS variable set in eucalyptus.conf includes a value for -XX:MaxPermSize=<xx>M. Set or increase this value as necessary: Eucalyptus overrides the standard java default of 64MB to 256MB, you'll likely need to set it to 384MB or higher. This value can be expressed in bytes, or as '384M', '1G' etc.

</li><li>Restart the cloud application

</li><li>If this condition continues despite a high value for --XX:MaxPermSize, it may be necessary to add more physical memory to the machine.

</li></ul> |  | 
| 1006 | Exhausting DB Connection Pool | Running very low db connections in db pool ${alias} | <ul><li>Edit the file EUCALYPTUS_HOME/etc/eucalyptus/cloud.d/scripts/setup_dbpool.groovy

</li><li>Modify the default_pool_props 'proxool.maximum-connection-count', increase the max connection value ('${maxConnections}') to a higher value, and save the file.

</li><li>Restart the cloud controller.

</li></ul> |  | 
| 1008 | Daemon is not running | An important system component has not been launched | <ul><li>Install ${daemon}

</li><li>Run ${daemon} For ${daemon} this can be eucalyptus-cc, eucalyptus-nc, eucanetd or other components introduced in the future

</li></ul> |  | 
| 1009 | Keys out of sync | Mismatched cryptographical keys. | 
1. Ensure that the keys (.pem files) in

 EUCALYPTUS_HOME/var/lib/eucalyptus/keys

are the same on ${sender} and ${receiver}.  


1. Restart ${sender} and ${receiver}



 |  | 
| 1010 | ${DB_LOCK_FILE} is present | DISABLED CLC will not start with the ${DB_LOCK_FILE} present on the file system. | <ul><li>Start the previously enabled cloud controller (CLC) or post a question on [https://engage.eucalyptus.com/](https://engage.eucalyptus.com/)</li></ul> |  | 
| 1011 | inconsistent DB | This CLC host was previously involved in a network partition where DB state may have become inconsistent. | The system has undergone a network partition that lead to a network split between the cloud controllers (CLCs).The CLCs may have become inconsistent and to avoid data loss the system has fail-stopped.If you are able to determine which of the CLC hosts has your canonical data (HOST_1) and which one doesn't (HOST_2), then:
1. Backup the /var/lib/eucalyptus/db/ directory on both HOST_1 and HOST_2.
1. Delete ${DB_LOCK_FILE} on HOST_1.
1. Start /etc/init.d/eucalyptus-cloud on HOST_1.
1. Verify that the service is working as expected on HOST_1.
1. Really. Convince yourself you haven't chosen the wrong database state.
1. Delete ${DB_LOCK_FILE} on HOST_2.
1. Start /etc/init.d/eucalyptus-cloud on HOST_2.
1. Verify that the system comes back up and behaves as you expected.  Please report any problems or post a question on [https://engage.eucalyptus.com/](https://engage.eucalyptus.com/)  

 |  | 
| 1013 | No Virtualization on host | no virtualization support on host operating system (or hardware) | 
1. Check your CPU supports vendor virtualization extensions.


1. Enable CPU virtualization extensions in the systems BIOS.


1. Ensure KVM modules can be loaded.



 |  | 
| 1014 | Loadbalancer image cannot launch | Load balancer image not configured. LoadBalancing service will not be available. | 
1. Install the load balancer image package:

yum install eucalyptus-load-balancer-image  


1. Install and register the load balancer image:

euca-install-load-balancer --install-default



 |  | 
| 1015 | Imaging worker image cannot launch | Imaging worker image not configured. Imaging service will not be available. | 
1. Install the imaging worker image package:

yum install eucalyptus-imaging-worker-image  


1. Install and register the imaging worker image:

euca-install-imaging-worker --install-default



 |  | 
| 1500 | Disabled Cloudwatch | CloudWatch service has been disabled. This means alarms will not be evaluated, and new data and alarms will not be added. Existing data and alarms can be queried. | <ul><li>Run euca-modify-property -p cloudwatch.disable_cloudwatch_service=false</li></ul> |  | 
| 1501 | Disabled Reporting | Reporting service data collection has been disabled. This means no new reporting data will be populated. Existing data can still be queried. | <ul><li>Run euca-modify-property -p reporting.data_collection_enabled=true</li></ul> |  | 
| 2000 | tgtadmin unresponsive | Storage operation timed out after waiting for ${configured timeout} milliseconds for a response from tgtadm command | 
1. Run 'ps -ef | grep tgtd', look for the process-ID of tgtd.


1. Kill the tgtd process with 'kill process-ID'.


1. Start the tgt daemon with 'service tgtd start'.


1. Run 'tgtadm --op show --mode target', check that it does not hang or return an error.



 |  | 
| 2001 | iscsiadm command failing | Volume operation (attach/detach/snapshot) failed to complete | 
1. Ensure all iSCSI kernel modules are loaded (e.g. 'lsmod | grep iscsi' should output at least 'iscsi_tcp')


1. Restart iscsid service (e.g. service iscsid restart).


1. Run 'iscsiadm -m session -P 1' should say: 'iscsiadm: no active sessions' or display session information.



 |  | 
| 2002 | tgt errors | tgt service (Linux SCSI target daemon) is responding with errors | 
1. Run 'service tgtd status', check the service status


1. If the tgt service is not started or if the status contains errors, restart the service with 'service tgtd restart'


1. Run 'tgtadm --op show --mode target', check that it does not hang or return an error.



 |  | 
| 2003 | Storage login failure | Volume operation (attach/detach/snapshot) failed to complete. Unable to login to storage target | <ul><li>If the initiator timed out during the login process, check the network path to the storage target causing the error.</li></ul> |  | 





*****

[[category.monitoring]] 
[[category.confluence]] 
