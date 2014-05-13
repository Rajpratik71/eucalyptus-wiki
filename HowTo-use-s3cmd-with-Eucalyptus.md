## Introduction

The s3cmd tool is a simple command line tool for accessing buckets in S3 (AWS) and Walrus (Eucalyptus).

## Installation

Eucalyptus keeps a [local fork](https://github.com/eucalyptus/s3cmd) of the [upstream (https://github.com/s3tools/s3cmd) s3cmd repository.  To install:

```
git clone https://github.com/eucalyptus/s3cmd
cd s3cmd
python setup.py install
```

## Configuration and Usage

A sample configuration is provided for [reference](https://gist.github.com/jeevanullas/5114186/raw/dcda873adb8c76f9d8ffcad370e3f1761b67daec/gistfile1.txt); just save it as s3cfg on the system.

We need to modify the following 4 variables in this file, according to the private cloud configuration:

* access_key 
* secret_key 
* host_base 
* host_bucket
* service_path

To list buckets:

```
s3cmd -c s3cfg ls
```

To create a new bucket:

```
s3cmd -c s3cfg mb s3://newbucket
```

To upload a file into the new bucket:

```
s3cmd -c s3cfg put file.txt s3://newbucket
```

To list the contents of the new bucket:

```
s3cmd -c s3cfg ls s3://newbucket
```

To get the file from the new bucket:

```
s3cmd -c s3cfg get s3://newbucket/file.txt
```

To delete the object from the new bucket:

```
s3cmd -c s3cfg del s3://newbucket/file.txt
```

To delete the bucket:

```
s3cmd -c s3cfg rb s3://newbucket
```

## Known Issues

There are no currently known issues with this tool.

## Questions/Issues

If you have any questions about using Fog with Eucalyptus, please first check the [knowledgebase](https://engage.eucalyptus.com/customer/portal/articles/search?q=s3cmd).  

If you do not find an answer in the knowledgebase, please [ask a question](https://engage.eucalyptus.com/customer/portal/questions/new?q=s3cmd).

*****
[[category.HOWTO]]
[[category.tools]]