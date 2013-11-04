Eucalyptus 3.4.0 Tech Preview includes an implementation of an object storage system that is designed to scale out horizontally, with multiple active front end nodes implementing the S3 API.

In 3.4.0, Eucalyptus uses [RiakCS](http://basho.com/riak-cloud-storage/) as the distributed object storage backend for Walrus.

The legacy, single node Walrus implementation is superceded by multiple active Object Storage Gateways (OSGs), which act as pass through proxies for user requests. The cloud administrator can register multiple, active OSGs, which are designed to act as redundant active/active nodes. The client may direct requests to any active OSG. OSGs handle authentication and enforce IAM (Identity and Access Management) settings.

On the backend, each OSG connects to a distributed object store (currently RiakCS). A typical RiakCS installation consists of a number of storage nodes (at least 5 for production use) that run Riak/RiakCS components. We expect that Riak/RiakCS is installed and functioning before attempting OSG configuration.

We do NOT recommend accessing the RiakCS installation directly. Doing so might lead to an inconsistent state and you may not be able to access your data through Eucalyptus.

### Registering Object Storage Gateways (OSGs)

The cloud administration can register multiple Object Storage Gateway (OSG) components with the Eucalyptus Cloud Controller (CLC). 

To do so, you may use the euca_conf utility or the euca-regiser-object-storage-gateway command. For example,

    euca_conf --register-osg <OSG IP>

You should do this for every OSG you wish to register (substituting the IP with the correct one, of course).

### Configuring Storage Provider
### Checking Service State
### Accessing Object Storage
### Testing Basic Operations