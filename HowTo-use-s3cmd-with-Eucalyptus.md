## Introduction

This simple set of instructions would help you to, use s3cmd against Eucalyptus Walrus just like you could use it against AWS S3.

## s3cmd repository

Eucalyptus keeps a [local fork](https://github.com/eucalyptus/s3cmd) of the [upstream](https://github.com/s3tools/s3cmd) s3cmd repository 

## Installation Instructions

```
git clone https://github.com/eucalyptus/s3cmd
cd s3cmd
python setup.py install
```

## Writing the s3cmd configuration file

A sample configuration is provided for [reference](https://gist.github.com/jeevanullas/5114186/raw/dcda873adb8c76f9d8ffcad370e3f1761b67daec/gistfile1.txt) , just save it as s3cfg on the system.

We need to modify the following 4 variables in this file, according to the private cloud configuration

* access_key 
* secret_key 
* host_base 
* host_bucket

# Testing

Here are some basic things we can do to verify, this works

###  To list all the buckets

```
s3cmd -c s3cfg ls
```

### To create a new bucket

```
s3cmd -c s3cfg mb s3://newbucket
```

### To upload a file into the new bucket

```
s3cmd -c s3cfg put file.txt s3://newbucket
```

### To list content of the new bucket

```
s3cmd -c s3cfg ls s3://newbucket
```

### To get the file from the new bucket

```
s3cmd -c s3cfg get s3://newbucket/file.txt
```

### To delete the object from the new bucket

```
s3cmd -c s3cfg del s3://newbucket/file.txt
```

### To delete the bucket

```
s3cmd -c s3cfg rb s3://newbucket
```

## Problems?

Report at [eucalyptus user group](https://groups.google.com/a/eucalyptus.com/forum/#!forum/euca-users)

*****
[[category.HOWTO]]