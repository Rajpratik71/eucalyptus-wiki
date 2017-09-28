
# Troubleshooting and Configuration of Block Storage in Eucalyptus



## Snapshot upload to Object Storage
The storage controllers use S3 Multipart upload to upload compressed snapshots to Eucalyptus Object Storage to make them available across clusters. MPU is used to allow more efficient storage usage so that the entire snapshot does not have to fit on the SC's disk, just 1 or more "parts" of the snapshot as they are compressed. S3 API requires the uploader to state the upload size at the initiation of the upload, so on-the-fly compression is not possible because the final file size is unknown until compression is completed.

If there are issues using MPU or snapshots are typically very small and may be more efficient to upload directly, you can disable the MPU upload and use a regular S3 object PUT instead. To use regular uploads instead of multipart-uploads, configure the cluster as follows:

```
 euca-modify-property -p <partition>.storage.snapshotpartsizeinmb=500000
```

This will set the SC to only use MPU when the snapshot parts are larger than 5TB, which is the maximum S3 object size. Thus, the SC will never use multipart upload for snapshot uploads.


*****

[[category.storage]] 
[[category.confluence]] 
[[category.ebs]]
