The goal of this wiki entry is to document best practices for creating images, and using instances in a secure manner on hybrid (e.g. AWS and Eucalyptus) and on-premise cloud environments.  Additional information can be found in the [Instance Best Practices](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-overview.html) documentation provided by Amazon Web Services. 

### Image Creation

Security starts at the foundation.  Since all instances are based off of images, implementing any type of security is helpful in securing instances.  Below are a list of things that can be done to add additional security during image creation:

* Enabled [PAM](http://www.linux-pam.org/) to help control authentication.
* Enable [Selinux](http://selinuxproject.org/page/Main_Page) or [Apparmor](http://wiki.apparmor.net/index.php/Main_Page).  Additional resources are below:
  * [Redhat Enterprise Linux 6 Security-Enhanced Linux](https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security-Enhanced_Linux/)
  * [CentOS 5 Deployment Guide - Security and Selinux](http://www.centos.org/docs/5/html/Deployment_Guide-en-US/selg-overview.html)
  * [Documentation - Apparmor](http://wiki.apparmor.net/index.php/Documentation)
  * [AppArmor - Ubuntu Wiki](https://wiki.ubuntu.com/AppArmor)
  * [openSUSE 12.3 - Introducing AppArmor](http://doc.opensuse.org/documentation/html/openSUSE/opensuse-security/cha.apparmor.intro.html)
* Turn off password-based authentication.  This can be done - along with enabling PAM authentication for ssh - by enabling the following options in /etc/ssh/sshd_config:
```
PasswordAuthentication no
UsePAM yes
```
* Encourage non-root access by providing a user that has the ability to have root privileges using [sudo](http://www.sudo.ws/). For example, Amazon AMI Linux images have the "ec2-user" user and Ubuntu Cloud images have the "ubuntu" user.
* Lock down passwords of the root and non-root default users on the image.  This can be done by executing "passwd -l <user-id>".  _NOTE: Some Linux distributions will lock the account when using this command.  If this is the case, please leverage PAM and/or create random password for the accounts.  A clever way of accomplishing this is by using dd:_ 
```
dd if=/dev/urandom count=50|md5sum|passwd --stdin <user-id>
```
* Always delete the shell history before bundling. If you attempt more than one bundle upload in the same image, the shell history contains your secret access key.
* Bundling a running instance requires your private key and X.509 certificate. Put these and other credentials in a location that is not bundled (e.g. when using [euca-bundle-vol](http://www.eucalyptus.com/docs/euca2ools/3.0/euca2ools-guide/euca-bundle-vol.html#euca-bundle-vol), pass the folder location where the certificates are stored as part of the values for the -e option).  AWS provides more in-depth information here regarding [what things should be considered when creating a shared AMI (EMI)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AESDG-chapter-sharingamis.html).
* (Optional) By default, all images registered have private launch permissions.  Use [euca-modify-image-attribute](http://www.eucalyptus.com/docs/euca2ools/3.0/euca2ools-guide/euca-modify-image-attribute.html#euca-modify-image-attribute) to limit the accounts that can access the image.
* (Optional) Install [cloud-init](https://cloudinit.readthedocs.org/en/latest/) in the image to help control root and non-root access. If cloud-init isn't available, using a [custom /etc/rc.local](https://github.com/eucalyptus/Eucalyptus-Scripts/blob/master/rc-ec2user.local) script can be used.
* (Optional) Zero-out any unused space on the image.  This can be accomplished with tools like [zerofree](http://manpages.ubuntu.com/manpages/precise/man8/zerofree.8.html).
* (Optional) Edit /etc/rc.local to clear out the swap every time the instance is booted.  This can be done by the following command: 
```
sync && /sbin/sysctl vm.drop_caches=3 && swapoff -a && swapon -a
```
* (Optional) Edit /etc/rc.local to zero-out the ephemeral disk (this is usually /dev/vda2) on boot. This will add some additional time to the instance being fully ready to be used.  _NOTE: After zeroing out the ephemeral space, the ephemeral store will need to be formatted with a filesystem (e.g. mkfs.ext3 /dev/vda2)_.

### Instance Security

After locking down the image by using the steps above, additional steps can be done to further secure the instance.  Below are some examples:

* Restrict access by only allowing trusted hosts or networks to access ports on your instance. Controlling access can be accomplished by using [euca-authorize](http://www.eucalyptus.com/docs/euca2ools/3.0/euca2ools-guide/euca-authorize.html#euca-authorize) and [euca-revoke](http://www.eucalyptus.com/docs/euca2ools/3.0/euca2ools-guide/euca-revoke.html#euca-revoke).  Review the rules in your security groups regularly, and ensure that you apply the principle of least privilege—only open up permissions that you require. You can also create different security groups to deal with instances that have different security requirements. Consider creating a [bastion](http://www.thefreedictionary.com/bastion) security group that allows external logins and keep the remainder of your instances in a group that does not allow external logins.
* Consider using Eucalyptus IAM policies to control access to resources.  For example, providing a policy that restricts access coming from a source IP and/or specific Eucalyptus IAM account can be leveraged by using certain IAM EC2-specific keys.  For more information, please check out the [IAM Policy Elements Reference](http://docs.aws.amazon.com/IAM/latest/UserGuide/AccessPolicyLanguage_ElementDescriptions.html#AvailableKeys).
* When using a highly-available instance with [Eucalyptus AutoScaling](http://www.eucalyptus.com/docs/eucalyptus/3.3/user-guide/autoscaling_overview.html#autoscaling_overview), use an AutoScaling group of 1.  Also refer to [AWS AutoScaling documentation](http://docs.aws.amazon.com/AutoScaling/latest/DeveloperGuide/WorkingWithASG.html) as well for additional information.

As mentioned before, this is an effort to help provide a centralized document for instance security best practices.  Any feedback/suggestions/ideas are greatly appreciated. 

*****

[[category.images]]