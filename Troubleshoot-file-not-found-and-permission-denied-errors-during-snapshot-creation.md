
## Problem
EBS snapshots are a global entity. Within Eucalyptus this statement translates to snapshots being available across multiple Eucalyptus clusters regardless of the cluster they originate from. In order to make this happen, the Storage Controller (SC) uploads the snapshot to Object Storage Gateway (S3 equivalent) during snapshot creation. They are downloaded by Storage Controllers in other Eucalyptus clusters on demand (i.e. when a volume is requested from the snapshot in the cluster).

When EBS storage is backed by SANs such as Netapp, EMC or Equallogic, SC orchestrates the creation of snapshot and exports it as a block device to the SC host.It is necessary that SC be able to read from the block device in order to upload the contents to OSG. Eucalyptus provides for udev rules  _(55-openiscsi.rules, 12-dm-permissions.rules)_ and udev script  _(iscsidev.sh)_  that assign the ownership of the block device to the 'eucalyptus' user. If the udev rules/script are missing or the udev script does not account for the storage SAN IQN, the ownership of the block device is not assigned to Eucalyptus and may cause snapshot creation to fail with  _'File not found (permission denied)'_ error.


## Solution
Follow the [[instructions|Manual-installation-of-udev-rules-for-SC-in-source-install]] for installing the udev rules and udev script


## Debug/troubleshooting steps
The following debug steps are to be executed manually  **without the use of Eucalyptus** . This helps isolate the issue and pinpoint it to a specific component. These steps can be also be used to troubleshoot the issue and fix the issues as you go. 


1. Log out of all iSCSI sessions on the SC host logging the error


```
iscsiadm --mode session --logout
```
Delete all the iscsiadm nodeson the system


```
iscsiadm --mode node --op=delete
```

1. Verify that 'eucalyptus' user exists on that system from the output of


```
cat /etc/passwd
```
If eucalyptus is not listed as one of the users, add it 


```
useradd eucalyptus
```

1. Verify that the necessary udev rules and the udev script are in place. Refer to the . 


1. Create a volume on the SAN backend,  **DO**  **NOT**  use Eucalyptus to create the volume


1. Establish an iSCSI connection on the SC host to the SAN volume using [[static targets|iscsiadm-basics]] method


1. After successfully connecting to the SAN volume, check the block device (/dev/sd) of the attachment from the output of
```
iscsiadm -m session -P3
```

1. List the permissions of the block device /dev/sdX and verify that ownership is assigned to 'eucalyptus' user.

If the ownership is not assigned to 'eucalyptus' user, it is safe to assume that the issue might be related to udev rules or udev script. Further debugging of udev may be necessary. 

Follow the steps below to debug udev artifacts responsible for adjusting the ownership of block device attachments


1. Verify that all the necessary udev artifacts discussed in the  are in place
1. Bump up the udev log level to 'DEBUG'. udev generally logs to syslog at /var/log/messages


```
udevadm control --log-priority=debug
```

1. Reload the udev rules by


```
udevadm control --reload-rules
```

1. Repeat 1 through 6 steps in the above section and get the /dev/sd block device of the attachment
1. Look for udev 55-openiscsi.rules invocation in /var/log/messages.If you find the script invocation, proceed to next step

If you don't see the script being invoked,verify that 55-openiscsi.rules matches the /dev/sd block device correctly. This can be checked by triggering udev rules in dry-run mode


```
udevadm trigger --verbose --dry-run --type=devices --subsystem-match=block  
```
/dev/sd block device obtained from step 4 should be listed as one of the matches in the output of the above command. If not, the udev rule may not be accurate. Refer to [55-openiscsi.rules](https://github.com/eucalyptus/eucalyptus/blob/testing/clc/modules/block-storage-common/udev/rules.d/55-openiscsi.rules) in Eucalyptus github repo


1. Check for the return code from the udev rule invocation. If the script does not return success, i.e. 0, there might be an issue with the iscsidev.sh script. Copy the invocation from the log and try executing the iscisdev.sh script by itself to debug it further. Refer to [iscsidev.sh](https://github.com/eucalyptus/eucalyptus/blob/testing/clc/modules/block-storage-common/udev/scripts/iscsidev.sh) script in Eucalyptus github repo


## Related articles
[[Manual installation of udev rules for SC in source install|Manual-installation-of-udev-rules-for-SC-in-source-install]]

[[Connect to iSCSI targets using static target discovery|iscsiadm-basics]]


*****

[[category.storage]] 
[[category.confluence]] 
[[category.ebs]]
