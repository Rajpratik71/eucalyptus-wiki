### Install dependencies
```
yum install python-devel python-setuptools gcc make python-virtualenv java-1.6.0-openjdk.x86_64 git ntp
```

### Install Jenkins
```
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat/jenkins.repo
sudo rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
yum install jenkins
```

### Sync NTP
```
chkconfig ntpd on
service ntpd start
ntpdate -u pool.ntp.org
```

### Unbundle /var/lib/jenkins tarball
```
pushd /var/lib/jenkins
wget http://MYURL/micro-qa-v2.2-src.tgz
tar zxfv micro-qa-v2.2-src.tgz
popd
service jenkins restart
```