# General
* A problem has been observed where cloud-init does not run on the load balancer servo vm.  This results in haproxy not being initialized and the lb vm will not accept and route traffic to registered instances.  To fix this particulkar issue log into the load balancer vm and reboot

* Due to changes in ELB bootstrapper, The AWS_ELBURL will not be configured in eucarc. You must install the ELB VM image and set loadbalancing.loadbalancer_emi property then manually edit eucarc to add the ELB service URL. 
example. 
> export AWS_ELB_URL=http://\<your clc ip\>:8773/services/LoadBalancing
 
* There is a problem identified with the devel load balancer image.  Probably should use the non-devel version at first

* Eustore image install broken right now, have to manually install load balancer image to your cloud. You have to manually bundle, upload and register the kernel, ramdisk and image. Something like this:

> mkdir -p /disk1/storage/elb_image

> cd /disk1/storage/elb_image


> wget http://jenkins.release.eucalyptus-systems.com/job/load-balancer-image/lastSuccessfulBuild/artifact/eucalyptus-load-balancer-image-devel.tgz


> tar xvfz eucalyptus-load-balancer-image-devel.tgz


> cd eucalyptus-load-balancer-image-devel/


> euca-bundle-image -i vmlinuz-2.6.32-358.2.1.el6.x86_64  --kernel true -d .

> euca-upload-bundle -b elb_image_bucket -m vmlinuz-2.6.32-358.2.1.el6.x86_64.manifest.xml

> euca-register -n elb-image-kernel elb_image_bucket/vmlinuz-2.6.32-358.2.1.el6.x86_64.manifest.xml


> euca-bundle-image -i initramfs-2.6.32-358.2.1.el6.x86_64.img  --ramdisk true -d .

> euca-upload-bundle -b elb_image_bucket -m initramfs-2.6.32-358.2.1.el6.x86_64.img.manifest.xml

> euca-register -n elb-image-ramdisk elb_image_bucket/initramfs-2.6.32-358.2.1.el6.x86_64.img.manifest.xml


> \# change load balancer image to the one you are using

> euca-bundle-image -i eucalyptus-load-balancer-image-devel-1.0.0-48.130411git1a55ab3.img  --ramdisk <ramdisk 
ID from above> --kernel <kernel id from above> -d .

> euca-upload-bundle -b elb_image_bucket -m eucalyptus-load-balancer-image-devel-1.0.0-48.130411git1a55ab3.img.manifest.xml

> euca-register -n elb-image elb_image_bucket/eucalyptus-load-balancer-image-devel-1.0.0-48.130411git1a55ab3.img.manifest.xml

* After installing the lb image you have to set system property of the load balancer image ID:
euca-modify-property -p loadbalancing.loadbalancer_emi=<emi id of lb image>

* Jenkins load balancer build:
http://jenkins.release.eucalyptus-systems.com/job/load-balancer-image/

* Github version (the read me here is helpful):
https://github.com/eucalyptus/load-balancer-image 

# Load Balancer VM
 * What processes should we be looking for on the VM?
    - you should look for 1) python load-balancer-servo, 2) haproxy
 
 * Where are the logs? (servo/haproxy)
    - /var/log/load-balancer-servo/servo.log
 
 * Where is HA proxy's configuration file?
    - /var/lib/load-balancer-servo/servo-haproxy.conf 

 * Is there any way to poke servo for status/describe type operations?
    - no, but you can check if it listens on using curl localhost:port 

 * Do we need to explicitly add the listener ports to the loadbalancer VM security group?
    - no elb code takes care of that 

*****
[[category.faqs]]