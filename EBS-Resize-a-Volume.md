# Resize an EBS Volume

You can resize an EBS volume by creating a new volume of the target size and copying the contents to the new volume by following these steps. # indicates commands on CLC, $ indicates commands inside a "scratch" instance:

## Requirements

* A "scratch" instance running in your Eucalyptus cloud onto which you can attach volumes. This instance must have the "parted" package installed.
* The source volume to be resized.

## Steps

1. Create a new volume with the target size. Say, 100GB:

 ```
 # euca-create-volume -s 100 -z cluster01
 ```

2. Attach both source and destination volumes to a "scratch" instance:

 ```
 # euca-attach-volume -i i-SCRATCH vol-OLDVOLUME -d /dev/vdc
 # euca-attach-volume -i i-SCRATCH vol-NEWVOLUME -d /dev/vdd
 ```

3. Verify the attachments by logging into the scratch instance and running "fdisk -l" to determine the actual device names of the newly-attached volumes. Then, in the scratch instance, "dd" the contents of source volume (/dev/vdc) to destination volume (/dev/vdd):

 ```
 $ dd if=/dev/sdc of=/dev/sdd bs=1M
 ```

4. Rescan the partitions and then resize the partition and then resize the filesystem:

 ```
 $ partprobe /dev/sdd
 $ parted /dev/sdd rm 1
 $ parted /dev/sdd mkpart primary ext2 0% 100%
 $ e2fsck -f /dev/sdd1
 $ resize2fs /dev/sdd1
 ```

5. If this new volume is intended to be used as an EBS-backed image, log out of the scratch instance, detach the new volume, create a snapshot, and register it:

 ```
 # euca-detach-volume vol-NEWVOLUME
 # euca-create-snapshot vol-NEWVOLUME
 # euca-register -n "resized BFEBS" --root-device-name /dev/vda -b /dev/vda=snap-XXXXXX
 ```

*****

[[category.ebs]]
