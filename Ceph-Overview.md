 _A great resource for Ceph architecture and design is:http://docs.ceph.com/docs/master/architecture/_ 

Overall, Ceph has a very simple architecture based on the fundamental premise that data placement calculations should be made symmetrically by any and all nodes in the system, including clients. Removing special proxy/routing nodes means that clients can reach any available data node in the system and that there is no special view of the system from within the system versus externally to it. This serves to fully decentralize the operations of the cluster and allow very large scale. The downside is that these distributed placement decisions require a consistent view of the entire cluster to calculate and that view must be distributed and maintained on clients as well as the cluster hosts.

There are two basic components of a ceph cluster: Object Storage Device daemons (OSDs) and Monitors (MON). OSDs are the primary storage units that manage and persist data and the Monitors maintain and distribute the cluster state to all OSDs as well as clients. Placement decisions are made using the CRUSH algorithm with input: requested pool, object key, and cluster map. All clients and OSDs use this same tuple to compute location for data writes, reads, and re-replication in failure conditions. As long as the cluster map is consistent on all participants, they will all compute the same locations and be able to find data in the cluster in a fully distributed way. Ceph also has a Metadata Server Daemon (MDS), which is used for the CephFS filesystem to maintain the filesystem metadata, but since Eucalyptus does not currently use CephFS, further discussion of that component will be omitted.


### Components in a Ceph deployment for Euca
 **MON**  - 1-3 "monitor" nodes that manage the clustermap and CephX authentication. These nodes are not on the data path, but must be reachable for clients and cluster servers to be able to get the latest cluster state to make data location calculations. Recommended configuration is that these should be distinct and non-OSD hosting machines to isolate failure and reduce network contention for cluster map updates and data operations. MONs use a consensus algorithm if there are more than one to ensure the cluster map is replicated between them in a strongly consistent way for high availability to the rest of the system. Monitors also send and receive heartbeats to OSDs to check on status and update the cluster map in the event of failures of OSDs.

 **OSD**  - Object Storage Device daemons read and write data to a local disk (not SAN, that is an anti-pattern) and send heartbeats to each other to detect failures. Each OSD is typically responsible for a single local disk, and RAID is discouraged by Ceph due to its specific IO patterns that cause lots of RAID overhead due to small writes. Each OSD stores objects which are composed of a key, data, and metadata k/v pairs. Typically this is implemented as files in XFS using xattrs for the metadata pairs. When data is written it is first written to a write-ahead log and then later during log scrubbing moved to the main data store. It is recommended to make the journal and main data store separate disks to avoid a 2x write penalty, and often this is done with a RAID 1 pair of SSDs on each OSD host fronting many OSD SATA/SAS disks to give high throughput and low latency on the journal path.

![](images/storage/)


### The Cluster Map
The cluster map is comprised of: OSD map, PG map, MON Map, Crush Map, and MDS Map. In our deployments of Ceph for use with Eucalyptus we don't use the Ceph FS feature so we don't use MDS nodes and thus the MDS map is empty.

Here are examples from the QA Ceph cluster we run:

MON Map: State and config of Monitors in the System.

250

PG Map: Map of all the PGs in a pool and their status. PGs are the units that hold objects adn are replicated/handed-off atomically.

250

OSD Map: What OSDs are in the system and their status and location.

250

CRUSH Map: The rules used to determine object placement within a pool. placement_pg(pool_x, obj_y) = evaluate_rules(get_rules(get_pool(pool_x)), obj_y)

250


### Data Striping
Because Ceph clients write data directly to OSDs, most special clients, like librbd and RadosGW (S3/Swift) leverage data striping to ensure that each object stays relatively small but overall large writes can be handled efficiently. Without a striping mechanism, each OSD would have to store large atomic objects, say for example 1GB, and that would lead to lots of fragmentation and unbalance in the PGs. Ceph's striping is described in the Data Striping section of:[http://docs.ceph.com/docs/master/architecture/](http://docs.ceph.com/docs/master/architecture/). So, we'll skip how it works and focus on implications instead. Since ceph is using striping for large objects, including RBD images, a given RBD image is distributed across many objects which may map to many PGs and thus many physical hosts. In the extreme case each object is on a different physical host and each object may only be a few MB in size--you can config this on a per image basis, see:[http://docs.ceph.com/docs/master/man/8/rbd/](http://docs.ceph.com/docs/master/man/8/rbd/). Let's use 10MB objects as an example for a 100GB volume. In that case IO for a single EBS volume in Euca is hitting 10000 different hosts if each object mapped to a distinct host and assuming the cluster is that big. In reality, objects will likely collide on PG mapping (typical numbers of PGs per pool are in the 1-4K range), so far fewer hosts are used, but you see the point: a single client may be hitting many hosts with TCP traffic. This means kernel configs must be able to handle these connection counts.

![](images/storage/)


### Ceph RBD Performance Trends and Observations
The overall effect is high IO throughput for a volume overall, but limited latency capabilities since a given LBA (logical block address) of an EBS volume (think sector on a virtual disk) is handled by a single host and IO to that particular LBA can only go as fast as that host can handle it. This leads to a behavior where as OSDs are added to a Ceph cluster overall IO will increase but then hit a plateau and level off. If so much IO is put in the system that the journals start to fill because they cannot flush to disk fast enough, then the journal writes become blocked by the flush of enough data to the backing disk and now the cluster performs at SAS/SATA HDD speed instead of SSD journal speed, causing a huge drop in overall cluster performance. When building a cluster, these overall ops and overall load projections should be done and mapped to the HW config to ensure that saturation is unlikely. Factors that influence this are: network capabilities, backing disk performance, size and speed of the journal SSDs, workload intensity and bursting behavior, and overall IO capacity of the system both journal and backing disks.

There are lots of performance studies of Ceph available online, read them and see these trends emerge. However, Ceph is constantly evolving and moving quickly, so each release may impact the specific shape of the curve. There have been attempts to fix the 2x penalty for writes without a journal disk for a while now to facilitate using SSDs as the main data disks and skipping journals altogether. An append-only journal isn't needed given the high random IO performance of an SSD, but the filesystem throughput itself could still be a factor. This area of Ceph performance is expected to evolve rapidly as more users learn Ceph and it becomes more mission critical.







*****

[[category.storage]] 
[[category.confluence]] 
