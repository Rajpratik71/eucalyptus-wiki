**_NOTE: the content on this page is from the old projects.eucalyptus.com site and might be out of date._**

h1. Images

The _starter_ images, which you may find at http://open.eucalyptus.com/wiki/EucalyptusUserImageCreatorGuide, are based on popular distro. To made them more _cloud_ friendly they have few twists:

* they have a very small root file system (so far 1.3 and 4.5 GB): this allows them to boot very fast, with the downside of having a small root, hence they rely heavily on EBS and ephemeral;
* they are based on the original distro, but the package list has been trimmed from the original default install;
* packages have been added from the original default install;
* an rc.local has been added to allow the instance to interact with the "meta-data":http://open.eucalyptus.com/participate/wiki/accessing-instance-metadata service;
* a /etc/image-id to keep track of the version of the base image used for the instance.

Most of the files, scripts and stuff needed for the images are kept at https://github.com/eucalyptus/Eucalyptus-Scripts. The git repo is mirrored here on projects if you prefer to review it here.

h1. rc.local

The source:rc.local we use is a very simple one, and it mainly interact with the meta-data service. Right now it grabs the ssh keys (to allow root to login), mount ephemeral0 to the right place, and try to grab the user-data (if there is any) and execute it if it is a script.

The latter allow the images to be easily customized at boot time: check out project:recipes for some scripts to use with this rc.local.

h1. EMI id

We keep an image-id on each image at /etc/image-id to allow for easy identification of the image used for the instance. We keep a list of the current image-ids in the well known repo at source:EMI-IDs. Check the git log as a changelog on the images. The fields are

* image_name: brief description of the EMI for example "euca-debian-emi"
* image_version: a string defining the version of the EMI for example "2011.08.15"
* image_arch: the architecture of the installed binaries, for example "x86_64"
* image_file: the file name for the image, for example "euca-debian-2011.08.15-x86_64.img"
* image_stamp: a unique ID for the specific image, for example "082d-d3a4". Currently it is generated using 
```
date +%s|md5sum|sed '1,$ s/\(....\)\(....\).*/\1-\2/'
```
* image_date: currently generated using 
```date "+%Y%m%d%H%M%S"```
 for example "20110815122021"
* recipe_name: this will tell which distro the EMI is based off for example "debian-based"

Here are some image-id files for images we're creating [[image-ids]]
Look here for descriptions of the images we've created [[image-info]]


h1. Root file systems on different devices

Depending on the hypervisor and guestOS configuration, block devices may show up as xvd* or vd* or sd*. In "ticket 97:https://projects.eucalyptus.com/redmine/issues/97" there is the discussion. The solution is to modify udev:

<pre>
cat /etc/udev/rules.d/euca-udev.rules
KERNEL=="xvd*", PROGRAM="/sbin/euca-udev %k", SYMLINK+="%c"
</pre>

The program "/sbin/euca-udev" is as follows:

<pre>
#!/bin/bash
if [ "$#" -ne 1 ] ; then
   echo "$0 <device>" >&2
   exit 1
else
   if echo "$1"|grep -qE 'xvd[a-z][0-9]?' ; then
      echo "$1" | sed -e 's/xvd/sd/'
   else
      echo "$1"
   fi
fi
</pre>


h1. single kernel

We have a separate project to tackle the single-kernel issue at project:kernel.

h1. zerofree

Before tarring the images up, we run _zerofree_ on them (on a Debian system there is a zerofree package) which will fill all the unused sectors with zeros (it works only for ext2/3 file systems). This allows for the images to be more easily compressed hence making the tar file much smaller.

h1. aelastic PPA

We did add the "aelastic":aelastic.com PPA to the list of repos to pull packages from (when applicable) to allow for ec2-constiste-snapshot to be present on the image. So on debian-based distro we added the following key: <pre> apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BE09C571</pre>

h1. partner image requirements

[[partnerimages]]

h1. tips and tricks

A quick way to modify the images is to mount them on the loop device and them bind-mound some of the needed kernel  file systems

<pre>
mount -o loop <emi file name> /mnt
mount -o bind /proc /mnt/proc
mount -o bind /sys /mnt/sys
mount -o bind /dev /mnt/dev
mount -o bind /dev/pts /dev/pts
</pre>

You can then chroot and modify the image (for example install or remove packages). Before exiting, you should clean up after yourself, for example

<pre>
yum clean all
apt-get clean
</pre>

you may want to remove the history of your commands (after you end the chroot session)

<pre>
rm /mnt/root/.bash_history
</pre>

h1. Extend your root FS with the ephemeral disk

[[Inject_ephemeral_disk_into_root_disk]]

h1. IRC meeting summaries and full logs

* "2011 Dec 9":https://meetbot.eucalyptus.com/meeting-logs/eucalyptus-meeting/2011-12-09/images.2011-12-09-19.02.html
* "2011 Dec 16":https://meetbot.eucalyptus.com/meeting-logs/eucalyptus-meeting/2011-12-16/images.2011-12-16-19.03.html
* "2012 Jan 6":https://meetbot.eucalyptus.com/meeting-logs/eucalyptus-meeting/2012-01-06/images.2012-01-06-19.06.html
* "2012 Jan 20":https://meetbot.eucalyptus.com/meeting-logs/eucalyptus-meeting/2012-01-20/images.2012-01-20-22.03.html
* "2012 Jan 27":https://meetbot.eucalyptus.com/meeting-logs/eucalyptus-meeting/2012-01-27/images.2012-01-27-19.02.html
* "2012 Feb 3":https://meetbot.eucalyptus.com/meeting-logs/eucalyptus-meeting/2012-02-03/images.2012-02-03-19.01.html
* "2012 Feb 10":https://meetbot.eucalyptus.com/meeting-logs/eucalyptus-meeting/2012-02-10/images.2012-02-10-19.02.html
* "2012 Mar 2":https://meetbot.eucalyptus.com/meeting-logs/eucalyptus-meeting/2012-03-02/images.2012-03-02-18.00.html
* "2012 Mar 9":https://meetbot.eucalyptus.com/meeting-logs/eucalyptus-meeting/2012-03-09/images.2012-03-09-19.01.html
* "2012 Mar 16":https://meetbot.eucalyptus.com/meeting-logs/eucalyptus-meeting/2012-03-16/eucalyptus-meeting.2012-03-16-18.17.html
* "2012 Mar 23":https://meetbot.eucalyptus.com/meeting-logs/eucalyptus-meeting/2012-03-23/eucalyptus-meeting.2012-03-23-18.00.html