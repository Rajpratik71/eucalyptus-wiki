This task assumes that a Ceph storage cluster is available and Ceph RGW is being installed on CentOS. To install and configure Ceph itself, refer to <link here>. The following instructions are derived from Ceph and RedHat docs. Some steps were either modified or skipped as necessary for integration with Eucalyptus.


## Step-by-step guide
* [Ceph user setup](#ceph-user-setup)
* [Webserver configuration](#webserver-configuration)
* [radosgw user setup](#radosgw-user-setup)
* [radosgw: Get users list and info about existing user](#radosgw:-get-users-list-and-info-about-existing-user)
* [radosgw: Modify user e.g change max bucket](#radosgw:-modify-user-e.g-change-max-bucket)



### Ceph user setup
ceph-radosgw daemon runs on the gateway host and requires a Ceph user for authenticating with the storage cluster. Execute the following commands fromone of the monitors or the admin node from where ceph-deploy was run. Credentials with sufficient priveleges such as ceph.client.admin.keyringare required to run these commands


1. Create a new (empty) keyring file


```
# ceph-authtool --create-keyring ceph.client.radosgw.keyring
```

1. Ceph username must match the name of the ceph-radosgw instance in ceph.conf (explained in web server configuration section). If they don't match ceph-radosgw service won't start up correctly. For instance, if ceph-radosgw service is installed on host xyz, the username assigned to it should be of in the formatclient.radosgw.xyz


```
# ceph-authtool ceph.client.radosgw.keyring -n client.radosgw.xyz --gen-key
```

1. Add capabilities to the key. Capabilities are privilegesto read, write and execute over osds and monitors


```
# ceph-authtool -n client.radosgw.xyz --cap osd 'allow rwx' --cap mon 'allow rwx' ceph.client.radosgw.keyring
```

1. So far a key was generated and had some privileges assigned to it, but its still not associated with the Ceph cluster. Now add the key to the cluster


```
# ceph auth add client.radosgw.xyz -i ceph.client.radosgw.keyring
```



### Webserver configuration
radosgw requires a web server for processing HTTP requests. There are two options available here - civetweb and apache with fastcgi. civitweb is a light weight web server that ships with radosgw and does not need any further installation. For using apache and fastcgi Ceph specific apache and fastcgi packages (support for 100 continue) need to be installed and configured. Although we have not compared/evaluated performance characteristics of the different web servers, I would recommend going with civetweb due to its ease of installation and configuration

Civetweb (Recommended)Ceph version 0.94.6 from Ceph [http://download.ceph.com/rpm-hammer/el6/x86_64/](http://download.ceph.com/rpm-hammer/el6/x86_64/)


1. Installceph-radosgwfrom Ceph repositoris


```
# yum install ceph-radosgw
```
# yum info ceph-radosgw
Installed Packages
Name        : ceph-radosgw
Arch        : x86_64
Epoch       : 1
Version     : 0.94.6
Release     : 0.el6
Size        : 9.6 M
Repo        : installed
From repo   : ceph
Summary     : Rados REST gateway
URL         : http://ceph.com/
License     : LGPL-2.1 and CC-BY-SA-1.0 and GPL-2.0 and BSL-1.0 and GPL-2.0-with-autoconf-exception and BSD-3-Clause and MIT
Description : This package is an S3 HTTP REST gateway for the RADOS object store. It
            : is implemented as a FastCGI module using libfcgi, and can be used in
            : conjunction with any FastCGI capable web server.
1. Copy/etc/ceph/ceph.conffrom Ceph cluster to the host running Ceph RGW. Add the following gateway configuration to the copied ceph.conf. Replace {host-name} with the actual name of the host (output of hostname -s, not FQDN)

radosgw instance name between the \[ ] below should match the Ceph username created earlier

\[client.radosgw.xyz]
        host = {host-name}
        keyring = /etc/ceph/ceph.client.radosgw.keyring
        log file = /var/log/ceph/radosgw.log
        rgw frontends = "civetweb port=7480"
1. Start ceph-radosgw service


```
# service ceph-radosgw start
```


Apache and Fastcgi (Not Recommended)Ceph version 0.80.5 from epel [https://dl.fedoraproject.org/pub/epel/6/x86_64/repoview/ceph.html](https://dl.fedoraproject.org/pub/epel/6/x86_64/repoview/ceph.html)


1. Installhttpdfrom CentOS repository. **DO NOT** install from Ceph repo


```
# yum install httpd
```
# yum info httpd
Installed Packages
Name        : httpd
Arch        : x86_64
Version     : 2.2.15
Release     : 47.el6.centos
Size        : 2.9 M
Repo        : installed
From repo   : centos-6-x86_64-updates
Summary     : Apache HTTP Server
URL         : http://httpd.apache.org/
License     : ASL 2.0
Description : The Apache HTTP Server is a powerful, efficient, and extensible
            : web server.
1. Installmod_fastcgifrom Ceph repository. This modified version of mod_fastcgihas support for 100 continue


```
# yum install http://gitbuilder.ceph.com/mod_fastcgi-rpm-centos6-x86_64-basic/ref/master/x86_64/mod_fastcgi-2.4.7-1.ceph.el6.x86_64.rpm
```
# yum info mod_fastcgi
Installed Packages
Name        : mod_fastcgi
Arch        : x86_64
Version     : 2.4.7
Release     : 1.ceph.el6
Size        : 369 k
Repo        : installed
From repo   : /mod_fastcgi-2.4.7-1.ceph.el6.x86_64
Summary     : Apache module that enables FastCGI
URL         : http://www.fastcgi.com/
License     : GPL/Apache License
Description : mod_fastcgi is a module for the Apache web server, that enables
            : FastCGI - a standards based protocol for communicating with
            : applications that generate dynamic content for web pages.
1. Install ceph-radosgwfrom epel repository


```
# yum install ceph-radosgw
```
# yum info ceph-radosgw
Installed Packages
Name        : ceph
Arch        : x86_64
Epoch       : 1
Version     : 0.80.5
Release     : 9.el6
Size        : 42 M
Repo        : installed
From repo   : epel
Summary     : User space components of the Ceph file system
URL         : http://ceph.com/
License     : GPL-2.0
Description : Ceph is a massively scalable, open-source, distributed
            : storage system that runs on commodity hardware and delivers object,
            : block and file system storage.
1. Copy /etc/ceph/ceph.conffrom Ceph cluster to the host running Ceph RGW. Add the following gateway configuration to the copied ceph.conf. Replace {host-name} with the actual name of the host (not FQDN)

radosgw instance name between the \[ ] below should match the Ceph username created earlier

\[client.radosgw.xyz]
        host = {host-name}
        keyring = /etc/ceph/ceph.client.radosgw.keyring
        rgw socket path = /tmp/radosgw.sock
        log file = /var/log/ceph/radosgw.log
1. Configuration for httpd

    
    1. Open/etc/httpd/httpd.conffile. Uncomment#ServerNameand add the name of your server. Provide the fully qualified domain name of the server machine (output ofhostname-f)

ServerName {fgdn}
    1. Ensure that rewrite module is enabled in/etc/httpd/httpd.conf. Search for the following and add if not present

LoadModule rewrite_module modules/mod_rewrite.so
    1. Ensure that/etc/httpd/conf.d/fastcgi.conffileis present andFastCGI module is enabled. Search for the following and add if not present

LoadModule fastcgi_module modules/mod_fastcgi.so


    1. Ensure thatFastCgiWrapperis turned off in/etc/httpd/conf.d/fastcgi.conf

FastCgiWrapper Off

    
1. Configuration for ceph-radosgw


    1. Create a script/var/www/s3gw.fcgiwith the following content

#!/bin/sh
exec /usr/bin/radosgw -c /etc/ceph/ceph.conf -n client.radosgw.gateway
    1. Add execute privilege to the script


```
# chmod +x /var/www/s3gw.fcgi
```

    1. Create a gateway configuration file/etc/httpd/conf.d/rgw.conf. Replace {fqdn} withfully qualified domain name of the server machine (output ofhostname-f)and{email-address} with something

FastCgiExternalServer /var/www/s3gw.fcgi -socket /tmp/radosgw.sock

<VirtualHost \*:80>
  ServerName {fqdn}
  ServerAdmin {email-address}
  DocumentRoot /var/www
  RewriteEngine On
  RewriteRule ^/([a-zA-Z0-9-_.]\*)([/]?.\*) /s3gw.fcgi?page=$1&params=$2&%{QUERY_STRING} [E=HTTP_AUTHORIZATION:%{HTTP:Authorization},L]
  <IfModule mod_fastcgi.c>
    <Directory /var/www>
      Options +ExecCGI
      AllowOverride All
      SetHandler fastcgi-script
      Order allow,deny
      Allow from all
      AuthBasicAuthoritative Off
    </Directory>
  </IfModule>
  AllowEncodedSlashes On
  ErrorLog /var/log/httpd/rgw_error.log
  CustomLog /var/log/httpd/rgw_access.log combined
  ServerSignature Off
</VirtualHost>	
    1. Create the default socket directory for Ceph/var/run/cephand assign the ownership to userapache.radosgw init scripts run radosgw as the userapacheand can't create socket under/var/run/cephif its owned by another user


```
# mkdir -p /var/run/ceph
# chmod apache:apache /var/run/ceph
```

    1. Change the ownership of/var/log/ceph/radosgw.logtoapachefor the same reasons as above


```
# chown apache:apache /var/log/ceph/radosgw.log 
```


    
1. Restart httpd and ceph-radosgw services


```
# service restart httpd
# service restart ceph-radosgw
```



### radosgw user setup
A radosgw user has an access key and secret key, much like an IAM account. Since radosgw service is an S3 compatible API, any end user interaction with the service requires these keys. radosgw user is entirely different from the Ceph user management. Create a radosgw user and save the output. Eucalyptus will need the access and secret keys for interacting with radosgw


```
# radosgw-admin user create --uid={username} --display-name="{display-name}"
 
{ "user_id": "username",
  "display_name": "display-name",
  "email": "",
  "suspended": 0,
  "max_buckets": 1000,
  "auid": 0,
  "subusers": [],
  "keys": [
        { "user": "johndoe",
          "access_key": "11BS02LGFB6AL6H1ADMW",
          "secret_key": "vzCEkuryfn060dfee4fgQPqFrncKEIkh3ZcdOANY"}],
  "swift_keys": [],
  "caps": [],
  "op_mask": "read, write, delete",
  "default_placement": "",
  "placement_tags": [],
  "bucket_quota": { "enabled": false,
      "max_size_kb": -1,
      "max_objects": -1},
  "user_quota": { "enabled": false,
      "max_size_kb": -1,
      "max_objects": -1},
  "temp_url_keys": []}
```

### radosgw: Get users list and info about existing user

```
# radosgw-admin metadata list user
[
    "rgw-euca-osg",
    "shaon"

]
 
# radosgw-admin metadata get user:shaon
{
    "key": "user:shaon",
    "ver": {
        "tag": "_h1OKAJZKUi8aHz9NhZNqU1z",
        "ver": 1
    },
    "mtime": 1463420530,
    "data": {
        "user_id": "shaon",
        "display_name": "shaon",
        "email": "",
        "suspended": 0,
        "max_buckets": 1000,
        "auid": 0,
        "subusers": [],
        "keys": [
            {
                "user": "shaon",
                "access_key": "FXI6H5QH1XLY0D8J8OWP",
                "secret_key": "hHaBd9kcqaCCBdCSZc3dczMrLLkriI5HUqBEAhWF"
            }
        ],
        "swift_keys": [],
        "caps": [],
        "op_mask": "read, write, delete",
        "default_placement": "",
        "placement_tags": [],
        "bucket_quota": {
            "enabled": false,
            "max_size_kb": -1,
            "max_objects": -1
        },
        "user_quota": {
            "enabled": false,
            "max_size_kb": -1,
            "max_objects": -1
        },
        "temp_url_keys": []
    }
}
 
```

### radosgw: Modify user e.g change max bucket

```
# radosgw-admin user modify --uid=rgw-euca-osg --max-buckets=5000
{
    "user_id": "rgw-euca-osg",
    "display_name": "radosgw user for euca osg",
    "email": "",
    "suspended": 0,
    "max_buckets": 5000,
    "auid": 0,
    "subusers": [],
    "keys": [
        {
            "user": "rgw-euca-osg",
            "access_key": "73X4FTH5GVS2S26A0YCI",
            "secret_key": "K00Ob5zzX5tOS8YiyWLdyMtc6DjTYRW944NSi5so"
        }
    ],
    "swift_keys": [],
    "caps": [],
    "op_mask": "read, write, delete",
    "default_placement": "",
    "placement_tags": [],
    "bucket_quota": {
        "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1
    },
    "user_quota": {
        "enabled": false,
        "max_size_kb": -1,
        "max_objects": -1
    },
    "temp_url_keys": []
}
```



## Related articles
Related articles appear here based on the labels you select. Click to edit the macro and add or change labels.

false5STORfalsemodifiedtruepagelabel = "ceph-rgw" and type = "page" and space = "STOR"kb-how-to-article

true

|  | 
|  --- | 
|  | 





*****

[[category.storage]] 
[[category.confluence]] 
[[category.ceph]] 
[[category.objectstorage]]

