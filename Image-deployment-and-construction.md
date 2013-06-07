# Image deployment and construction

Content for instance-store images enters the system with `euca-upload-bundle` operation and becomes an EMI after registration with `euca-register`. Uploading places data into a Walrus bucket, while registration designates the bucket as an image and, in the background, triggers a decryption and caching event so that the image may be ready for deployment by the time the user issues `euca-run-instances` request.

## Instance-store image construction (v3.3)

Instance-store images are downloaded from Walrus's cache (where the uncompressed and decrypted EMIs reside) or from an image cache on the CC (if enabled) to the local disk on each Node Controller. If cache is enabled on the NC and there is sufficient room in it, the image will be downloaded into the cache. Otherwise it will go directly into the instance work directory. 

Both cache and work use sparse files for backing with loopback devices providing block-level access to the data. If a cached image is available, the work image will be a copy-on-write clone of the cached image, using Linux's device mapper. Loopback devices may be in use as long as at least one instance is running. Furthermore, device-mapper devices entries may be in use as long as at least one instance is using an image that was cloned from the cache. 

The following lists the external commands invoked by the NC to provision a Linux partition-based instance (for clarity, we omit some of the non-interesting steps such as creation of directories and setting of ownership and permissions on the artifacts): 

### Initial instance run with empty cache

* First, the EKI is downloaded into cache and copied into work space (snapshots cannot be used as kernel is not a block device and a separate copy for each instance is required by libvirt/KVM)

```
15:54:56 /sbin/losetup -o 0 /dev/loop0 /disk1/storage/eucalyptus/instances/cache/eki-FEFD3946-b23956d5/blocks
15:54:56 /sbin/losetup -o 0 /dev/loop1 /disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-CA6D42F3/eki-FEFD3946-b23956d5.blocks
15:54:57 /bin/dd if=/disk1/storage/eucalyptus/instances/cache/eki-FEFD3946-b23956d5/blocks of=/disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-CA6D42F3/eki-FEFD3946-b23956d5.blocks bs=16 count=309453 seek=0 skip=0 conv=notrunc,fsync
15:54:57 /sbin/losetup -d /dev/loop0
15:54:57 /sbin/losetup -d /dev/loop1
```

* Then, the ERI is downloaded into cache and copied into work space (snapshots cannot be used as ramdisk is not a block device and a separate copy for each instance is required by libvirt/KVM)

```
15:54:57 /sbin/losetup -o 0 /dev/loop0 /disk1/storage/eucalyptus/instances/cache/eri-A55F37FF-91ae3f28/blocks
15:54:58 /sbin/losetup -o 0 /dev/loop1 /disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-CA6D42F3/eri-A55F37FF-91ae3f28.blocks
15:54:58 /bin/dd if=/disk1/storage/eucalyptus/instances/cache/eri-A55F37FF-91ae3f28/blocks of=/disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-CA6D42F3/eri-A55F37FF-91ae3f28.blocks bs=8 count=549981 seek=0 skip=0 conv=notrunc,fsync
15:54:59 /sbin/losetup -d /dev/loop0
15:54:59 /sbin/losetup -d /dev/loop1
```

* Next, the file that will contain the EMI is created in both cache and work, downloaded into cache (thus the ~20-second gap between the first and second lines), and cloned into work space.

```
15:54:59 /sbin/losetup -o 0 /dev/loop0 /disk1/storage/eucalyptus/instances/cache/emi-856C3CDE-ed9bf4f9/blocks
15:55:19 /sbin/losetup -o 0 /dev/loop1 /disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-CA6D42F3/emi-856C3CDE-dc63aa25.blocks
15:55:19 echo "0 2867200 linear /dev/loop1 0" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-emi-856C3CDE-dc63aa25-p0-back
15:55:19 echo "0 2867200 snapshot /dev/loop0 /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-emi-856C3CDE-dc63aa25-p0-back n 16" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-emi-856C3CDE-dc63aa25-p0-snap
15:55:19 echo "0 2867200 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-emi-856C3CDE-dc63aa25-p0-snap 0"
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-emi-856C3CDE-dc63aa25
```

* The work copy is then augmented by 
  * running `tune2fs` (to adjust the number of mounts after which the filesystem will be checked by e2fsck and the maximal time between filesystem checks) and by 
  * copying the user's public SSH key, if supplied, into `/root/.ssh/authorized_keys` on the file system.

```
15:55:19 /sbin/tune2fs /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-emi-856C3CDE-dc63aa25 -c 0 -i 0
15:55:24 mount /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-emi-856C3CDE-dc63aa25 /tmp/euca-mount-N3hC57
15:55:24 /bin/mkdir -p /tmp/euca-mount-N3hC57/root/.ssh
15:55:24 /bin/cp /tmp/euca-temp-0yMy4g /tmp/euca-mount-N3hC57/root/.ssh/authorized_keys
15:55:24 umount /tmp/euca-mount-N3hC57
```

* An ephemeral partition is created in cache space, formatted as 'ext3', and is cloned into work space.

```
15:55:24 /sbin/losetup -o 0 /dev/loop2 /disk1/storage/eucalyptus/instances/cache/prt-03208ext3-bac292b9/blocks
15:55:24 /sbin/mkfs.ext3 -b 4096 /dev/loop2 821248
15:55:26 /sbin/losetup -o 0 /dev/loop3 /disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-CA6D42F3/prt-03208ext3-bac292b9.blocks
15:55:26 echo "0 6569984 linear /dev/loop3 0" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-03208ext3-bac292b9-p0-back
15:55:26 echo "0 6569984 snapshot /dev/loop2 /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-03208ext3-bac292b9-p0-back n 16" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-03208ext3-bac292b9-p0-snap
15:55:26 echo "0 6569984 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-03208ext3-bac292b9-p0-snap 0" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-03208ext3-bac292b9
```

* A swap partition is created in cache space, formatted as 'swap', and is cloned into work space.

```
15:55:27 /sbin/losetup -o 0 /dev/loop4 /disk1/storage/eucalyptus/instances/cache/prt-00512swap-ac8d5670/blocks
15:55:27 /sbin/mkswap /dev/loop4 524288
15:55:27 /sbin/losetup -o 0 /dev/loop5 /disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-CA6D42F3/prt-00512swap-ac8d5670.blocks
15:55:27 echo "0 1048576 linear /dev/loop5 0" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-00512swap-ac8d5670-p0-back
15:55:27 echo "0 1048576 snapshot /dev/loop4 /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-00512swap-ac8d5670-p0-back n 16" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-00512swap-ac8d5670-p0-snap
15:55:27 echo "0 1048576 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-00512swap-ac8d5670-p0-snap 0" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-00512swap-ac8d5670
```

* Finally, the disk device is created that is big enough to host the three partitions as well as a 63-sector header to host the MSDOS-style partition table and the Master Boot Record, if required by a later stage (MBR is not added for KVM disk images to make them bootable since Eucalyptus has a separate kernel & ramdisk to pass to KVM). The disk is backed by a .blocks file in work space, only the first 63 sectors of which can ever be written to since the rest of the device consists of the three partitions mapped in from writable devices in work space. The partition table is created with several `parted` invocations. 

```
15:55:27 /sbin/losetup -o 0 /dev/loop6 /disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-CA6D42F3/dsk-856C3CDE-6bbf5871.blocks
15:55:27 echo "0 63 linear /dev/loop6 0"\
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871-p0-back
15:55:27 echo "0 63 snapshot /dev/mapper/euca-zero /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871-p0-back n 1" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871-p0-snap
15:55:27  echo -e \
"0 63 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871-p0-snap 0\n"\
"63 2867200 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-emi-856C3CDE-dc63aa25 0\n"\
"2867263 6569984 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-03208ext3-bac292b9 0\n"\
"9437247 1048576 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-00512swap-ac8d5670 0"\
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871 
15:55:27 /sbin/parted --script /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871 mklabel msdos
15:55:27 /sbin/parted --script /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871 mkpart primary  63s 2867262s
15:55:28 /sbin/parted --script /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871 mkpart primary  2867263s 9437246s
15:55:28 /sbin/parted --script /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871 mkpart primary  9437247s 10485822s
```

At this point device `/dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871` is used as the root disk for a KVM virtual machine.

### Second instance run with primed cache

If cached artifacts exist from a previous instance invocation, downloading of the EMI and construction and formatting of the swap and ephemeral partitions will not take place upon subsequent instance invocations.

* EKI and ERI are copied from the cache space into work space.

```
12:58:10 /sbin/losetup -o 0 /dev/loop7 /disk1/storage/eucalyptus/instances/cache/eki-FEFD3946-b23956d5/blocks
12:58:10 /sbin/losetup -o 0 /dev/loop8 /disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-3E4B3CE6/eki-FEFD3946-b23956d5.blocks
12:58:10 /bin/dd if=/disk1/storage/eucalyptus/instances/cache/eki-FEFD3946-b23956d5/blocks of=/disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-3E4B3CE6/eki-FEFD3946-b23956d5.blocks bs=16 count=309453 seek=0 skip=0 conv=notrunc,fsync
12:58:11 /sbin/losetup -d /dev/loop7
12:58:11 /sbin/losetup -d /dev/loop8
12:58:11 /sbin/losetup -o 0 /dev/loop7 /disk1/storage/eucalyptus/instances/cache/eri-A55F37FF-91ae3f28/blocks
12:58:11 /sbin/losetup -o 0 /dev/loop8 /disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-3E4B3CE6/eri-A55F37FF-91ae3f28.blocks
12:58:11 /bin/dd if=/disk1/storage/eucalyptus/instances/cache/eri-A55F37FF-91ae3f28/blocks of=/disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-3E4B3CE6/eri-A55F37FF-91ae3f28.blocks bs=8 count=549981 seek=0 skip=0 conv=notrunc,fsync
12:58:12 /sbin/losetup -d /dev/loop7
12:58:12 /sbin/losetup -d /dev/loop8
```

* The EMI downloaded earlier is cloned into work space and augmented with `tune2fs` and the user's SSH key.

```
12:58:12 /sbin/losetup -o 0 /dev/loop7 /disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-3E4B3CE6/emi-856C3CDE-dc63aa25.blocks
12:58:12 echo "0 2867200 linear /dev/loop7 0" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-emi-856C3CDE-dc63aa25-p0-back
12:58:12 echo "0 2867200 snapshot /dev/loop0 /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-emi-856C3CDE-dc63aa25-p0-back n 16" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-emi-856C3CDE-dc63aa25-p0-snap
12:58:12 echo "0 2867200 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-emi-856C3CDE-dc63aa25-p0-snap 0" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-emi-856C3CDE-dc63aa25
12:58:12 /sbin/tune2fs /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-emi-856C3CDE-dc63aa25 -c 0 -i 0
12:58:12 mount /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-emi-856C3CDE-dc63aa25 /tmp/euca-mount-yhuuuA
12:58:12 /bin/mkdir -p /tmp/euca-mount-yhuuuA/root/.ssh
12:58:12 /bin/cp /tmp/euca-temp-1n72lM /tmp/euca-mount-yhuuuA/root/.ssh/authorized_keys
12:58:12 umount /tmp/euca-mount-yhuuuA
```

* Ephemeral partition is cloned from cache to work space.

```
12:58:12 /sbin/losetup -o 0 /dev/loop8 /disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-3E4B3CE6/prt-03208ext3-bac292b9.blocks
12:58:12 echo "0 6569984 linear /dev/loop8 0" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-03208ext3-bac292b9-p0-back
12:58:12 echo "0 6569984 snapshot /dev/loop2 /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-03208ext3-bac292b9-p0-back n 16" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-03208ext3-bac292b9-p0-snap
12:58:12 echo "0 6569984 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-03208ext3-bac292b9-p0-snap 0" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-03208ext3-bac292b9
```

* Swap partition is cloned from cache to work space.

```
12:58:13 /sbin/losetup -o 0 /dev/loop9 /disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-3E4B3CE6/prt-00512swap-ac8d5670.blocks
12:58:13 echo "0 1048576 linear /dev/loop9 0" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-00512swap-ac8d5670-p0-back
12:58:13 echo "0 1048576 snapshot /dev/loop4 /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-00512swap-ac8d5670-p0-back n 16" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-00512swap-ac8d5670-p0-snap
12:58:13 echo "0 1048576 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-00512swap-ac8d5670-p0-snap 0" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-00512swap-ac8d5670
```

* As in the cold-cache scenario, the last step is construction of the disk, mapping of other work-space artifacts into it, and creation of partition table with `parted`.

```
12:58:13 /sbin/losetup -o 0 /dev/loop10 /disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK/i-3E4B3CE6/dsk-856C3CDE-6bbf5871.blocks
12:58:13 echo "0 63 linear /dev/loop10 0" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871-p0-back
12:58:13 echo "0 63 snapshot /dev/mapper/euca-zero /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871-p0-back n 1" \
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871-p0-snap
12:58:13 echo -e \
"0 63 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871-p0-snap 0\n"\
"63 2867200 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-emi-856C3CDE-dc63aa25 0\n"\
"2867263 6569984 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-03208ext3-bac292b9 0\n"\
"9437247 1048576 linear /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-00512swap-ac8d5670 0"\
         | /sbin/dmsetup create euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871
12:58:13 /sbin/parted --script /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871 mklabel msdos
12:58:13 /sbin/parted --script /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871 mkpart primary  63s 2867262s
12:58:13 /sbin/parted --script /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871 mkpart primary  2867263s 9437246s
12:58:13 /sbin/parted --script /dev/mapper/euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871 mkpart primary  9437247s 10485822s
```

At this point, there are 11 loopback devices in use by Eucalyptus: three by the cache-space artifacts (EMI, ephemeral, and swap partition) and four by each instance (the work copies of EMI/root, ephemeral, and swap partitions and the ultimate root disk device):

```
# losetup -a
/dev/loop0: [0803]:7979030 (/disk1/storage/eucalyptus/instances/cache/emi-856C3CDE-ed9bf4f*)
/dev/loop1: [0803]:7979034 (/disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK*)
/dev/loop2: [0803]:8126467 (/disk1/storage/eucalyptus/instances/cache/prt-03208ext3-bac292*)
/dev/loop3: [0803]:7979042 (/disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK*)
/dev/loop4: [0803]:8126473 (/disk1/storage/eucalyptus/instances/cache/prt-00512swap-ac8d56*)
/dev/loop5: [0803]:7979049 (/disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK*)
/dev/loop6: [0803]:7979056 (/disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK*)
/dev/loop7: [0803]:8126488 (/disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK*)
/dev/loop8: [0803]:8126495 (/disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK*)
/dev/loop9: [0803]:8126502 (/disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK*)
/dev/loop10: [0803]:8126509 (/disk1/storage/eucalyptus/instances/work/ZV3GM6S3NMA1RQRNXZREK*)
```

Also, there are 37 device-mapper table entries in use by Eucalyptus:

* 1 for the huge /dev/euca-zero device, to be used for creating zeroed out disk sections
* 18 for each running instance (2 of them)
  * 3 for each snapshot from cache to work of a partition (3 of them)
  * 3 for the snapshot of /dev/euca-zero to create an empty header for partition table
  * 3 for the mapping of work-space partition devices into the final disk device
  * 3 entries for each partition on disk, created automatically by `parted`

```
# dmsetup table | sort
euca-zero: 0 2199023255552 zero 
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871: 0 63 linear 253:26 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871: 2867263 6569984 linear 253:21 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871: 63 2867200 linear 253:18 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871: 9437247 1048576 linear 253:24 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871-p0-back: 0 63 linear 7:10 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871-p0-snap: 0 63 snapshot 253:9 253:25 N 1
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871p1: 0 2867200 linear 253:27 63
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871p2: 0 6569984 linear 253:27 2867263
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-dsk-856C3CDE-6bbf5871p3: 0 1048576 linear 253:27 9437247
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-emi-856C3CDE-dc63aa25: 0 2867200 linear 253:17 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-emi-856C3CDE-dc63aa25-p0-back: 0 2867200 linear 7:7 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-emi-856C3CDE-dc63aa25-p0-snap: 0 2867200 snapshot 7:0 253:16 N 16
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-00512swap-ac8d5670: 0 1048576 linear 253:23 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-00512swap-ac8d5670-p0-back: 0 1048576 linear 7:9 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-00512swap-ac8d5670-p0-snap: 0 1048576 snapshot 7:4 253:22 N 16
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-03208ext3-bac292b9: 0 6569984 linear 253:20 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-03208ext3-bac292b9-p0-back: 0 6569984 linear 7:8 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-3E4B3CE6-prt-03208ext3-bac292b9-p0-snap: 0 6569984 snapshot 7:2 253:19 N 16
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871: 0 63 linear 253:11 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871: 2867263 6569984 linear 253:5 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871: 63 2867200 linear 253:2 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871: 9437247 1048576 linear 253:8 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871-p0-back: 0 63 linear 7:6 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871-p0-snap: 0 63 snapshot 253:9 253:10 N 1
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871p1: 0 2867200 linear 253:12 63
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871p2: 0 6569984 linear 253:12 2867263
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-dsk-856C3CDE-6bbf5871p3: 0 1048576 linear 253:12 9437247
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-emi-856C3CDE-dc63aa25: 0 2867200 linear 253:1 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-emi-856C3CDE-dc63aa25-p0-back: 0 2867200 linear 7:1 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-emi-856C3CDE-dc63aa25-p0-snap: 0 2867200 snapshot 7:0 253:0 N 16
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-00512swap-ac8d5670: 0 1048576 linear 253:7 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-00512swap-ac8d5670-p0-back: 0 1048576 linear 7:5 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-00512swap-ac8d5670-p0-snap: 0 1048576 snapshot 7:4 253:6 N 16
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-03208ext3-bac292b9: 0 6569984 linear 253:4 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-03208ext3-bac292b9-p0-back: 0 6569984 linear 7:3 0
euca-ZV3GM6S3NMA1RQRNXZREK-i-CA6D42F3-prt-03208ext3-bac292b9-p0-snap: 0 6569984 snapshot 7:2 253:3 N 16
```

### State after all instances are terminated on the NC

The only in-kernel artifact left after the instances are terminated is the `euca-zero` device, which is not expensive to keep around and which can be safely removed at any point.

```
# losetup -a
# dmsetup table 
euca-zero: 0 2199023255552 zero
```

*****

[[category.debugging]]