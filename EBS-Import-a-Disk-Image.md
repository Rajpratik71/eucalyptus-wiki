# Import a Disk Image

Import an raw disk image into your cloud as a new EBS volume.

## Requirements

* The disk image file you wish to import into your cloud. Our example will use "disk.img".
* A "scratch" instance running in your cloud to which you can attach volumes.

## Steps

1. Create a new volume of the same size as your disk image file, and attach it to your scratch instance:

 ```
 $ euca-create-volume -s 5 -z cluster01
 $ euca-attach-volume -i i-XXXXX vol-XXXXXX -d /dev/vdc
 ```

2. Determine which actual device the volume attached to by logging into your scratch instance and running "fdisk -l". This example will assume /dev/vdc, but it might be different.

3. Transfer the disk image to the attached volume by using an SSH-based pipe:

 ```
 $ gzip - < disk.img | ssh -i euca-key.priv root@SCRATCH-INSTANCE-IP "zcat > /dev/vdc"
 ```

4. Now you can optionally log into the instance and verify the volume contents. Detach the volume from the instance:

 ```
 $ euca-detach-volume vol-XXXXXX
 ```

5. If your disk image is intended to be an EBS-backed (Boot-From-EBS) image, snapshot the volume and register it like this:

 ```
 $ euca-create-snapshot vol-XXXXXX
 $ euca-register -n "imported disk image" --root-device-name /dev/vda -b /dev/vda=snap-XXXXXX
 ```

## 

*****

[[category.ebs]]
