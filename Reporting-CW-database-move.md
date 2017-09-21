 _The purpose of this documentation is to illustrate the changes for moving reporting/CW databases into Euca-controlled instances._ 

 **Background** To overcome the scalability challenges in HA due to a large number of DB connections, we started to use schemas as persistence contexts in 4.1 ([EUCA-9262 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-9262)[EUCA-9617 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-9617)). The exceptions to this change are reporting and cloud watch databases, as their size causes large delays in database sync. The proposed architectural change is to move the databases into Euca-controlled instances, so that we can take them out of the existing DB sync (HA) and use the other mechanisms (volume-snapshot) for backup/restore. We anticipate that this architectural change increases the scalability of Eucalyptus front-end.

 **Tasks** 
*  **Instance launch workflow** 

    To setup various resources related to DB instances, such as security group, server certificates, and autoscaling groups, we need a workflow that construct the resources in proper order. This is a similar to what we have in ELB and Imaging Service.

    

    
*  **(Optional)Schemas for append only data** 

    It is proposed that we split the reporting and CW databases into the schemas in a way that one schema contains updatable data while the other schema contains the append-only data. For both the reporting and CW, the large portion of data would be append-only. Based on this scheme, the updatable schemas would be in the same database (co-hosted with Eucalyptus service), while the append-only schemas would be externally hosted on the VM.

    

    
*  **Database initialization** 

    When Eucalyptus service is initialized, it creates and initializes databases (in fact it does that every time it fails to find the database). The initialization of the reporting and CW databases should be no longer performed during the bootstrap (or creates updatable schema if we implement the split schemas).

    

    
*  **Database connection pool** 

    We need to change when and to which endpoint the connection pool is created. Currently, the connection pools for each database are created during the bootstrap, while the proposed change requires the connection pool constructed dynamically at run-time. In other words, we need to coordinate the service lifecycle and the management of connection pool such that the connections are made after the remote DBs are ready to accept the connection. The reporting and CW services should be enabled only after the connection pools are created for each remote database.

    

    
*  **Service life-cycle changes** 

    We need to create a service-specific bootstrapper (similar to LoadBalancingPropertyBootstrapper) that monitors the certain preconditions that should be met (such as setting the EMI property for the reporting/cw db) and triggers actions related to service transition. The service transition would trigger instance-launch workflow. The service should be enabled only after the connection pools are created for the remote DB.

    

    
*  **Initializing database in the instances** 

    In the instance, we need the scripts that starts a postgre process and initializes the database if it is not found. This is what <setup_db.groovy> is doing currently.

    

    
*  **Properties** 

    To help cloud-admins to manage the parameters of the service, we need the properties and the corresponding handlers. This includes but not limited to changing instance types, keyname, emis, and availability zones of the db instances.

    

    
*  **Packaging/Release** 

    The similar mechanism to installing ELB/imaging images should be implemented. Also the image should have the necessary packages installed (e.g., postgre) and the service scripts built.

 **Order of operations** This is one possible order of tasks:


*  **(Optional)Schemas for append only data** 
*  **Database connection pool** 
*  **Initializing database in the instances** 
*  **Database initialization** 

After the above tasks are completed, we should be able to manually construct the DB instance and have the reporting and cloudwatch services to use the external DB.


*  **Instance launch workflow** 
*  **Service life-cycle changes** 
*  **Properties** 

After these tasks, cloud admins should be able to change the properties so that the reporting and cloudwatch services are enabled with external DB.


*  **Packaging/Release** 

 **Challenges / Open Questions** 
*  **DB credential** 

    Currently when connection pool is created locally, we use the database password that is protected with the file system permission. To connect to the remote database, we need to deliver the DB password to the VM in secure way (and the scripts should use the password when initializing the DB). We may use the same mechanism that we used for imaging and ELB (server certificates).
*  **Security** 
*  **Performance issues** 





*****

[[category.confluence]] 
