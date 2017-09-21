Install the proper udev rules and scripts so that the storage controller will have sufficient permissions for accessing attached snapshots for upload to object storage during the snapshot operation.

This is useful for source-builds and replicates the steps taken by the packaging scripts. See:[https://github.com/eucalyptus/eucalyptus-rpmspec/blob/master/eucalyptus.spec](https://github.com/eucalyptus/eucalyptus-rpmspec/blob/master/eucalyptus.spec)


## Step-by-step guide
Assuming the source has been cloned from github into $EUCA_SRC


1. Run regular build & install process for source


    1.  _cd $EUCA_SRC_ 
    1.  _./configure --prefix=$EUCALYPTUS_ 
    1.  _make_ 
    1.  _make install_ 

    
1.  _sudo cp $EUCALYPTUS/usr/share/eucalyptus/udev/55-openiscsi.rules /etc/udev/rules.d/_ 


1.  _sudo cp $EUCALYPTUS/usr/share/eucalyptus/udev/12-dm-permissions.rule /etc/udev/rules.d/_ 
1.  _sudo cp $EUCALYPTUS/usr/share/eucalpytus/udev/iscsidev.sh /etc/udev/scripts/_ 

If you are developing a new SAN adapter, you will need to add the SAN's common IQN prefix to the _iscsidev.sh_ script found in _$EUCA_SRC/clc/modules/block-storage-common/udev/._  Example diff:


```
diff --git a/clc/modules/block-storage-common/udev/iscsidev.sh b/clc/modules/block-storage-common/udev/iscsidev.sh
index f5905b7..2edd3db 100755
--- a/clc/modules/block-storage-common/udev/iscsidev.sh
+++ b/clc/modules/block-storage-common/udev/iscsidev.sh
@@ -44,7 +44,7 @@ if [ -z "${target_name}" ]; then
 fi
 
 check_target_name=${target_name%%:*}
-if [ $check_target_name = "iqn.2001-05.com.equallogic" ] || [ $check_target_name = "iqn.1992-08.com.netapp" ] || [ $check_target_name = "iqn.1992-04.com.emc" ]; then
+if [ $check_target_name = "iqn.2001-05.com.equallogic" ] || [ $check_target_name = "iqn.1992-08.com.netapp" ] || [ $check_target_name = "iqn.1992-04.com.emc" ] || [$check_target_name = "<MYNEWSAN-IQN>"] ; then
     target_name=`echo "${target_name#${check_target_name}:}"`
 else
    exit 1
```

## Related articles
Related articles appear here based on the labels you select. Click to edit the macro and add or change labels.

falsetruemodified5udev sc blockstoragefalsepage

true

|  | 
|  --- | 
|  | 



*****

[[category.storage]] 
[[category.confluence]] 
