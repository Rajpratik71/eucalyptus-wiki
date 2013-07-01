# Eucalyptus Storage
_Buckets_ and _Volumes_ are the persistent storages within Eucalyptus.
Eucalyptus implement the S3 and EC2 API: more about these storages can be
found at http://aws.amazon.com/ebs/ and http://aws.amazon.com/s3/.

The Eucalyptus component **Walrus** implements Buckets (sometimes intentionally misspelled
bukkits within Eucalyptus), that is the S3 interface. S3 implements a
simple get/put interface: there are quite a few commands/programs that can
be used with Walrus (check out
http://www.eucalyptus.com/eucalyptus-cloud/tools/s3 for a sample of
tools). Files needs can be added to or removed from the bucket: this is
considered and atomic operation, either the full action succeed, in which
case the file will be added/removed, or, in case of failure, the old
content will remain in the bucket.

The **Storage Controller** is the Eucalyptus component implementing the
Elastic Block Storage, which allows instances to use Volumes. Volumes can
be create, deleted, attached to running instances, detached, and
snapshotted using the EC2 interface, thus the euca2ools has all the needed
command to operate on the Volumes. Volumes will appear to the running
instance as a new disk: unlike ephemeral storage, this disk will persist
across instance termination, and instance failures, until the user
explicitly deletes it. 

To recap, both storages are persistent, but buckets are accessed only via
the API or commands, meaning that the application will have to account for
staging the files somewhere else, while volumes will have the familiar
block device interface.

Read more about [[Walrus]] and [[Walrus/S3 API]]

Read more about [[Storage Controller]]

Read more about EBS Instances and Large Instance-Store Instances [[EBS Instances Considerations and Large Instance-Store Images]]

*****
[[category.storage]]
[[category.components]]