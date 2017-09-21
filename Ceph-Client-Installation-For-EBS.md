With Ceph-RBD as the EBS backend in a Eucalyptus cluster, Storage Controller (SC) and Node Controllers (NC) interact with Ceph via a client that primarily comprises of librbd and librados. In the case of SC, JNA bindings used for Ceph interaction depend on librbd and librados. In the case of NC, qemu-kvm driver for Ceph depends librbrd and librados.

Beginning 4.1.0, eucalyptus-sc and eucalyptus-nc rpm packages will have a dependency on necessary Ceph packages. This dependency will ensure that the eucalyptus install process also installs Ceph client packages on the SC and NC. You can skip the below instructions to install Ceph client if eucalyptus was installed from packages. Follow the Ceph client installation otherwise (source eucalyptus install)


## Step-by-step guide
The following instructions will walk you through installing and configuring Ceph client on SCs and NCs. This process must be repeated on every NC and SC that uses Ceph as the EBS backend.


1. Install librbd and librados packages fromepel repository. Version used for dev/test as of this wiki -0.80.5-9

DO NOT install these packages from Ceph's repositories as the operation requires installation of yum-plugin-priorities and assigning the Ceph repositories a higher priority. The yum priorities plugin may break RHEL support and cause an upgrade nightmare. (Swathi to add more a more refined message with help from Garrett)


```
yum install --enablerepo epel librbd1 librados2
```

1. The above packages don't install rbd executable. This executable, though strictly not required for Eucalyptus-Ceph interaction, provides command line interaction with Ceph. It can be very handy for debugging/checking the sanity of the setup. If you don't want to install it skip this step. If you want to install it run:


```
yum install --enablerepo epel /usr/bin/rbd
```

1. If /etc/ceph does not exist, create it. Place Ceph configuration file -ceph.confunder/etc/ceph/

ceph.conf should be readable by eucalyptus user


1. Place Ceph kerying file - ceph.client.username.keyring under /etc/ceph

ceph.client.username.keyring should be owned by eucalyptus group on SC, kvm group on NC and root user in both the cases. Permissions on the keyring file should be 640 (owner:rw, group:r)


1. If rbd executable was installed in step 2, invoke Ceph operations on the command line with


```
rbd --id username --keyring /etc/ceph/ceph.client.username.keyring <command> <options>
```



## Related articles
[[Shared Ceph Cluster|Shared-Ceph-Cluster]]



true

|  | 
|  --- | 
|  | 



*****

[[category.storage]] 
[[category.confluence]] 
