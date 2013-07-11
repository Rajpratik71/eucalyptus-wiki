**Author:** Mitch Garnaat, et al

**Website:** <a href="https://github.com/boto/boto">https://github.com/boto/boto</a>

**Download URL:** <a href="http://pypi.python.org/pypi/boto">http://pypi.python.org/pypi/boto</a>

**License:** MIT

***

Python library 'boto' is an integrated interface to current and future services offered by AWS and by AWS-compatible services, such as Eucalyptus.

# Installation

You can follow any installation method recommended in boto's documentation, such as

* Install via [pip installer](http://www.pip-installer.org/):
```
	pip install boto
```
* Install from source:
```
	git clone git://github.com/boto/boto.git
	cd boto
	python setup.py install
```

# Using boto with Eucalyptus

As with AWS, one must pass the access key id and the secret access key to the library. In addition to that, one must override the defaults for cloud endpoints (S3 and EC2), their ports, and their service paths. The code snippets below assume that we define the values specific to your Eucalyptus installation ahead of time:

```
euca_ec2_host="10.111.5.3"
euca_s3_host="10.111.5.3"
euca_id="CYYBSGXPRSDFFOEUZXOBN"
euca_key="fFwIUBcFjEXvkevE5f4cVmMshsZLUHM75mhe8Eyr"
```

Where `euca_ec2_host` is the host running the CLC and `euca_s3_host` is the host running the Walrus. For instance, if EC2_URL is set to http://10.111.5.3:8773/services/Eucalyptus then `euca_ec2_host` should be set to `10.111.5.3`. When CLC and Walrus are co-located, the hostname will be the same for both.

## Eucalyptus EC2 interface

```
import boto
from boto.ec2.regioninfo import RegionInfo
euca_region = RegionInfo(name="eucalyptus", endpoint=euca_ec2_host)
ec2conn = boto.connect_ec2(
	aws_access_key_id=euca_id,
	aws_secret_access_key=euca_key, 
	is_secure=False,
	port=8773, 
	path="/services/Eucalyptus", 
	region=euca_region)
ec2conn.get_all_zones()
```
## Eucalyptus S3 interface

```
import boto
s3conn = boto.connect_s3(
	aws_access_key_id=euca_id,
	aws_secret_access_key=euca_key,
	is_secure=False,
	port=8773,
	path="/services/Walrus",
	host=euca_s3_host)
s3conn.get_all_buckets()
```

*****

[[category.tools]]