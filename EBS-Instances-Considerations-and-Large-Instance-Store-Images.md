# Things to Consider When Planning for and Deploying EBS-backed Instances
For EBS images, there are significant performance implications to exactly how your deployment is configured and the network topology and hardware specs of the SC. If you do not have a SAN, then the performance of the SC machine can be the bottleneck for all EBS operations.

Specifically, if you are running the TGT-based storage controller (i.e. not a SAN) then the SC machine itself is hosting *all* iscsi traffic for the partition/availability-zone. For a couple of ebs instances that may not be a problem, but beware that when starting new EBS instances there is a lot of IO on the SC itself and it will have a performance impact on other running EBS instances. Your SC needs to be a powerful machine with lots of RAM and lots of disk IO capacity (note: IOPS is the critical metric here, not storage capacity). SSDs are ideal in this case if you can afford them because you are essentially aggregating a lot of IO onto a single machine, which has the effect of making the SC do a lot of random IO on its disks.

Particularly, be careful about doing a boot-io storm where you 'euca-run-instances -n 20' or something similar because while with ephemeral instances (instance-store) those 20 instances are essentially all isolated, in the boot-from-EBS case all of those instances will be hitting the same SC for boot IO all at the same time and it will likely make the SC very slow and the instances boot slowly.

Also, be aware that disk IO may be competing for network bandwidth with the regular NC and instance traffic. This is an issue we know about for TGT-backed SCs, in the future we will support configuring a distinct network and IP for data traffic on the SC. You will want to ensure that heavy EBS traffic does not saturate the network such that normal eucalyptus control messages (CC->NC for example) begin to fail or that instance traffic is not longer reaching instances. Ideally, you should put the SC on its own network that also connects to the NCs and CLC so that the SC traffic is on a different network than normal Eucalyptus or instance network traffic.

# Startup performance of instance-store instances and large Windows images
The performance bottleneck for large instance-store images is the decryption and transfer cost for the image to the NC on the first instance start of that image on that NC. This cost is only incurred on the first run of the image after registration. Walrus will ensure that an image is decrypted only once and then cached for subsequent use on both Walrus and any NCs that run the image.

## Why are images chunked into 10MB files during bundling and upload?
The reason for the chunking of the image during upload to Walrus is 1.) AWS compliance, and 2.) minimize the transfer time and failure probability for each chunk. The tools can retry a specific chunk of the image during upload rather than having to redo the entire 20GB image if the last 10MB fails.

The reason that both the individual chunks as well as the assembled image are stored is that the image is stored in the image-cache and may be flushed if it is unused and space is needed for other images in the cache. The cached image happens to reside in the same bucket as the uploaded image chunks but it is managed independently as part of the image cache.

[[category.ebs]]
[[category.storage]]

