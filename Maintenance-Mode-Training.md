## Feature Overview
Maintenance Mode allows Cloud Administrators to perform maintenance on a node controller without interrupting applications or services running on the the cloud.

#### Component level responsibilities
Cloud Controller sends the migration request to Cluster Controller and finds the available resources for the instances using the same scheduler that it use to run/start new instances.
There is no new component added for migration mode. It comes with eucalyptus admin tool. The command for migration is `euca-migrate-instances`.

* Are there any new components being added? 
* What is the flow of a user request? 
* What actions/processes are happening in the background at all times? 

#### User level operation and tooling
Maintenance Mode operations are only to be performed by Eucalyptus admins only. Since this is a live migration, end-user should not have any impact during migration.

For any successful migration request it does not show any output.

```
# evacuate node controller
euca-migrate-instances --source 10.111.1.116

# migrate specific instance to another destination
euca-migrate-instances -i i-38A74228 --dest 10.111.1.119
```

## Administrative Tasks
Maintenance mode works just out of the box.

## Debugging through log messages
* Source NC
```
2013-06-11 08:52:10 DEBUG 000019327 doMigrateInstances       | verifying 1 instance[s] for migration...
2013-06-11 08:52:10 DEBUG 000019327 doMigrateInstances       | verifying instance # 0...
2013-06-11 08:52:10 DEBUG 000019327 doMigrateInstances       | invoked (action=prepare instance[0].{id=i-3624467F src=10.111.1.116 dst=10.111.5.31) creds=present
2013-06-11 08:52:10 DEBUG 000019327 doMigrateInstances       | verifying 1 instance[s] for migration...
2013-06-11 08:52:10 DEBUG 000019327 doMigrateInstances       | verifying instance # 0...
2013-06-11 08:52:10 DEBUG 000019327 doMigrateInstances       | [i-3624467F] proposed migration action 'prepare' (10.111.1.116 > 10.111.5.31) [creds=present]
2013-06-11 08:52:10 DEBUG 000019327 doMigrateInstances       | [i-3624467F] processing instance # 0 (10.111.1.116 > 10.111.5.31)
2013-06-11 08:52:10  INFO 000019327 doMigrateInstances       | [i-3624467F] migration source preparing 10.111.1.116 > 10.111.5.31 [creds=present]
2013-06-11 08:52:10 DEBUG 000019327 generate_migration_keys  | [i-3624467F] regeneration of migration credentials
2013-06-11 08:52:10 DEBUG 000019327 generate_migration_keys  | [i-3624467F] migration key-generator path: '//usr/lib/eucalyptus/euca_rootwrap //usr/share/eucalyptus/generate-migration-keys.sh 10.111.1.116 qrPdfN1Hgri5QlHb restart'
2013-06-11 08:52:11 DEBUG 000019327 generate_migration_keys  | [i-3624467F] migration key generation succeeded
2013-06-11 08:52:11 DEBUG 000019327 doDescribeResource       | returning status=enabled cores=3/4 mem=6743/6999 disk=52/57 iqn=iqn.1994-05.com.redhat:dac76a638b8
2013-06-11 08:52:13 DEBUG 000019421 statfs_path              | path '/disk1/storage/eucalyptus/instances/work' resolved
2013-06-11 08:52:13 DEBUG 000019421 statfs_path              |   to '/disk1/storage/eucalyptus/instances/work' with ID 0
2013-06-11 08:52:13 DEBUG 000019421 statfs_path              |   of size 211378954240 bytes with available 195452882944 bytes
2013-06-11 08:52:13 DEBUG 000019421 statfs_path              | path '/disk1/storage/eucalyptus/instances/cache' resolved
2013-06-11 08:52:13 DEBUG 000019421 statfs_path              |   to '/disk1/storage/eucalyptus/instances/cache' with ID 0
2013-06-11 08:52:13 DEBUG 000019421 statfs_path              |   of size 211378954240 bytes with available 195452882944 bytes
2013-06-11 08:52:17 DEBUG 000019327 doDescribeInstances      | invoked userId=eucalyptus correlationId=UNSET epoch=31 services[0]{.name=CC_112 .type=cluster .uris[0]=http://10.111.1.112:8774/axis2/services/EucalyptusCC}
2013-06-11 08:52:17 DEBUG 000019327 doDescribeInstances      | [i-3624467F] Extant (ready >10.111.5.31) pub=10.111.101.15 vols=
2013-06-11 08:52:17 DEBUG 000019327 doMigrateInstances       | verifying 1 instance[s] for migration...
2013-06-11 08:52:17 DEBUG 000019327 doMigrateInstances       | verifying instance # 0...
2013-06-11 08:52:17 DEBUG 000019327 doMigrateInstances       | invoked (action=commit instance[0].{id=i-3624467F src=10.111.1.116 dst=10.111.5.31) creds=UNSET
2013-06-11 08:52:17 DEBUG 000019327 doMigrateInstances       | verifying 1 instance[s] for migration...
2013-06-11 08:52:17 DEBUG 000019327 doMigrateInstances       | verifying instance # 0...
2013-06-11 08:52:17 DEBUG 000019327 doMigrateInstances       | [i-3624467F] proposed migration action 'commit' (10.111.1.116 > 10.111.5.31) [creds=present]
2013-06-11 08:52:17 DEBUG 000019327 doMigrateInstances       | [i-3624467F] processing instance # 0 (10.111.1.116 > 10.111.5.31)
2013-06-11 08:52:17  INFO 000019327 doMigrateInstances       | [i-3624467F] migration source initiating 10.111.1.116 > 10.111.5.31 [creds=present] (1 of 1 active outgoing migrations)
2013-06-11 08:52:17 DEBUG 000020401 migrating_thread         | [i-3624467F] connecting to remote hypervisor at 'qemu+tls://10.111.5.31/system'
2013-06-11 08:52:17  INFO 000020401 migrating_thread         | [i-3624467F] migrating instance
```

* Destination NC
```
2013-06-11 08:52:11 DEBUG 000018352 doMigrateInstances       | verifying 1 instance[s] for migration...
2013-06-11 08:52:11 DEBUG 000018352 doMigrateInstances       | verifying instance # 0...
2013-06-11 08:52:11 DEBUG 000018352 doMigrateInstances       | invoked (action=prepare instance[0].{id=i-3624467F src=10.111.1.116 dst=10.111.5.31) creds=present
2013-06-11 08:52:11 DEBUG 000018352 doMigrateInstances       | verifying 1 instance[s] for migration...
2013-06-11 08:52:11 DEBUG 000018352 doMigrateInstances       | verifying instance # 0...
2013-06-11 08:52:11 DEBUG 000018352 doMigrateInstances       | [i-3624467F] proposed migration action 'prepare' (10.111.1.116 > 10.111.5.31) [creds=present]
2013-06-11 08:52:11 DEBUG 000018352 doMigrateInstances       | [i-3624467F] processing instance # 0 (10.111.1.116 > 10.111.5.31)
2013-06-11 08:52:11 DEBUG 000018352 save_instance_struct     | [i-3624467F] save_instance_struct: failed to create instance checkpoint at /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/instance.checkpoint
2013-06-11 08:52:11  INFO 000018352 doMigrateInstances       | [i-3624467F] migration destination preparing 10.111.1.116 > 10.111.5.31 [creds=present]
2013-06-11 08:52:11 DEBUG 000018352 doMigrateInstances       | [i-3624467F] authorizing migration source node 10.111.1.116
2013-06-11 08:52:11 DEBUG 000018352 authorize_migration_keys | [i-3624467F] migration key authorization command: '//usr/lib/eucalyptus/euca_rootwrap //usr/share/eucalyptus/authorize-migration-keys.pl -a 10.111.1.116 qrPdfN1Hgri5QlHb'
2013-06-11 08:52:11 DEBUG 000018352 authorize_migration_keys | [i-3624467F] migration key authorization/deauthorization succeeded
2013-06-11 08:52:11 DEBUG 000018352 generate_migration_keys  | [i-3624467F] regeneration of migration credentials
2013-06-11 08:52:11 DEBUG 000018352 generate_migration_keys  | [i-3624467F] migration key-generator path: '//usr/lib/eucalyptus/euca_rootwrap //usr/share/eucalyptus/generate-migration-keys.sh 10.111.5.31 qrPdfN1Hgri5QlHb restart'
2013-06-11 08:52:13 DEBUG 000018352 generate_migration_keys  | [i-3624467F] migration key generation succeeded
2013-06-11 08:52:13 DEBUG 000018352 vnetStartNetworkManaged  | params: vnetconfig=0x7f7629b06010, vlan=580, uuid=UNSET, userName=UNSET, netName=UNSET
2013-06-11 08:52:13  INFO 000018352 ensure_directories_exist | creating path /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F
2013-06-11 08:52:13 DEBUG 000018352 diskutil_ch              | ch(own|mod) '/disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F' *.root 771
2013-06-11 08:52:13 DEBUG 000018352 vbr_alloc_tree           | [i-3624467F] found 2 prereqs and 3 partitions in the VBR
2013-06-11 08:52:13 DEBUG 000018352 art_alloc                | [i-3624467F] allocated artifact 070|i-3624467F size=-1 vbr=(nil) cache=0 file=0
2013-06-11 08:52:13 DEBUG 000018352 walrus_request_timeout   | writing GET output to /tmp/walrus-digest-M2RIzf
2013-06-11 08:52:13  INFO 000018352 walrus_request_timeout   | downloading http://10.111.1.110:8773/services/Walrus/loadbalancer/vmlinuz-2.6.32-358.6.2.el6.x86_64.manifest.xml
2013-06-11 08:52:13 DEBUG 000018352 walrus_request_timeout   | wrote 3503 byte(s) in 1 write(s)
2013-06-11 08:52:13  INFO 000018352 walrus_request_timeout   | downloaded /tmp/walrus-digest-M2RIzf
2013-06-11 08:52:13 DEBUG 000018352 art_alloc                | [i-3624467F] allocated artifact 071|eki-EFDB3939-04813333 size=4045680 vbr=0x7f7650e3ba00 cache=1 file=1
2013-06-11 08:52:13 DEBUG 000018352 art_alloc                | [i-3624467F] allocated artifact 072|eki-EFDB3939-04813333 size=4045680 vbr=0x7f7650e3ba00 cache=0 file=1
2013-06-11 08:52:13 DEBUG 000018352 art_add_dep              | [i-3624467F] added to artifact 072|eki-EFDB3939-04813333 artifact 071|eki-EFDB3939-04813333
2013-06-11 08:52:13 DEBUG 000018352 art_add_dep              | [i-3624467F] added to artifact 070|i-3624467F artifact 072|eki-EFDB3939-04813333
2013-06-11 08:52:13 DEBUG 000018352 walrus_request_timeout   | writing GET output to /tmp/walrus-digest-eXz3qQ
2013-06-11 08:52:13  INFO 000018352 walrus_request_timeout   | downloading http://10.111.1.110:8773/services/Walrus/loadbalancer/initramfs-2.6.32-358.6.2.el6.x86_64.img.manifest.xml
2013-06-11 08:52:13 DEBUG 000018352 walrus_request_timeout   | wrote 3458 byte(s) in 1 write(s)
2013-06-11 08:52:13  INFO 000018352 walrus_request_timeout   | downloaded /tmp/walrus-digest-eXz3qQ
2013-06-11 08:52:13 DEBUG 000018352 art_alloc                | [i-3624467F] allocated artifact 073|eri-D5913452-e0ccaec6 size=6049036 vbr=0x7f7650e3cf28 cache=1 file=1
2013-06-11 08:52:13 DEBUG 000018352 art_alloc                | [i-3624467F] allocated artifact 074|eri-D5913452-e0ccaec6 size=6049036 vbr=0x7f7650e3cf28 cache=0 file=1
2013-06-11 08:52:13 DEBUG 000018352 art_add_dep              | [i-3624467F] added to artifact 074|eri-D5913452-e0ccaec6 artifact 073|eri-D5913452-e0ccaec6
2013-06-11 08:52:13 DEBUG 000018352 art_add_dep              | [i-3624467F] added to artifact 070|i-3624467F artifact 074|eri-D5913452-e0ccaec6
2013-06-11 08:52:13 DEBUG 000018352 walrus_request_timeout   | writing GET output to /tmp/walrus-digest-EdfTqr
2013-06-11 08:52:13  INFO 000018352 walrus_request_timeout   | downloading http://10.111.1.110:8773/services/Walrus/loadbalancer/eucalyptus-load-balancer-image-1.0.0-92.img.manifest.xml
2013-06-11 08:52:13 DEBUG 000018352 walrus_request_timeout   | wrote 6046 byte(s) in 1 write(s)
2013-06-11 08:52:13  INFO 000018352 walrus_request_timeout   | downloaded /tmp/walrus-digest-EdfTqr
2013-06-11 08:52:13 DEBUG 000018352 art_alloc                | [i-3624467F] allocated artifact 075|emi-904B38A6-eeff8afb size=549453824 vbr=0x7f7650e3a4d8 cache=0 file=0
2013-06-11 08:52:13 DEBUG 000018352 art_alloc                | [i-3624467F] allocated artifact 076|emi-904B38A6-a76b7851 size=549453824 vbr=0x7f7650e3a4d8 cache=0 file=0
2013-06-11 08:52:13 DEBUG 000018352 art_add_dep              | [i-3624467F] added to artifact 076|emi-904B38A6-a76b7851 artifact 075|emi-904B38A6-eeff8afb
2013-06-11 08:52:13 DEBUG 000018352 art_alloc                | [i-3624467F] allocated artifact 077|prt-04084ext3-e6c8efd7 size=4282384384 vbr=0x7f7650e38fb0 cache=0 file=0
2013-06-11 08:52:13 DEBUG 000018352 art_alloc                | [i-3624467F] allocated artifact 078|prt-04084ext3-e6c8efd7 size=4282384384 vbr=0x7f7650e38fb0 cache=0 file=0
2013-06-11 08:52:13 DEBUG 000018352 art_add_dep              | [i-3624467F] added to artifact 078|prt-04084ext3-e6c8efd7 artifact 077|prt-04084ext3-e6c8efd7
2013-06-11 08:52:13 DEBUG 000018352 art_alloc                | [i-3624467F] allocated artifact 079|prt-00512swap-ac8d5670 size=536870912 vbr=0x7f7650e37a88 cache=0 file=0
2013-06-11 08:52:13 DEBUG 000018352 art_alloc                | [i-3624467F] allocated artifact 080|prt-00512swap-ac8d5670 size=536870912 vbr=0x7f7650e37a88 cache=0 file=0
2013-06-11 08:52:13 DEBUG 000018352 art_add_dep              | [i-3624467F] added to artifact 080|prt-00512swap-ac8d5670 artifact 079|prt-00512swap-ac8d5670
2013-06-11 08:52:13 DEBUG 000018352 art_alloc                | [i-3624467F] allocated artifact 081|dsk-904B38A6-f60db8a9 size=5368741376 vbr=0x7f7650e3e450 cache=0 file=0
2013-06-11 08:52:13 DEBUG 000018352 art_add_dep              | [i-3624467F] added to artifact 081|dsk-904B38A6-f60db8a9 artifact 076|emi-904B38A6-a76b7851
2013-06-11 08:52:13 DEBUG 000018352 art_add_dep              | [i-3624467F] added to artifact 081|dsk-904B38A6-f60db8a9 artifact 078|prt-04084ext3-e6c8efd7
2013-06-11 08:52:13 DEBUG 000018352 art_add_dep              | [i-3624467F] added to artifact 081|dsk-904B38A6-f60db8a9 artifact 080|prt-00512swap-ac8d5670
2013-06-11 08:52:13 DEBUG 000018352 art_add_dep              | [i-3624467F] added to artifact 070|i-3624467F artifact 081|dsk-904B38A6-f60db8a9
2013-06-11 08:52:13 DEBUG 000018352 art_print_tree           | [i-3624467F] artifacts tree: 070|i-3624467F cache=0 file=0 creator=(nil) vbr=(nil)
2013-06-11 08:52:13 DEBUG 000018352 art_print_tree           | [i-3624467F] artifacts tree: 	072|eki-EFDB3939-04813333 cache=0 file=1 creator=0x7f7646fd8770 vbr=0x7f7650e3ba00
2013-06-11 08:52:13 DEBUG 000018352 art_print_tree           | [i-3624467F] artifacts tree: 		071|eki-EFDB3939-04813333 cache=1 file=1 creator=0x7f7646fd9850 vbr=0x7f7650e3ba00
2013-06-11 08:52:13 DEBUG 000018352 art_print_tree           | [i-3624467F] artifacts tree: 	074|eri-D5913452-e0ccaec6 cache=0 file=1 creator=0x7f7646fd8770 vbr=0x7f7650e3cf28
2013-06-11 08:52:13 DEBUG 000018352 art_print_tree           | [i-3624467F] artifacts tree: 		073|eri-D5913452-e0ccaec6 cache=1 file=1 creator=0x7f7646fd9850 vbr=0x7f7650e3cf28
2013-06-11 08:52:13 DEBUG 000018352 art_print_tree           | [i-3624467F] artifacts tree: 	081|dsk-904B38A6-f60db8a9 cache=0 file=0 creator=0x7f7646fd9d50 vbr=0x7f7650e3e450
2013-06-11 08:52:13 DEBUG 000018352 art_print_tree           | [i-3624467F] artifacts tree: 		076|emi-904B38A6-a76b7851 cache=0 file=0 creator=0x7f7646fd8770 vbr=0x7f7650e3a4d8
2013-06-11 08:52:13 DEBUG 000018352 art_print_tree           | [i-3624467F] artifacts tree: 			075|emi-904B38A6-eeff8afb cache=0 file=0 creator=0x7f7646fd9850 vbr=0x7f7650e3a4d8
2013-06-11 08:52:13 DEBUG 000018352 art_print_tree           | [i-3624467F] artifacts tree: 		078|prt-04084ext3-e6c8efd7 cache=0 file=0 creator=0x7f7646fd8770 vbr=0x7f7650e38fb0
2013-06-11 08:52:13 DEBUG 000018352 art_print_tree           | [i-3624467F] artifacts tree: 			077|prt-04084ext3-e6c8efd7 cache=0 file=0 creator=0x7f7646fd9400 vbr=0x7f7650e38fb0
2013-06-11 08:52:13 DEBUG 000018352 art_print_tree           | [i-3624467F] artifacts tree: 		080|prt-00512swap-ac8d5670 cache=0 file=0 creator=0x7f7646fd8770 vbr=0x7f7650e37a88
2013-06-11 08:52:13 DEBUG 000018352 art_print_tree           | [i-3624467F] artifacts tree: 			079|prt-00512swap-ac8d5670 cache=0 file=0 creator=0x7f7646fd9400 vbr=0x7f7650e37a88
2013-06-11 08:52:13 DEBUG 000018352 art_implement_tree       | [i-3624467F] implementing artifact 070|i-3624467F
2013-06-11 08:52:13 DEBUG 000018352 art_implement_tree       | [i-3624467F] implementing artifact 072|eki-EFDB3939-04813333
2013-06-11 08:52:13 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 072|eki-EFDB3939-04813333 (do_create=0 ret=1)
2013-06-11 08:52:13 DEBUG 000018352 art_implement_tree       | [i-3624467F] implementing artifact 071|eki-EFDB3939-04813333
2013-06-11 08:52:13  INFO 000018352 ensure_directories_exist | creating path /disk1/storage/eucalyptus/instances/cache/eki-EFDB3939-04813333
2013-06-11 08:52:13 DEBUG 000018352 diskutil_ch              | ch(own|mod) '/disk1/storage/eucalyptus/instances/cache/eki-EFDB3939-04813333' *.* 771
2013-06-11 08:52:13 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 071|eki-EFDB3939-04813333 (do_create=0 ret=2)
2013-06-11 08:52:13  INFO 000018352 ensure_directories_exist | creating path /disk1/storage/eucalyptus/instances/cache/eki-EFDB3939-04813333
2013-06-11 08:52:13 DEBUG 000018352 diskutil_ch              | ch(own|mod) '/disk1/storage/eucalyptus/instances/cache/eki-EFDB3939-04813333' *.* 771
2013-06-11 08:52:13 DEBUG 000018352 diskutil_loop            | attaching file /disk1/storage/eucalyptus/instances/cache/eki-EFDB3939-04813333/blocks
2013-06-11 08:52:13 DEBUG 000018352 diskutil_loop            |             to /dev/loop0 at offset 0
2013-06-11 08:52:13 DEBUG 000018352 art_implement_tree       | [i-3624467F] created a blob for an artifact 071|eki-EFDB3939-04813333 on try 1
2013-06-11 08:52:13  INFO 000018352 walrus_creator           | [i-3624467F] downloading http://10.111.1.110:8773/services/Walrus/loadbalancer/vmlinuz-2.6.32-358.6.2.el6.x86_64.manifest.xml
2013-06-11 08:52:13 DEBUG 000018352 walrus_request_timeout   | writing GET/GetDecryptedImage output
2013-06-11 08:52:13 DEBUG 000018352 walrus_request_timeout   |         from http://10.111.1.110:8773/services/Walrus/loadbalancer/vmlinuz-2.6.32-358.6.2.el6.x86_64.manifest.xml
2013-06-11 08:52:13 DEBUG 000018352 walrus_request_timeout   |         to /disk1/storage/eucalyptus/instances/cache/eki-EFDB3939-04813333/blocks
2013-06-11 08:52:13  INFO 000018352 walrus_request_timeout   | downloading http://10.111.1.110:8773/services/Walrus/loadbalancer/vmlinuz-2.6.32-358.6.2.el6.x86_64.manifest.xml
2013-06-11 08:52:13 DEBUG 000018352 walrus_request_timeout   | wrote 4045680 byte(s) in 452 write(s)
2013-06-11 08:52:13  INFO 000018352 walrus_request_timeout   | downloaded /disk1/storage/eucalyptus/instances/cache/eki-EFDB3939-04813333/blocks
2013-06-11 08:52:13 DEBUG 000018352 art_implement_tree       | [i-3624467F] implemented artifact 071|eki-EFDB3939-04813333 on try 1
2013-06-11 08:52:13 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 072|eki-EFDB3939-04813333 (do_create=1 ret=1)
2013-06-11 08:52:13 DEBUG 000018352 diskutil_loop            | attaching file /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/eki-EFDB3939-04813333.blocks
2013-06-11 08:52:13 DEBUG 000018352 diskutil_loop            |             to /dev/loop1 at offset 0
2013-06-11 08:52:13 DEBUG 000018352 art_implement_tree       | [i-3624467F] created a blob for an artifact 072|eki-EFDB3939-04813333 on try 1
2013-06-11 08:52:13  INFO 000018352 copy_creator             | [i-3624467F] copying/cloning blob eki-EFDB3939-04813333 to blob 4DK9MZTO9LZYIRK8BKSS4/i-3624467F/eki-EFDB3939-04813333
2013-06-11 08:52:13  INFO 000018352 diskutil_dd2             | copying data from '/disk1/storage/eucalyptus/instances/cache/eki-EFDB3939-04813333/blocks'
2013-06-11 08:52:13  INFO 000018352 diskutil_dd2             |                to '/disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/eki-EFDB3939-04813333.blocks'
2013-06-11 08:52:13  INFO 000018352 diskutil_dd2             |                of 252855 blocks (bs=16), seeking 0, skipping 0
2013-06-11 08:52:14 DEBUG 000018352 diskutil_ch              | ch(own|mod) '/disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/eki-EFDB3939-04813333.blocks' *.* 664
2013-06-11 08:52:14 DEBUG 000018352 update_vbr_with_backing_ | [i-3624467F] symlinked /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/eki-EFDB3939-04813333.blocks to /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/link-to-kernel
2013-06-11 08:52:14 DEBUG 000018352 diskutil_unloop          | detaching from loop device /dev/loop0
2013-06-11 08:52:14 DEBUG 000018352 art_implement_tree       | [i-3624467F] implemented artifact 072|eki-EFDB3939-04813333 on try 1
2013-06-11 08:52:14 DEBUG 000018352 diskutil_unloop          | detaching from loop device /dev/loop1
2013-06-11 08:52:14 DEBUG 000018352 art_implement_tree       | [i-3624467F] implementing artifact 074|eri-D5913452-e0ccaec6
2013-06-11 08:52:14 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 074|eri-D5913452-e0ccaec6 (do_create=0 ret=1)
2013-06-11 08:52:14 DEBUG 000018352 art_implement_tree       | [i-3624467F] implementing artifact 073|eri-D5913452-e0ccaec6
2013-06-11 08:52:14  INFO 000018352 ensure_directories_exist | creating path /disk1/storage/eucalyptus/instances/cache/eri-D5913452-e0ccaec6
2013-06-11 08:52:14 DEBUG 000018352 diskutil_ch              | ch(own|mod) '/disk1/storage/eucalyptus/instances/cache/eri-D5913452-e0ccaec6' *.* 771
2013-06-11 08:52:14 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 073|eri-D5913452-e0ccaec6 (do_create=0 ret=2)
2013-06-11 08:52:14  INFO 000018352 ensure_directories_exist | creating path /disk1/storage/eucalyptus/instances/cache/eri-D5913452-e0ccaec6
2013-06-11 08:52:14 DEBUG 000018352 diskutil_ch              | ch(own|mod) '/disk1/storage/eucalyptus/instances/cache/eri-D5913452-e0ccaec6' *.* 771
2013-06-11 08:52:14 DEBUG 000018352 diskutil_loop            | attaching file /disk1/storage/eucalyptus/instances/cache/eri-D5913452-e0ccaec6/blocks
2013-06-11 08:52:14 DEBUG 000018352 diskutil_loop            |             to /dev/loop0 at offset 0
2013-06-11 08:52:14 DEBUG 000018352 art_implement_tree       | [i-3624467F] created a blob for an artifact 073|eri-D5913452-e0ccaec6 on try 1
2013-06-11 08:52:14  INFO 000018352 walrus_creator           | [i-3624467F] downloading http://10.111.1.110:8773/services/Walrus/loadbalancer/initramfs-2.6.32-358.6.2.el6.x86_64.img.manifest.xml
2013-06-11 08:52:14 DEBUG 000018352 walrus_request_timeout   | writing GET/GetDecryptedImage output
2013-06-11 08:52:14 DEBUG 000018352 walrus_request_timeout   |         from http://10.111.1.110:8773/services/Walrus/loadbalancer/initramfs-2.6.32-358.6.2.el6.x86_64.img.manifest.xml
2013-06-11 08:52:14 DEBUG 000018352 walrus_request_timeout   |         to /disk1/storage/eucalyptus/instances/cache/eri-D5913452-e0ccaec6/blocks
2013-06-11 08:52:14  INFO 000018352 walrus_request_timeout   | downloading http://10.111.1.110:8773/services/Walrus/loadbalancer/initramfs-2.6.32-358.6.2.el6.x86_64.img.manifest.xml
2013-06-11 08:52:14 DEBUG 000018352 walrus_request_timeout   | wrote 6049036 byte(s) in 664 write(s)
2013-06-11 08:52:14  INFO 000018352 walrus_request_timeout   | downloaded /disk1/storage/eucalyptus/instances/cache/eri-D5913452-e0ccaec6/blocks
2013-06-11 08:52:14 DEBUG 000018352 art_implement_tree       | [i-3624467F] implemented artifact 073|eri-D5913452-e0ccaec6 on try 1
2013-06-11 08:52:14 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 074|eri-D5913452-e0ccaec6 (do_create=1 ret=1)
2013-06-11 08:52:14 DEBUG 000018352 diskutil_loop            | attaching file /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/eri-D5913452-e0ccaec6.blocks
2013-06-11 08:52:14 DEBUG 000018352 diskutil_loop            |             to /dev/loop1 at offset 0
2013-06-11 08:52:14 DEBUG 000018352 art_implement_tree       | [i-3624467F] created a blob for an artifact 074|eri-D5913452-e0ccaec6 on try 1
2013-06-11 08:52:14  INFO 000018352 copy_creator             | [i-3624467F] copying/cloning blob eri-D5913452-e0ccaec6 to blob 4DK9MZTO9LZYIRK8BKSS4/i-3624467F/eri-D5913452-e0ccaec6
2013-06-11 08:52:14  INFO 000018352 diskutil_dd2             | copying data from '/disk1/storage/eucalyptus/instances/cache/eri-D5913452-e0ccaec6/blocks'
2013-06-11 08:52:14  INFO 000018352 diskutil_dd2             |                to '/disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/eri-D5913452-e0ccaec6.blocks'
2013-06-11 08:52:14  INFO 000018352 diskutil_dd2             |                of 1512259 blocks (bs=4), seeking 0, skipping 0
2013-06-11 08:52:16 DEBUG 000018352 diskutil_ch              | ch(own|mod) '/disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/eri-D5913452-e0ccaec6.blocks' *.* 664
2013-06-11 08:52:16 DEBUG 000018352 update_vbr_with_backing_ | [i-3624467F] symlinked /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/eri-D5913452-e0ccaec6.blocks to /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/link-to-ramdisk
2013-06-11 08:52:16 DEBUG 000018352 diskutil_unloop          | detaching from loop device /dev/loop0
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implemented artifact 074|eri-D5913452-e0ccaec6 on try 1
2013-06-11 08:52:16 DEBUG 000018352 diskutil_unloop          | detaching from loop device /dev/loop1
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implementing artifact 081|dsk-904B38A6-f60db8a9
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 081|dsk-904B38A6-f60db8a9 (do_create=0 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implementing artifact 076|emi-904B38A6-a76b7851
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 076|emi-904B38A6-a76b7851 (do_create=0 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implementing artifact 075|emi-904B38A6-eeff8afb
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 075|emi-904B38A6-eeff8afb (do_create=0 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 075|emi-904B38A6-eeff8afb (do_create=1 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 diskutil_loop            | attaching file /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/emi-904B38A6-eeff8afb.blocks
2013-06-11 08:52:16 DEBUG 000018352 diskutil_loop            |             to /dev/loop0 at offset 0
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] created a blob for an artifact 075|emi-904B38A6-eeff8afb on try 1
2013-06-11 08:52:16  INFO 000018352 walrus_creator           | [i-3624467F] skipping download of http://10.111.1.110:8773/services/Walrus/loadbalancer/eucalyptus-load-balancer-image-1.0.0-92.img.manifest.xml
2013-06-11 08:52:16 DEBUG 000018352 update_vbr_with_backing_ | [i-3624467F] symlinked /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/emi-904B38A6-eeff8afb.blocks to /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/link-to-sda1
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implemented artifact 075|emi-904B38A6-eeff8afb on try 1
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 076|emi-904B38A6-a76b7851 (do_create=1 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 diskutil_loop            | attaching file /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/emi-904B38A6-a76b7851.blocks
2013-06-11 08:52:16 DEBUG 000018352 diskutil_loop            |             to /dev/loop1 at offset 0
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] created a blob for an artifact 076|emi-904B38A6-a76b7851 on try 1
2013-06-11 08:52:16  INFO 000018352 copy_creator             | [i-3624467F] skipping copying to 4DK9MZTO9LZYIRK8BKSS4/i-3624467F/emi-904B38A6-a76b7851
2013-06-11 08:52:16 DEBUG 000018352 diskutil_unloop          | detaching from loop device /dev/loop0
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implemented artifact 076|emi-904B38A6-a76b7851 on try 1
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implementing artifact 078|prt-04084ext3-e6c8efd7
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 078|prt-04084ext3-e6c8efd7 (do_create=0 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implementing artifact 077|prt-04084ext3-e6c8efd7
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 077|prt-04084ext3-e6c8efd7 (do_create=0 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 077|prt-04084ext3-e6c8efd7 (do_create=1 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 diskutil_loop            | attaching file /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/prt-04084ext3-e6c8efd7.blocks
2013-06-11 08:52:16 DEBUG 000018352 diskutil_loop            |             to /dev/loop0 at offset 0
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] created a blob for an artifact 077|prt-04084ext3-e6c8efd7 on try 1
2013-06-11 08:52:16  INFO 000018352 partition_creator        | [i-3624467F] skipping formatting of prt-04084ext3-e6c8efd7
2013-06-11 08:52:16 DEBUG 000018352 update_vbr_with_backing_ | [i-3624467F] symlinked /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/prt-04084ext3-e6c8efd7.blocks to /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/link-to-sda2
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implemented artifact 077|prt-04084ext3-e6c8efd7 on try 1
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 078|prt-04084ext3-e6c8efd7 (do_create=1 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 diskutil_unloop          | detaching from loop device /dev/loop0
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 078|prt-04084ext3-e6c8efd7 (do_create=0 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 diskutil_loop            | attaching file /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/prt-04084ext3-e6c8efd7.blocks
2013-06-11 08:52:16 DEBUG 000018352 diskutil_loop            |             to /dev/loop0 at offset 0
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] found existing artifact 078|prt-04084ext3-e6c8efd7 on try 2
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implemented artifact 078|prt-04084ext3-e6c8efd7 on try 2
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implementing artifact 080|prt-00512swap-ac8d5670
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 080|prt-00512swap-ac8d5670 (do_create=0 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implementing artifact 079|prt-00512swap-ac8d5670
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 079|prt-00512swap-ac8d5670 (do_create=0 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 079|prt-00512swap-ac8d5670 (do_create=1 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 diskutil_loop            | attaching file /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/prt-00512swap-ac8d5670.blocks
2013-06-11 08:52:16 DEBUG 000018352 diskutil_loop            |             to /dev/loop2 at offset 0
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] created a blob for an artifact 079|prt-00512swap-ac8d5670 on try 1
2013-06-11 08:52:16  INFO 000018352 partition_creator        | [i-3624467F] skipping formatting of prt-00512swap-ac8d5670
2013-06-11 08:52:16 DEBUG 000018352 update_vbr_with_backing_ | [i-3624467F] symlinked /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/prt-00512swap-ac8d5670.blocks to /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/link-to-sda3
2013-06-11 08:52:16 DEBUG 000018352 art_implement_tree       | [i-3624467F] implemented artifact 079|prt-00512swap-ac8d5670 on try 1
2013-06-11 08:52:16 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 080|prt-00512swap-ac8d5670 (do_create=1 ret=1)
2013-06-11 08:52:16 DEBUG 000018352 diskutil_unloop          | detaching from loop device /dev/loop2
2013-06-11 08:52:17 DEBUG 000018446 statfs_path              | path '/disk1/storage/eucalyptus/instances/work' resolved
2013-06-11 08:52:17 DEBUG 000018446 statfs_path              |   to '/disk1/storage/eucalyptus/instances/work' with ID 0
2013-06-11 08:52:17 DEBUG 000018446 statfs_path              |   of size 211378954240 bytes with available 196419121152 bytes
2013-06-11 08:52:17 DEBUG 000018446 statfs_path              | path '/disk1/storage/eucalyptus/instances/cache' resolved
2013-06-11 08:52:17 DEBUG 000018446 statfs_path              |   to '/disk1/storage/eucalyptus/instances/cache' with ID 0
2013-06-11 08:52:17 DEBUG 000018446 statfs_path              |   of size 211378954240 bytes with available 196419117056 bytes
2013-06-11 08:52:17 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 080|prt-00512swap-ac8d5670 (do_create=0 ret=1)
2013-06-11 08:52:17 DEBUG 000018352 diskutil_loop            | attaching file /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/prt-00512swap-ac8d5670.blocks
2013-06-11 08:52:17 DEBUG 000018352 diskutil_loop            |             to /dev/loop2 at offset 0
2013-06-11 08:52:17 DEBUG 000018352 art_implement_tree       | [i-3624467F] found existing artifact 080|prt-00512swap-ac8d5670 on try 2
2013-06-11 08:52:17 DEBUG 000018352 art_implement_tree       | [i-3624467F] implemented artifact 080|prt-00512swap-ac8d5670 on try 2
2013-06-11 08:52:17 DEBUG 000018352 find_or_create_artifact  | [i-3624467F] checking work blobstore for 081|dsk-904B38A6-f60db8a9 (do_create=1 ret=1)
2013-06-11 08:52:17 DEBUG 000018352 diskutil_loop            | attaching file /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/dsk-904B38A6-f60db8a9.blocks
2013-06-11 08:52:17 DEBUG 000018352 diskutil_loop            |             to /dev/loop3 at offset 0
2013-06-11 08:52:17 DEBUG 000018352 art_implement_tree       | [i-3624467F] created a blob for an artifact 081|dsk-904B38A6-f60db8a9 on try 1
2013-06-11 08:52:17 DEBUG 000018352 disk_creator             | [i-3624467F] offset for the first partition will be 32256 bytes
2013-06-11 08:52:17 DEBUG 000018352 disk_creator             | [i-3624467F] mapping partition 1 from /dev/loop1 [63-1073214]
2013-06-11 08:52:17 DEBUG 000018352 disk_creator             | [i-3624467F] mapping partition 2 from /dev/loop0 [1073215-9437246]
2013-06-11 08:52:17 DEBUG 000018352 disk_creator             | [i-3624467F] mapping partition 3 from /dev/loop2 [9437247-10485822]
2013-06-11 08:52:17  INFO 000018352 disk_creator             | [i-3624467F] skipping construction of dsk-904B38A6-f60db8a9
2013-06-11 08:52:17 DEBUG 000018352 update_vbr_with_backing_ | [i-3624467F] symlinked /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/dsk-904B38A6-f60db8a9.blocks to /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/link-to-sda
2013-06-11 08:52:17 DEBUG 000018352 diskutil_unloop          | detaching from loop device /dev/loop1
2013-06-11 08:52:17 DEBUG 000018352 diskutil_unloop          | detaching from loop device /dev/loop0
2013-06-11 08:52:17 DEBUG 000018352 diskutil_unloop          | detaching from loop device /dev/loop2
2013-06-11 08:52:17 DEBUG 000018352 art_implement_tree       | [i-3624467F] implemented artifact 081|dsk-904B38A6-f60db8a9 on try 1
2013-06-11 08:52:17 DEBUG 000018352 diskutil_unloop          | detaching from loop device /dev/loop3
2013-06-11 08:52:17 DEBUG 000018352 art_implement_tree       | [i-3624467F] implemented artifact 070|i-3624467F on try 1
2013-06-11 08:52:17 DEBUG 000018352 write_xml_file           | [i-3624467F] wrote instance XML to /disk1/storage/eucalyptus/instances/work/4DK9MZTO9LZYIRK8BKSS4/i-3624467F/instance.xml
2013-06-11 08:52:17 DEBUG 000018352 change_state             | [i-3624467F] state change for instance: Unknown -> Booting (Extant)
2013-06-11 08:52:17  INFO 000018352 doMigrateInstances       | [i-3624467F] migration destination ready 10.111.1.116 > 10.111.5.31
2013-06-11 08:52:17 DEBUG 000018352 doDescribeResource       | returning status=enabled cores=3/4 mem=7536/7792 disk=52/57 iqn=iqn.1994-05.com.redhat:d490eaa66c9
2013-06-11 08:52:17 DEBUG 000018352 doDescribeInstances      | invoked userId=eucalyptus correlationId=UNSET epoch=31 services[0]{.name=CC_112 .type=cluster .uris[0]=http://10.111.1.112:8774/axis2/services/EucalyptusCC}
2013-06-11 08:52:17 DEBUG 000018352 doDescribeInstances      | [i-3624467F] Extant (ready <10.111.1.116) pub=10.111.101.15 vols=
2013-06-11 08:52:22  INFO 000018446 refresh_instance_info    | [i-3624467F] incoming (10.111.5.31 < 10.111.1.116) migration in progress (1 of 1)
2013-06-11 08:52:22 DEBUG 000018446 refresh_instance_info    | [i-3624467F] incoming (10.111.5.31 < 10.111.1.116) migration_state set to 'migrating'
2013-06-11 08:52:22 DEBUG 000018446 change_state             | [i-3624467F] state change for instance: Booting -> Paused (Extant)
```

* Cluster Controller Log
```
2013-06-11 08:52:10  INFO 000018028 doMigrateInstances       | preparing migration from node 10.111.1.116
2013-06-11 08:52:10 DEBUG 000018028 doMigrateInstances       | [i-3624467F] scheduling instance with a destination-exclusion list
2013-06-11 08:52:10 DEBUG 000018028 schedule_instance_migrat | invoked: include=0, exclude=0
2013-06-11 08:52:10 DEBUG 000018028 schedule_instance_migrat | [i-3624467F] scheduling migration using ROUNDROBIN scheduler
2013-06-11 08:52:10 DEBUG 000018028 schedule_instance_roundr | scheduler using ROUNDROBIN policy to find next resource
2013-06-11 08:52:10 DEBUG 000018028 schedule_instance_roundr | scheduler state starting at resource 1
2013-06-11 08:52:10 DEBUG 000018028 schedule_instance_roundr | scheduler state finishing at resource 0
2013-06-11 08:52:10 DEBUG 000018028 schedule_instance_migrat | [i-3624467F] scheduled: src_index=0, dst_index=2 (10.111.1.116 > 10.111.5.31)
2013-06-11 08:52:10 DEBUG 000018028 schedule_instance_migrat | done
2013-06-11 08:52:10  INFO 000018028 doMigrateInstances       | [i-3624467F] scheduled instance migration from 10.111.1.116 to 10.111.5.31
2013-06-11 08:52:10 DEBUG 000018028 doMigrateInstances       | about to ncClientCall source node '10.111.1.116' with nc_instances (prepare 1) [creds='qrPdfN1Hgri5QlHb'] i-3624467F
2013-06-11 08:52:10 DEBUG 000026457 ncMigrateInstancesStub   | marshalling 1 instance(s) [0].id=i-3624467F with action prepare
2013-06-11 08:52:10 DEBUG 000018029 init_thread              | init=1 0x7f2914009000 0x7f28916de000 0x7f28c8321000 0x7f28c811b000
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | invoked: userId=UNSET, serviceIdsLen=1
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | received input serviceId[0]: node PARTI00 10.111.5.31 http://10.111.5.31:8775/axis2/services/EucalyptusNC
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal serviceInfos type=cluster name=CC_112 partition=PARTI00 urisLen=1
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal serviceInfos     uri[0]:http://10.111.1.112:8774/axis2/services/EucalyptusCC
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal serviceInfos type=dns name=10.111.1.110 partition=eucalyptus urisLen=1
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal serviceInfos     uri[0]:http://10.111.1.110:8773/services/Dns
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal serviceInfos type=eucalyptus name=10.111.1.110 partition=eucalyptus urisLen=1
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal serviceInfos     uri[0]:http://10.111.1.110:8773/services/Eucalyptus
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal serviceInfos type=storage name=SC_110 partition=PARTI00 urisLen=1
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal serviceInfos     uri[0]:http://10.111.1.110:8773/services/Storage
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal serviceInfos type=walrus name=WS_110 partition=walrus urisLen=1
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal serviceInfos     uri[0]:http://10.111.1.110:8773/services/Walrus
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal disabled serviceInfos type=eucalyptus name=10.111.1.112 partition=eucalyptus urisLen=1
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal disabled serviceInfos    uri[0]:http://10.111.1.112:8773/services/Eucalyptus
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal disabled serviceInfos type=storage name=SC_112 partition=PARTI00 urisLen=1
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal disabled serviceInfos    uri[0]:http://10.111.1.112:8773/services/Storage
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal disabled serviceInfos type=cluster name=CC_110 partition=PARTI00 urisLen=1
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal disabled serviceInfos    uri[0]:http://10.111.1.110:8774/axis2/services/EucalyptusCC
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal disabled serviceInfos type=walrus name=WS_112 partition=walrus urisLen=1
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | internal disabled serviceInfos    uri[0]:http://10.111.1.112:8773/services/Walrus
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | external services         uri[0]: node PARTI00 10.111.5.31 ENABLED http://10.111.5.31:8775/axis2/services/EucalyptusNC
2013-06-11 08:52:10 DEBUG 000018029 doDescribeServices       | done
2013-06-11 08:52:11 DEBUG 000018028 doMigrateInstances       | calling destination node[s] asynchronously to prepare for 1 incoming migration[s]; using pid 26459
013-06-11 08:52:11 DEBUG 000026459 doMigrateInstances       | about to ncClientCall destination node[s] with nc_instances (prepare 1) [creds='qrPdfN1Hgri5QlHb']
2013-06-11 08:52:11 DEBUG 000026459 doMigrateInstances       | [i-3624467F] about to ncClientCall destination node '10.111.5.31' with nc_instances (prepare 1) [creds='qrPdfN1Hgri5QlHb']
2013-06-11 08:52:11 DEBUG 000026460 ncMigrateInstancesStub   | marshalling 1 instance(s) [0].id=i-3624467F with action prepare
```

## Gotchas
1. While migrating many EBS instances, after migration, sometimes some of the instances become inaccessible for a few moment, but eventually it becomes available.
2. In multi-cluster configuration, migrations using --source broken for all clusters but first registered - [EUCA-6400](https://eucalyptus.atlassian.net/browse/EUCA-6400)