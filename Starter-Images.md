Find below a list of various images for Linux distributions. We also have the [[Kexec-images]] page, that should be merged with this one.

## Listing of Starter images

<table width="70%">
    <tr>
       <th>URL to Image</th>
       <th>Title</th>
       <th>Owner</th>
       <th>Latest Test</th>
       <th>Notes</th>
    </tr>
    <tr>
       <td><a href="http://emis.eucalyptus.com/starter-emis/euca-centos6.4-ec2user-2013.07.12-x86_64.tgz">Download</a></td>
       <td>CentOS 6.4 x86_64</td>
       <td><a href="mailto:harold.spencer.jr@eucalyptus.com">Harold Spencer, Jr.</a></td>
       <td><a href="https://github.com/eucalyptus/image-verification-results/blob/master/centos6.4-ec2user-testresults.txt">Test Results</a><br>(Eucalyptus 3.3.0)</td>
       <td>Hypervisor-Specific Kernel; 2.6.32-358.11.1.el6.x86_64 kernel version; ec2-user enabled, sudo rights; Selinux enabled; euca2ools 3.0.0 installed.</td>
    </tr>
    <tr>
       <td><a href="http://cloud-images.ubuntu.com/raring/current/raring-server-cloudimg-amd64.tar.gz">Download from Ubuntu</a></td>
       <td>Ubuntu 13.04 x86_64 - Official Ubuntu Cloud Image</td>
       <td><a href="mailto:harold.spencer.jr@eucalyptus.com">Harold Spencer, Jr.</a></td>
       <td><a href="https://github.com/eucalyptus/image-verification-results/blob/master/ubuntu13.04-eutester-testresults.txt">Test Results</a><br>(Eucalyptus 3.3.0)</td>
       <td>Latest official Ubuntu image from cloud-images.ubuntu.com (if we use this link, it'll always pull the latest version of the image as they get updated frequently)</td>
    </tr>
    <tr>
       <td><a href="http://cloud-images.ubuntu.com/precise/current/precise-server-cloudimg-amd64.tar.gz">Download from Ubuntu</a></td>
       <td>Ubuntu 12.04 LTS x86_64 - Official Ubuntu Cloud Image</td>
       <td><a href="mailto:harold.spencer.jr@eucalyptus.com">Harold Spencer, Jr.</a></td>
       <td><a href="https://github.com/eucalyptus/image-verification-results/blob/master/ubuntu12.04-eutester-testresults.txt">Test Results</a><br>(Eucalyptus 3.3.0)</td>
       <td>Latest official Ubuntu image from cloud-images.ubuntu.com (if we use this link, it'll always pull the latest version of the image as they get updated frequently)</td>
    </tr>
    <tr>
       <td><a href="http://cloud-images.ubuntu.com/lucid/current/lucid-server-cloudimg-amd64.tar.gz">Download from Ubuntu</a></td>
       <td>Ubuntu 10.04 LTS x86_64 - Official Ubuntu Cloud Image</td>
       <td><a href="mailto:harold.spencer.jr@eucalyptus.com">Harold Spencer, Jr.</a></td>
       <td><a href="https://github.com/eucalyptus/image-verification-results/blob/master/ubuntu10.04-eutester-testresults.txt">Test Results</a><br>(Eucalyptus 3.3.0)</td>
       <td>Latest official Ubuntu image from cloud-images.ubuntu.com (if we use this link, it'll always pull the latest version of the image as they get updated frequently)</td>
    </tr>
    <tr>
       <td><a href="http://emis.eucalyptus.com/starter-emis/euca-fedora17-ec2user-2013.07.04-x86_64.tgz">Download</a></td>
       <td>Fedora 17 x86_64</td>
       <td><a href="mailto:harold.spencer.jr@eucalyptus.com">Harold Spencer, Jr.</a></td>
       <td><a href="https://github.com/eucalyptus/image-verification-results/blob/master/fedora17-ec2user-testresults.txt">Test Results</a><br>(Eucalyptus 3.3.0)</td>
       <td>Hypervisor-Specific Kernel, 3.9.8-100.fc17 kernel version, 1.6GB root; cloud-init enabled, ec2-user enabled, sudo rights; Selinux Enabled; euca2ools 2.1.3 installed. It works w/ kexec kernel ( default kernel is provided as part of tar )</td>
    </tr>
    <tr>
       <td><a href="http://emis.eucalyptus.com/starter-emis/euca-fedora18-ec2user-2013.07.04-x86_64.tgz">Download</a></td>
       <td>Fedora 18 x86_64</td>
       <td><a href="mailto:harold.spencer.jr@eucalyptus.com">Harold Spencer, Jr.</a></td>
       <td><a href="https://github.com/eucalyptus/image-verification-results/blob/master/fedora18-ec2user-testresults.txt">Test Results</a><br>(Eucalyptus 3.3.0)</td>
       <td>Hypervisor-Specific Kernel, 3.9.6-200.fc18 kernel version, 1.7GB root; cloud-init enabled, ec2-user enabled, sudo rights; Selinux Enabled; euca2ools 2.1.3 installed. It works w/ kexec kernel ( default kernel is provided as part of tar )</td>
    </tr>
    <tr>
        <td><a href="http://emis.eucalyptus.com/starter-emis/euca-debian-ec2user-2013.07.08-x86_64.tgz">Download</a></td>
       <td>Debian 7 x86_64</td>
       <td><a href="mailto:harold.spencer.jr@eucalyptus.com">Harold Spencer, Jr.</a></td>
       <td><a href="https://github.com/eucalyptus/image-verification-results/blob/master/debian7-ec2user-testresults.txt">Test Results</a><br>(Eucalyptus 3.3.0)</td>
       <td>Debian 7 1.7GB root - Hypervisor-Specific Kernel, 3.9-1-amd64 kernel version; cloud-init enabled, ec2-user enabled, sudo rights; Apparmor enabled; euca2ools 2.1.3 installed</td>
    </tr>
    <tr>
       <td><a href="https://s3-eu-west-1.amazonaws.com/opensuse-images/opensuse-12.2-x86_64-emi.tar.gz">Download</a></td>
       <td>OpenSUSE 12.2 x86_64</td>
       <td><a href="mailto:lester@eucalyptus.com">Lester Wade</a></td>
       <td><a href="https://github.com/eucalyptus/image-verification-results/blob/master/opensuse12.2-eutester-testresults.txt">Test Results</a><br>(Eucalyptus 3.0.x - 3.1.x)</td>
       <td>KVM image. SUSE Firewall off.  Root disk of 2.5G.  Root user enabled.  Working with kexec kernel and ramdisk. OpenSUSE minimal base package set.</td>
    </tr>
    <tr>
        <td><a href="http://emis.eucalyptus.com/starter-emis/euca-centos5-2013.06.18-x86_64-ec2user.tgz">Download</a></td>
       <td>CentOS 5.9 x86_64</td>
       <td><a href="mailto:harold.spencer.jr@eucalyptus.com">Harold Spencer, Jr.</a></td>
       <td><a href="https://github.com/eucalyptus/image-verification-results/blob/updated-centos5-images-tests/centos5-ec2user-eustester-testresults.txt">Test Results</a><br>(Eucalyptus 3.3 - 3.4 devel)</td>
       <td>CentOS 5.9 1.4GB root, Hypervisor-Specific Kernel; 2.6.18-348.6.1.el5 kernel version; ec2-user enabled, sudo rights; Selinux enabled.</td>
    </tr>
    <tr>
         <td><a href="http://emis.eucalyptus.com/starter-emis/euca-centos5-2013.06.18-x86_64-root.tgz">Download</a></td>
       <td>CentOS 5.9 x86_64</td>
       <td><a href="mailto:harold.spencer.jr@eucalyptus.com">Harold Spencer, Jr.</a></td>
       <td><a href="https://github.com/eucalyptus/image-verification-results/blob/updated-centos5-images-tests/centos5-root-user-eustester-testresults.txt">Test Results</a><br>(Eucalyptus 3.3 - 3.4 devel)</td>
       <td>CentOS 5.9 1.4GB root, Hypervisor-Specific Kernel; 2.6.18-348.6.1.el5 kernel version; root user enabled; Selinux enabled.</td>
    </tr>
</table>

_Note: The CentOS 5 images contain euca2ools 2.1.3 (Limbo) due to the fact that the openssl required for euca2ools 3.0 isn't available on any official CentOS 5 repository._

## Uploading Images

Once you've retrieved the starter image you want, you must upload that image to your Eucalyptus cloud.  The basic steps that are required:

* Upload a kernel and ramdisk. A good option here is to upload the [[kexec-loader]] kernel and ramdisk.
* Upload the image, associating with the kernel and ramdisk in the process. Documentation is [here](http://www.eucalyptus.com/docs/latest/ug/associating_ekieri.html#associating_ekieri).


## Getting Help

If you have any questions about images, please join the [Eucalyptus community mailing list](http://lists.eucalyptus.com/cgi-bin/mailman/listinfo/community) and ask there, or find us on [IRC](http://webchat.freenode.net/?channels=eucalyptus).


*****

[[category.images]]