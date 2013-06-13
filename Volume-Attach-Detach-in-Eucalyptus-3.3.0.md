# Overview of Volume Attach/Detach in Eucalyptus 3.3.0

The message flow and logic for attaching and detaching volumes to instances has changed significantly in Eucalyptus 3.3.0. Specifically, a new communication path is now used and the Storage Controller keeps more metadata about the status of a volume with regard to attachment.

Specifically, the Node Controller now directly requests that a volume be exported to it by sending an ExportVolume request to the SC directly. Previously, the CLC would determine which NC was running an instance and then the CLC would send an AttachVolume request to the SC, which would then make the volume available for connections. The CLC would then send the AttachVolume request to the CC and on the NC, which would actually initiate the iSCSI connection to the SC/SAN, depending on configuration.

## Example: Running an EBS-instance (Attachment)

### Running an EBS-instance in Eucalyptus 3.2.2:
1. (create rootfs volume)
2. CLC -> createVolume(snap_source) -> SC
3. SC -> ok() -> CLC
4. (exportAll rootfs volume)
5. CLC -> attachVolume(all_ncs) -> SC
6. SC -> ok(remote_device_string) -> CLC
7. (run instance)
8. CLC -> run_instance(remote_device_string) -> CC
9. CC -> ok() -> CLC (async call)
10. CC -> run_instance(remote_device_string) -> NC
11. NC -> connect_iscsi(remote_device_string) -> ISCSI target (SC or SAN)
12. (state discovery)
13. CLC -> describeInstances() -> CC
14. CC -> describeInstances() ->* NC (each NC)
15. NC -> instances(i-X, vol-y:Attached).. -> CC
16. CC -> instances() -> CLC
17. CLC -> set_volume_attached(NC) -> DB

![BfEBS messaging in Euca 3.2](images/Euca-3.2-BfEBS_messages.png)

###Running an EBS-instance in Eucalyptus 3.3.0:
1. (create rootfs volume) Unchanged
2. CLC -> createVolume(snap_source) -> SC
3. SC -> ok() -> CLC
4. (get reference for rootfs volume)
5. CLC -> getVolumeToken() -> SC
6. SC -> ok(ref_token) -> CLC
7. (run instance)
8. CLC -> run_instance(ref_token) -> CC
9. CC -> ok() -> CLC (async call)
10. CC -> run_instance(ref_token) -> NC
11. NC -> exportVolume(ref_token) -> SC
12. SC -> exportVolume(NCx) -> SAN/TGT
13. SC -> ok(connect_string) -> NC
14. NC -> connect(connect_string) -> ISCSI target (SC or SAN)
15. (state discovery)
16. CLC -> describeInstances() -> CC
17. CC -> describeInstances() ->* NC (each NC)
18. NC -> instances(i-X, vol-y:Attached).. -> CC
19. CC -> instances() -> CLC
20. CLC -> set_volume_attached(NC) -> DB

![BfEBS messaging in Euca 3.3](images/Euca-3.3-BfEBS_messages.png)

## Detachment
The detach flow is similar, but now requires a lock on volume operations on the NC as well as a two phased detach.

On the NC, assuming vol-X attached as /dev/vdf:
1. Remove /dev/vdf from guest VM.
2. Delete lun from iscsi session (using 'echo 1' to a specific file in /sys/)
3. Call UnexportVolume(vol-X, ip, iqn) on Storage Controller
4. Rescan iscsi session to ensure volume is no longer available
5. Return success.

## Debugging and Common Issues
### Expected Log Entries During Normal Operation

The SC now logs 'Processing <operation> request for <volume|snap>' for all Volume and Snapshot operations at INFO level logging. This is the first thing the SC does once it receives the full request. So, you should be able to see all such log messages for each volume in cloud-output.log on the SC host.

For attachment you should see in cloud-output.log on the SC:

    Processing GetVolumeToken request for vol-X
    ...
    ...
    Processing ExportVolume request for vol-X`

On Detach:
    Processing UnexportVolume request for vol-X

On Delete:
    Processing DeleteVolume request for vol-X`

On EBS-instance run, for the root:
    Processing CreateVolume request for vol-X
    Processing GetVolumeToken request for vol-X
    Processing ExportVolume request for vol-X

On EBS-instance stop:
    Processing UnexportVolume request for vol-X

Termination may add to the 'stop' set:
    Processing DeleteStorageVolume request for vol-X

If any of those are missing then you can see how far the process got and what are the likely sources of failure.
* Missing GetVolumeToken? Check the CLC and CLC->SC communications.
* Missing ExportVolume? Check that the NC got the doAttachVolume request and that the NC can communicate with the SC on port 8773.
* Missing UnexportVolume? Check the NC. It should have a 'doDetachVolume' record in the logs as well as a logged entry of the URL used to contact the SC. If that URL is not http://<registered SC host or IP>:8773/services/Storage then the NC will not have been able to successfully send the message to the SC.

### Common Problems and Likely Causes and Solutions

**1.) Problem: Volume stuck in 'deleting' after delete issued.**

* Possible cause: If a failure occurred on the NC or SC during the last UnexportVolume operation for the affected volume, then the volume may still be exported. In this case, the SC is very conservative and will not delete a volume that is believes is exported based on the DB state it has. You will see in the SC logs that the volume is marked for deletion but is found to be exported so deletion is skipped. This will be repeated in the logs on the SC because the SC is periodically trying to delete the volume but keeps finding it exported. Once the export status is removed the volume will delete normally. This should never happen in normal usage, but may result from network errors or other failures during the UnexportVolume operation on an NC.
* Solution: Verify on the NC and SC that the volume is indeed not connected (iscsiadm on the NC is the easiest way) and then to invalidate the export and token in the eucalyptus_storage database. There is a tool provided in [deveutils git repository](https://github.com/eucalyptus/deveutils) for forcibly unexporting a volume. This should only be used as a last resort, but is preferrable to manual database manipulation. See its documentation for further information on how it works. After running it, the volume should be automatically deleted possibly after a delay (the normal SC delete checks that run every 10 seconds or so).

**2.) Problem: Volume fails to attach to instance, immediate failure, no 'attaching' state**

* Possible cause: CLC is failing to get a token from the SC for the attachment. This is likely the case if the attach-volume request is failing immediately and returning an error to the API caller. The volume will not go to the 'attaching' state if the CLC does not get a token. This can be caused by other failures as well, so some log parsing is necessary. On the SC you should see 'Processing GetVolumeToken request for vol-XYZ' in cloud-output.log. If you do not see this message immediately following the request being issued then there is a problem.
* Solution: Check connectivity to the SC. The CLC should return an error to the user if it cannot find an ENABLED SC to deliver the request to. Also, check DB connectivity on the SC. If the SC cannot read/write to the DB (eucalyptus_storage) then it cannot issue tokens. You should see clear indications of failure in the SC's cloud-error.log file if this is the case.

**3.) Problem: Volume fails to attach to instance, goes to 'attaching' but then back to 'available'**

* Possible cause: NC failing to get successful return on ExportVolume request to SC during attachment. If the volume goes to 'attaching' and then back to 'available' then the NC may be the cause. Check the NC logs and look for ExportVolume failed or any scClientCall lines that indicate opFail=1. The two causes for a failure here are that the SC cannot communicate with the SC (possible it doesn't know which SC to use), or that the SC itself failed during ExportVolume and returned an error the NC to indicate that. In either case, the NC should have log messages indicating that it could not attach and you should see no indication of it ever receiving a local block device for the volume.
* Solution: Check communications between the NC and SC. The NC must be able to connect to port 8773 on the SC and send SOAP requests via TCP. Check in the NC log that it is using the proper SC URL. This URL should be the IP of the SC as registered, and a path of /services/Storage. Firewall and routing rules may be the cause.