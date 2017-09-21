Eucalyptus 4.4 will feature tech preview of snapshot deltas for Ceph-RBD EBS backend (only). Use this guide to configure Eucalyptus Storage Controller (SC) to employ incremental upload of EBS snapshots. Ensure that all prerequisites are satisfied first.


## Prerequisites

* Ceph cluster is up and running
* Ceph user credentials assigned to Eucalyptus SC are[configured](https://eucalyptus.atlassian.net/wiki/display/STOR/Ceph+User+Permissions+For+EBS)with appropriate permissions


* ceph-common package is installed on the Eucalyptus SC host (provides rbd cli utility required for import/export of snapshot deltas)
* SC is registered and [[configured|Configure-Ceph-RBD-For-EBS]]with ceph-rbd as the EBS backend
* Snapshot upload (to OSG) is enabled


```
# euctl one.storage.shouldtransfersnapshots
one.storage.shouldtransfersnapshots = true
```



## Additional Storage Controller Configuration
After the prerequisites are met execute the following steps 


1. To enable delta snapshot uploads configure <az>.storage.maxsnapshotdeltasto a non-zero integer value. A value of 0 indicates that a newly created snapshot will be always be uploaded in its entirety. Since thats the expected behavior prior to 4.4 the property will default to 0 out of the box. A non-zero integer value enables upload of incremental snapshots when possible. The configured value indicates the SC to create/upload that many snapshot deltas for a given EBS volume before triggering a full upload of the snapshot contents. The cycle repeats after every full upload implying that between any two consecutive full snapshot uploads for a given EBS volume there will be at most maxsnapshotdeltasnumber of incremental snapshot uploads


```
euctl <az>.storage.maxsnapshotdeltas=<non-zero-integer>
```




*****

[[category.storage]] 
[[category.confluence]] 
