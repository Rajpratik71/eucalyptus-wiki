
### Overview
Bucket Lifecycle is implemented in three main pieces:


1. Database schema to support lifecycle configuration
1. Scheduled jobs to implement lifecycle functionality
1. Traditional bindings and processing for parsing and storing lifecycle-related requests


### Database Schema
First we will look at the database schema for lifecycle support.

lifecycle_rules_erd

The fields:


* enabled
* expiration_date
* expiration_days
* prefix
* rule_id
* transition_date
* transition_days
* transition_storage_class

each are directly related to a field for a rule within a bucket's set of lifecycle rules. For instance, the following lifecycle document:


```xml
<LifecycleConfiguration>
  <Rule>
    <ID>id1</ID>
    <Prefix>documents/</Prefix>
    <Status>Enabled</Status>
    <Transition>
      <Days>30</Days>
      <StorageClass>GLACIER</StorageClass>
    </Transition>
  </Rule>
  <Rule>
    <ID>id2</ID>
    <Prefix>logs/</Prefix>
    <Status>Enabled</Status>
    <Expiration>
      <Days>365</Days>
    </Expiration>
  </Rule>
</LifecycleConfiguration>
```
will result in the database being populated similar to the following:



| id | bucket_uuid | enabled | expiration_date | expiration_days | last_processing_start | prefix | rule_id | transition_date | transition_days | transition_storage_class | creation_timestamp | last_update_timestamp | metadata_perm_uuid | version | 
|  --- |  --- |  --- |  --- |  --- |  --- |  --- |  --- |  --- |  --- |  --- |  --- |  --- |  --- |  --- | 
| ... | 
| aaa | aaa-bbb-ccc | 1 |  |  | ... | documents/ | id1 |  | 30 | GLACIER | ... | ... | ... | ... | 
| aab | aaa-bbb-ccc | 1 |  | 365 | ... | logs/ | id2 |  |  |  | ... | ... | ... | ... | 



Fields left blank in the above table will have NULL as their value. Fields with ... will be populated, but the data is not important with regards to the design of the bucket lifecycle components.

Please notice that the bucket_uuid field in the lifecycle_rules table points to the bucket's uuid which is an internal identifier.

To see a listing of the rules for a given bucket, you can use the following SQL query:


```sql
SELECT b.bucket_name, a.rule_id, a.prefix, a.enabled, a.expiration_date, a.expiration_days, a.transition_date, a.transition_days, a.transition_storage_class
FROM lifecycle_rules a
INNER JOIN buckets b ON a.bucket_uuid = b.bucket_uuid
WHERE b.bucket_name = ?
```

### Scheduled Jobs
Once the lifecycle configuration is stored in the database, a background process will run within the Object Storage component that will execute the lifecycle's rules. The scheduling and execution of the job is governed by a library called [Quartz](http://quartz-scheduler.org/). When the Object Storage component is started, the quartz scheduler is also started. Within the eucalyptus_osg database, there is a table called scheduled_jobs. The Object Storage component's bootstrap system will scan this table and find a list of jobs to be scheduled. The jobs can be scheduled to run on an interval such as every 60 seconds. Jobs can also be scheduled to run on a [cron](http://www.quartz-scheduler.org/documentation/quartz-2.x/examples/Example3) schedule. The job for implementing lifecycle functionality is called the LifecycleReaperJob. The LifecycleReaperJob's functionality is implemented within the com.eucalyptus.objectstorage.jobs.LifecycleReaperJob class. By default, this job will be added to the scheduler configuration with the following cron schedule: "0 0 1 \* \* ?". This will tell the scheduler to execute the job at 1AM every night.

To understand the quartz scheduler configuration, it is important to realize that there are two pieces, the configuration in-memory and the configuration stored in the database. The database configuration is read when Eucalyptus starts up and creates the in-memory configuration. Changing the scheduling configuration in-memory is possible, but you will also need to change the database configuration for the changes to persist across restarts of the cloud. To view the in-memory configuration, you can use the following groovy:


```groovy
#!/usr/bin/groovy
import com.eucalyptus.objectstorage.bootstrap.ObjectStorageSchedulerManager;
import org.apache.log4j.Logger;
def runIt() {
    Logger LOG = Logger.getLogger(ObjectStorageSchedulerManager.class);
    LOG.info(ObjectStorageSchedulerManager.dumpInfo());
    return "script ";
}
return runIt();
```
you can run a groovy script by using: euca-modify-property -f euca=name_of_script.groovy

This script will dump the configuration into the log file, most likely $EUCALYPTUS/var/log/cloud-output.log. If you want to look at the configuration in the database, you can execute "select \* from scheduled_jobs" in the eucalyptus_osg database. Manipulating the in-memory configuration and database configuration is possible, but there are currently no tools for it.


### Bindings and Processing
There is nothing special about the binding and processing of lifecycle documents. The binding is performed in the WalrusRESTBindings class and the processing is done in the ObjectStorageGateway class much like nearly all other bindings and processing. When the lifecycle component was implemented, Amazon S3 did not allow bucket lifecycle and versioning to coexist. However, shortly after development on Eucalyptus 4.0.0 completed, Amazon S3 began allowing buckets to support versioning and lifecycle simultaneously.

Lifecycles cannot be modified. A lifecycle configuration is created for a bucket when a configuration is PUT against the ?lifecycle subresource of a bucket. If you wish to modify a lifecycle, you must PUT a new lifecycle configuration. You can also delete a lifecycle configuration by issuing a DELETE request against the ?lifecycle subresource.





*****

[[category.storage]] 
[[category.confluence]] 
[[category.objectstorage]]
