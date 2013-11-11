Eucalyptus 3.4.0 Tech Preview includes an implementation of an object storage system that is designed to scale out horizontally, with multiple active front end nodes implementing the S3 API.

In 3.4.0 Tech Preview mode, Eucalyptus uses [RiakCS](http://basho.com/riak-cloud-storage/) as the distributed object storage backend for Walrus.

The legacy, single node Walrus implementation is superceded by multiple active Object Storage Gateways (OSGs), which act as pass through proxies for user requests. The cloud administrator can register multiple, active OSGs, which are designed to act as redundant active/active nodes. The client may direct requests to any active OSG. OSGs handle authentication and enforce IAM (Identity and Access Management) settings.

On the backend, each OSG connects to a distributed object store (currently RiakCS). A typical RiakCS installation consists of a number of storage nodes (at least 5 for production use) that run Riak/RiakCS components. We expect that Riak/RiakCS is installed and functioning before attempting OSG configuration.

We do NOT recommend accessing the RiakCS installation directly. Doing so might lead to an inconsistent state and you may not be able to access your data through Eucalyptus.

### Registering Object Storage Gateways (OSGs)

The cloud administrator can register multiple Object Storage Gateway (OSG) components with the Eucalyptus Cloud Controller (CLC). 

To do so, you may use the euca_conf utility or the euca-regiser-object-storage-gateway command. For instance,

    euca_conf --register-osg <OSG IP>

You should do this for every OSG you wish to register (substituting the IP with the correct one, of course).

OSGs may be deregistered similarly, but remember to use the OSG's component name and NOT its IP address. 

    euca_conf --deregister-osg <OSG component name>

You can list the component name for your OSG by running euca-describe-services. For example,

    SERVICE	objectstorage  	objectstorage  	osg-192.168.1.16	NOTREADY  	23  	http://192.168.1.16:8773/services/objectstorage	arn:euca:bootstrap:objectstorage:objectstorage:osg-192.168.1.16/

To deregister this OSG, you would run the following command,

    euca_conf --deregister-osg osg-192.168.1.16

### Configuring the Storage Provider

After you register an OSG, you cannot use it until it is correctly configured. After registration, OSG state will be initially listed as BROKEN. For example,

    SERVICE	objectstorage  	objectstorage  	osg-192.168.1.16	BROKEN    	23  	http://192.168.1.16:8773/services/objectstorage	arn:euca:bootstrap:objectstorage:objectstorage:osg-192.168.1.16/

To configure the OSG, please specify a storage provider using the euca-modify-property command. To use RiakCS as the backend, you will need to pick "s3" as the storage provider,

    euca-modify-property objectstorage.providerclient=s3

You will now have to specify the RiakCS/S3 endpoint that you wish to use with Eucalyptus (you may configure a round robin DNS and specify a host name instead of an individual node IP). For example,

    euca-modify-property objectstorage.s3provider.s3endpoint=riakcs-01.riakcs-cluster.myorg.com

In addition, you will have to provide Eucalyptus with credentials to access your RiakCS installation.

    euca-modify-property objectstorage.s3provider.s3accesskey=<access key>

    euca-modify-property objectstorage.s3provider.s3secretkey=<secret key>

Please note that these are access and secret keys for the RiakCS cluster and NOT Eucalyptus. You may use the RiakCS front end web interface to create users.

Make sure that the user with these credentials has administrative access to RiakCS.

### Checking Service State

You may use euca-describe-services to check service status. After successful configuration, the state of the OSG will be reported as ENABLED.

    SERVICE	objectstorage  	objectstorage  	osg-192.168.1.16	ENABLED    	23  	http://192.168.1.16:8773/services/objectstorage	arn:euca:bootstrap:objectstorage:objectstorage:osg-192.168.1.16/

If the state appears as DISABLED or BROKEN, please check cloud-*.log files in /var/log/eucalyptus. "DISABLED" generally indicates that there is a problem with your network or credentials.

### Accessing Object Storage

You can now use your favorite S3 client (e.g. s3curl) to interact with Eucalyptus. Simply replace your S3_URL with the address of the OSG you wish to interact with and the service path with "/services/objectstorage" instead of "/services/Walrus". For example,

    S3_URL = http://<OSG IP>:8773/services/objectstorage

Or you may set your s3 endpoint manually.

If you have DNS enabled, you may use the "objectstorage" prefix to access object storage. Eucalyptus will return a list of IPs that correspond to ENABLED OSGs.

### Configuring Load Balancers

We recommend that you use a load balancer to balance traffic across OSG nodes, as well as RiakCS nodes. Below is an example of how to use [Nginx](http://wiki.nginx.org/Main) to get you started. You may use [HAProxy](http://haproxy.1wt.eu/) if you wish.

You will have to install Nginx on one of your servers and tell direct HTTP traffic to your RiakCS nodes. By default, RiakCS listens to web traffic on port 8080. In this example, riakcs-00.yourdomain.com, riakcs-01.yourdomain.com and riakcs-02.yourdomain.com are three RiakCS nodes that you have previously configured.

On many Linux installations, Nginx uses /etc/nginx/conf.d for server configuration. You can either edit the default configuration or create a new config file. Here is a sample configuration,

 upstream riak_cs_host {
  server riakcs-00.yourdomain.com:8080;
  server riakcs-01.yourdomain.com:8080;
  server riakcs-02.yourdomain.com:8080;
 }

 server {
  listen   80;
  server_name  _;
  access_log  /var/log/nginx/riak_cs.access.log;

  location / {
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_redirect off;

      proxy_connect_timeout      90;
      proxy_send_timeout         90;
      proxy_read_timeout         90;
      proxy_buffer_size    128k;
      proxy_buffers     4 256k;
      proxy_busy_buffers_size 256k;
      proxy_temp_file_write_size 256k;

      proxy_pass http://riak_cs_host;
    }
}

