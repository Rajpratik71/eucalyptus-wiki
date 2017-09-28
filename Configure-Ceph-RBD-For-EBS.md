Eucalyptus 4.1 will feature tech preview of Ceph-RBD integration as an EBS backend (only). This integration will be referred to as 'ceph-rbd provider'. Use this guide to configure Eucalyptus Storage Controller (SC) and Node Controllers (NC) to use Ceph for everything EBS. Ensure that all prerequisites are satisfied first.


## Prerequisites

* Ceph cluster is up and running
* Ceph user credentials assigned to Eucalyptus SC and NCs are[[configured|Ceph-User-Permissions-For-EBS]]with appropriate permissions (Different user credentials can be used for the SCs and NCs, below sections will explain how to configure them in Eucalyptus)


* [[NC package installation|NC-Package-Install-Guide-For-Ceph-EBS-Backend]] requirement is satisfied


* [[SC package installation|SC-Package-Install-Guide-For-Ceph-EBS-Backend]] requirement is satisfied





The following instructions will walk your through configuring Ceph as an EBS backend for a specific Eucalyptus cluster


## Storage Controller Configuration
After registering the SC, execute the below steps (as Eucalyptus admin).


1. Configure the EBS backend provider


```
euca-modify-property -p <cluster-name>.storage.blockstoragemanager=ceph-rbd
```

1. Monitor the service state for the SC and wait until the SC transitions to NOTREADY, DISABLED or ENABLED state


```
euca-describe-services
```

1. ceph-rbd provider will assume defaults for the following properties


```
euca-describe-properties -v | grep 'PARTI00.storage.ceph'

PROPERTY        PARTI00.storage.cephconfigfile  /etc/ceph/ceph.conf
DESCRIPTION     PARTI00.storage.cephconfigfile  Absolute path to Ceph configuration (ceph.conf) file. Default value is '/etc/ceph/ceph.conf'

PROPERTY        PARTI00.storage.cephkeyringfile /etc/ceph/ceph.client.eucalyptus.keyring
DESCRIPTION     PARTI00.storage.cephkeyringfile Absolute path to Ceph keyring (ceph.client.eucalyptus.keyring) file. Default value is '/etc/ceph/ceph.client.eucalyptus.keyring'

PROPERTY        PARTI00.storage.cephsnapshotpools       rbd
DESCRIPTION     PARTI00.storage.cephsnapshotpools       Ceph storage pool(s) made available to Eucalyptus for EBS snapshots. Use a comma separated list for configuring multiple pools. Default value is 'rbd'

PROPERTY        PARTI00.storage.cephuser        eucalyptus
DESCRIPTION     PARTI00.storage.cephuser        Ceph username employed by Eucalyptus operations. Default value is 'eucalyptus'

PROPERTY        PARTI00.storage.cephvolumepools rbd
DESCRIPTION     PARTI00.storage.cephvolumepools Ceph storage pool(s) made available to Eucalyptus for EBS volumes. Use a comma separated list for configuring multiple pools. Default value is 'rbd'

```
Skip the following steps if the default values work for your cloud


1. <cluster-name>.storage.cephuseris the Ceph username employed by Eucalyptus. Default value is'eucalyptus', change it by executing


```
euca-modify-property -p <cluster-name>.storage.cephuser=qauser
```

1. <cluster-name>.storage.cephkeyringfileis the absolute path to keyring file containing the key for the 'eucalyptus' user. Default value is'/etc/ceph/ceph.client.eucalyptus.keyring', change it by executing


```
euca-modify-property -p <cluster-name>.storage.cephkeyringfile='/etc/ceph/ceph.client.qauser.keyring'
```
Ifcephuserwas modified, ensure thatcephkeyringfileis also updated with the location to the keyring for the specificcephuser


1. <cluster-name>.storage.cephconfigfile is the absolute path to ceph.conf file. Default value is '/etc/ceph/ceph.conf', change it by executing


```
euca-modify-property -p <cluster-name>.storage.cephconfigfile=<path-to-ceph.conf>
```

1. <cluster-name>.storage.cephvolumepoolsis comma separated list of Ceph pools assigned to Eucalyptus for managing EBS volumes. Default value is'rbd', change it by executing


```
euca-modify-property -p <cluster-name>.storage.cephvolumepools=rbd,qavolumes
```

1. <cluster-name>.storage.cephsnapshotpoolsis comma separated list of Ceph pools assigned to Eucalyptus for managing EBS snapshots . Default value is'rbd', change it by executing


```
euca-modify-property -p <cluster-name>.storage.cephsnapshotpools=qasnapshots
```
Shared Ceph cluster currently offers 3 pools for data storage - rbd, qavolumes and qasnapshots.One of the users on the shared cluster, qauser, is configured with privileges for operating on these 3 pools




## Node Controller Configuration
Every NC will assume the following defaults

solidCEPH_USER_NAME="eucalyptus"CEPH_KEYRING_PATH="/etc/ceph/ceph.client.eucalyptus.keyring"CEPH_CONFIG_PATH="/etc/ceph/ceph.conf"

To override the above defaults, add/edit the following properties to$EUCALYPTUS/etc/eucalyptus/eucalyptus.conffile on the specific NC

solidCEPH_USER_NAME="<ceph-username-for-use-by-this-NC>"CEPH_KEYRING_PATH="<path-to-keyring-file-for-ceph-username>"CEPH_CONFIG_PATH="<path-to-ceph.conf-file>"

NC picks up these changes dynamically (restart of the eucalyptus-nc service is not necessary)


## Related articles
[[Ceph User Permissions|Ceph-User-Permissions-For-EBS]]

[[NC Package Install Guide|NC-Package-Install-Guide-For-Ceph-EBS-Backend]]

[[SC Package Install Guide|SC-Package-Install-Guide-For-Ceph-EBS-Backend]]


*****

[[category.storage]] 
[[category.confluence]] 
[[category.ceph]] 
[[category.ebs]]
