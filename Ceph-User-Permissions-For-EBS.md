Eucalyptus Storage Controller (SC) and Node Controller (NC) interact with Ceph cluster and require Ceph user credentials to authenticate. SC and NC can be assigned different or same Ceph user credentials.The following minimum set of permissions are recommended for Ceph users assigned to Eucalyptus hosts

SC ceph user privileges (superset of NC Ceph user's privileges):


* Read, write, execute (rwx) access to the pools used for storing EBS volumes and snapshots
* Execute (x)access to all pools (Ceph users must have execute permissions to use Ceph’s administrative commands such asunprotecting snapshots)
* Read (r) access to all monitors

NC ceph user privileges(subset of SC Ceph user's privileges):


* Read, write, execute (rwx) access to the pools used for storing EBS volumes only
* Read (r) access to all monitors

Eucalyptus recommends using different Ceph users for SC and NC as this allows fine grain control over privileges, i.e. restrict the privileges granted to Ceph user on NC relative to the privileges granted to the Ceph user on SC.Refer to the[Ceph user management](http://ceph.com/docs/giant/rados/operations/user-management/#background)for more details


### How to create a Ceph user with the required permissions
Following steps walk you through adding a new Ceph user. For more details refer to Ceph documentation in[add user section](http://ceph.com/docs/giant/rados/operations/user-management/#add-a-user)


1. To add a Ceph user, login to any of the Ceph monitor systems.Code snippet below shows how to create 'eucalyptus' user. The ' -o' option writes the output in keyring format to a file.


```
ceph auth get-or-create client.eucalyptus mon 'allow r' osd 'allow rwx pool=rbd, allow rwx pool=qavolumes, allow rwx pool=qasnapshots, allow x' -o ceph.client.eucalyptus.keyring
```

1. List the permissions and verify that they were indeed set correctly


```
ceph auth list

installed auth entries:
...
client.eucalyptus
        key: <not-shown-here>
        caps: [mon] allow r
        caps: [osd] allow rwx pool=rbd, allow rwx pool=qavolumes, allow rwx pool=qasnapshots, allow x
```



### How to modify permissions for an existing Ceph user
Following steps walk you through modifying permissions for an existing Ceph user. For more details refer to Ceph documentation in[modifying user permissions section](http://ceph.com/docs/giant/rados/operations/user-management/#modify-user-capabilities)


1. To view/change the permissions for an existing Ceph user, login to any of the Ceph monitor systems. Code snippet below uses 'eucalyptus' user as an example. To view the permissions execute


```
ceph auth list

installed auth entries:
...
client.eucalyptus
        key: <not-shown-here>
        caps: [osd] allow *

```

1. To allow 'eucalyptus' user to use rbd pool execute


```
ceph auth caps client.eucalyptus mon 'allow r' osd 'allow rwx pool=rbd, allow x' 
```
The above command translates to - assign read access to all monitors, assign read, write and execute privelegs to rbd pool for the eucalyptus user.


1. List the permissions and verify that they were indeed set correctly


```
ceph auth list
 
installed auth entries:
...
client.eucalyptus
        key: <not-shown-here>
        caps: [mon] allow r
        caps: [osd] allow rwx pool=rbd, allow x
```



### How to generate the keyring file for an existing Ceph user
If you don't have the keyring file required by Ceph clients such as SC and NCs, use the below command to generate it for an existing user. The ' -o' option writes the output to a file in keyring format


```
ceph auth get-or-create client.eucalyptus -o ceph.client.eucalyptus.keyring
```


*****

[[category.storage]] 
[[category.confluence]] 
[[category.ceph]] 
[[category.ebs]]
