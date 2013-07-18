This page is to track the ideas for improving [eustore](http://www.eucalyptus.com/docs/euca2ools/3.0/euca2ools-guide/eustore.html#eustore). The motivation behind this wiki page is the fact that there are improvements to the image management story in Eucalyptus 3.4.  We want to guarantee that eustore meets those improvements and offers more features to end-users.  This page will raised to the [Image Scrum Team for Eucalyptus 3.4](https://groups.google.com/a/eucalyptus.com/forum/#!forum/team-image) to see if it falls inline with the 3.4 improvements.  **Goal is to have these improvements be community contributed**.

## Current Design 

The current design of eustore has two parts:  server and client.

### Server

The server side of eustore is driven by an nginx web server, and a unique catalog file (JSON format) that contains metadata information regarding the images (stored as part of a tar-gzipped file; explained more below) that are accessible by the eustore client.  For additional information, please see the [Eustore Catalog Server project on Github](https://github.com/eucalyptus/eustore-catalog-server).

### Client

The client side of [eustore](https://github.com/eucalyptus/euca2ools/tree/master/euca2ools/commands/eustore) is part of the [euca2ools project on Github](https://github.com/eucalyptus/euca2ools). It currently supports files that are in a tar-gzipped format.  The contents of the tar-gzipped format file must be as follows:

* image
* <hypervisor>-kernel (where hypervisor is "kvm", "xen", "vmware")
   * vmlinuz file
   * initrd file

_Note: there can be multiple <hypervisor>-kernel directories_

These tar-gzipped files can be bundled, uploaded and registered to Walrus in the following ways:

* [eustore server catalog](https://github.com/eucalyptus/eustore-catalog-server)
* the -t option with [eustore-install-image](http://www.eucalyptus.com/docs/euca2ools/3.0/euca2ools-guide/eustore-install-image.html#eustore-install-image)

## Brainstorm List for Improvements

Below are a list of improvements mentioned in the [Eustore Improvement meeting held in #eucalyptus-devel on irc.freenode.net](http://meetbot.eucalyptus.com/meeting-logs/eucalyptus-devel/2013-07-18/eustore_improvements.2013-07-18-15.30.log.html).  

* Improve "packaging" format to be more flexible and robust (i.e. support files other than tar.gz).
* Server-side metadata information for images
* Kernel/Ramdisk dependency resolution (i.e. if the kernel/ramdisk for an image already exists on Walrus, don't bundle, upload and register)
* Tagging and/or versioning support for images 
* EMI/AMI support for images in the following formats:  .tar.gz, .tgz, .bz2, .raw, .qcow2
* Customizing EMIs through eustore command
* Support for bundling, uploading and register Windows images via eustore (using -t option)
* Image verification check through eustore command
* Add support for bundling, uploading, registering images from Ubuntu Cloud Images using [JSON file]( http://cloud-images.ubuntu.com/releases/streams/v1/com.ubuntu.cloud:released:download.json)
* Review [OpenStack Glance](http://docs.openstack.org/developer/glance/) integration (maybe replace eustore)?
* Look at using [virt-sysprep](http://libguestfs.org/virt-sysprep.1.html) for [sanity check of images](https://access.redhat.com/site/documentation/en-US/Red_Hat_OpenStack/2/html/Getting_Started_Guide/ch09s02.html)
* Integration of [libguestfs-tools](http://libguestfs.org/)

Any feedback regarding this page, please send to either:

* [Eucalyptus Users Group](mailto:euca-users@eucalyptus.com)
* [Eucalyptus Image Team](mailto:team-image@eucalyptus.com)