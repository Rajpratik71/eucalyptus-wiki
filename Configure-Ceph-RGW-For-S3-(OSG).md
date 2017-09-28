This wiki explains how to configure Ceph Rados Gateway (RGW) as the backend for Object Storage Gateway (OSG). OSG provides S3 compliant API in a Eucalyptus cloud. This task assumes that -


* Ceph storage cluster is available
* ceph-radosgw service has been installed (on the UFS or any other host) and configured to use the Ceph storage cluster. For instructions, refer to[[Install and configure Ceph RGW|Install-and-configure-Ceph-RGW]]


## Step-by-step guide
Follow all the steps (from the official documentation) to register UFS. After registration is complete:


1. Configureobjectstorage.providerclienttoceph-rgw


```
# euctl objectstorage.providerclient=ceph-rgw
```

1. Configureobjectstorage.s3provider.s3endpointto ip:port of the host running ceph-radosgw service. Depending on the front end web server used by ceph-radosgw service, the default port is 80 for apache and 7480 for civetweb. In case of Calyptos based installs, ceph-radosgw is colocated with UFS and configured with civetweb to run on port 7480


```
# euctl objectstorage.s3provider.s3endpoint=<radosgw-host-ip>:<radosgw-webserver-port>
```

1. Configureobjectstorage.s3provider.s3accesskeyand objectstorage.s3provider.s3secretkeyto radosgw user credentials


```
# euctl objectstorage.s3provider.s3accesskey=<radosgw-user-accesskey>
# euctl objectstorage.s3provider.s3secretkey=<radosgw-user-secretkey>
```

*****

[[category.storage]] 
[[category.confluence]] 
[[category.ceph]] 
[[category.objectstorage]]
