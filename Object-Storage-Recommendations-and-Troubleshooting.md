
# General Recommendations
Depending on the configuration of the deployment, some transient errors are to be expected. Systems like Riak CS and most other distributed object stores will periodically experience errors and/or return errors if load becomes too great. In these cases the OSG will also return an error to the user (almost always a '500 Internal Error' or '503 Service Unavailable'). The expected response is that the client will back-off and retry the request. This is the same expected client behavior for S3:[S3 Best Practices](http://aws.amazon.com/articles/1904) (note, however, that their recommendations for GET/PUT with respect to key orderings and bucket name prefixes are not necessarily applicable for Eucalyptus).


## Walrus Recommendations (these are not changed in 4.0.0)
If using Walrus, it is recommended that Walrus be hosted on a non-OSG and non-CLC host.

If using Walrus in HA-mode utilizing two Walrus hosts connected via DRBD, it is important for performance and health of the system when under load that those not be on the CLC host so as to not starve the CLC/DB of IO resources due to heavy user load on the object storage system. With DRBD replication, for every 1Gbps of data traffic into a Walrus host for data storage, it must also send 1Gbps of data to the DRBD replica, so there is up-to a 2X increase in network usage. This can be mitigated if you configure a dedicated network and interface on each Walrus host for DRBD traffic. The best practice is to have such a dedicated network link for DRBD traffic.


## Riak CS Recommendations

1. A load balancer between the OSG(s) and Riak CS cluster is highly recommended. [HAProxy](http://haproxy.1wt.eu/) is currently recommended over [nginx](http://nginx.org/) due to it's better HTTP 1.1 support, though nginx can be used if the latest version is installed (the default in the package repos for RHEL and CentOS is too old to support HTTP 1.1, which is necessary for some operations).
1. The minimum Riak CS cluster size for production, as recommended by Basho, is 5 hosts. These should be distinct hosts from those used by other Eucalyptus components.
1. Eucalyptus does not require exclusive access to the Riak CS cluster, but the credentials used by Eucalyptus should not be used by other users of Riak CS




# Object Storage Common Issues

1.  **_OSGs stay in 'BROKEN' state as shown byeuca-describe-services._**  This is typically a result of no configured OSP. After registering the first OSG you must configure the backend that Eucalyptus will use for object storage. See[[Object Storage Configuration|Object-Storage-Configuration]] for more details.
1.  **Client is getting lots of '403 Forbidden' errors when trying to do S3 operations like listing buckets.** This is usually a misconfiguration of the client and how it signs the requests to the OSG. The way the client is configured depends partly on the configuration of the Eucalyptus cloud, specifically with regard to DNS support. Many clients will only support 'dns-style' requests, meaning they will address a bucket by including it as a subdomain in the HOST header of the HTTP request. E.g. HOST mybucket.mycloud.com. Eucalyptus must have DNS configured in order to handle such requests and route them to the proper service. See[Eucalyptus DNS Configuration Documentation](https://www.eucalyptus.com/docs/eucalyptus/3.4/index.html#shared/setting_up_dns.html) for more information. See [S3 Working with Buckets](http://docs.aws.amazon.com/AmazonS3/latest/dev/VirtualHosting.html) for more information on dns-style vs path-style requests and S3 virtual-hosting features.Below is the proper configuration for most clients in both dns and non-dns cases:
    1.  **Eucalyptus DNS enabled, client uses 'dns-style' S3 requests:** 
    1. Set your client's service URL to _objectstorage.<your_configured_euca_domain>:8773_ . The port number is necessary unless it can be configured independently in your client or you have set up a redirection mechanism to redirect port 80 -> 8773 for your cloud.
    1. You should see requests that look like: 'GET /mykey' with host header 'Host: bucket.objectstorage.<your_configured_euca_domain>:8773

    
    1.  **Eucalyptus DNS enabled, client uses 'path-style' request:** 
    1. Set your client's service URL as above.
    1. You will see requests for that look like:'GET bucket/mykey' with host header 'Host: objectstorage.<your_configured_euca_domain>:8773
    1. NOTE: Including the osg service path (/services/objectstorage) will not work if also using the dns-name for the service. In that case the server will interpret '/services' as the bucket name and '/objectstorage/bucket/key' as the key name.

    
    1.  **No Eucalyptus DNS enabled, client uses 'dns-style' S3 requests:** 
    1. This is not supported and will not work with the OSG

    
    1.  **No Eucalyptus DNS enabled, client uses 'path-style' S3 requests:** 
    1. Set your client's service URL to _<your_osg_host_or_IP>:8773/services/objectstorage_ . The port number is necessary unless it can be configured independently in your client or you have set up a redirection mechanism to redirect port 80 -> 8773 for your cloud. The service path is necessary if not using the subdomain 'objectstorage' in conjunction with using Eucalyptus DNS.
    1. You should see requests that look like: 'GET /services/objectstorage/bucket/key' with host header 'Host: <your_osg_host_or_IP>:8773'.
    1. Some clients will sign the service path and others will not. The OSG accepts both types of requests. It will assume that the client has not signed the service-path and if signature verification fails, it will re-insert the service path and try the signature again.

    

    
1.  **Object copies fail or take a long time to complete with RiakCS OSP/backend.** RiakCS does not natively support object copy operations. If the operation fails, you can configure the OSGs to attempt to stream the object from RiakCS through the OSG and back into RiakCS. This is configured by setting _euca-modify-property -p objectstorage.dogetputoncopyfail=true_ .However, because this is now a streaming write operation, the client may see the operation take a long time depending on the size of the object. Some clients will simply terminate the connection. If you have configured this behavior, you may need to adjust client timeouts to handle large object copy operations. We are working on a better long-term solution to this. Also note that Walrus has native support for internal object copies so the OSG is not on the data path in that configuration and will not exhibit the same behavior.
1.  **The OSG gets out-of-heap memory errors during high load.**  This may be caused by a substantial difference in the network performance into the OSG versus out of the OSG and into the backend. In this case, the buffers in the OSG will consume more memory and under high load may, in aggregate, cause memory pressure. If the backend/OSP is unresponsive or too slow the requests will error out, but until that point some buffering is done to keep the client alive. By default, the OSG will buffer up to 100 chunks of data, each up to 100KB in size, per upload. You may reduce this value to reduce memory usage at the cost of returning errors to the user. To configure the value run: _euca-modify-property -p objectstorage.queue_size=<an integer, usually <= 100>._ 
1.  **_Client seeing a lot of 500 Internal Error responses:_** This can be many factors, some of the most common are covered below
    1. RiakCS/S3 endpoint and/or credentials are invalid. Verify that the endpoint and credentials are valid and work. The best way to do this is to use an S3 client like s3curl or s3cmd and list the buckets at that endpoint using the configured values. If that fails, the OSG will not work and you will see internal errors any time you GET, PUT, or HEAD objects or buckets.
    1. Network issues between the OSG and OSP (RiakCS or Walrus). Ensure that the backend is functioning and reachable. High load on the system or the network may also induce this behavior. If you see lots of errors in the logs like ' _Timed out writing data to stream'_  or get error responses with  _'Aborted Upload, could not process data in time. Either increase upload queue size or retry the upload_  later', then the issue is that the OSG is receiving data faster than it can write it to the backend and the internal buffers have filled. This may require either larger buffers, or smaller ones to fail the operation early and allow the client to back-off and retry. See the solution to #4 above for more information.

    
1.  **Under heady load to OSG, when OSG is running on the CLC/DB host, Eucalyptus services become unavailable.** If an OSG or Walrus is co-located with the CLC/DB, heavy load to that component can starve the other Eucalyptus services on that host of IO and make it unable to respond to the multicast messages that track service state. If this happens, Eucalyptus services on other hosts will no longer see the CLC/DB as 'ENABLED' and will halt waiting for a "system view with DB". In extreme cases, if running the CLC/DB in HA-mode it is possible to induce an artificial network partition and have the CLCs go into recovery mode because they can no longer communicate. The nature and threshold of this will greatly depend on your hardware and network, but such a condition is possible under extremely heavy load where storage and non-storage services are co-located. The general rule is that if you want to do HA you need a separate host for each component of Eucalyptus, OSGs and Walrus included. Additionally, any service that has heavy IO load as a result of user actions (OSG, Walrus, SC) should not be co-located with the CLC/DB to prevent resource starvation.







*****

[[category.storage]] 
[[category.confluence]] 
