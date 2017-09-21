
## Download


This version of the script has been modified to work against Eucalyptus Object Storage. Do not download the one provided by Amazon! The changes in this script allow Eucalyptus endpoints and computes the header signature correctly.


## Introduction
s3curl is a perl wrapper script around curl. It computes the signature and parameters based on the request, and uses curl to send HTTP requests to an endpoint. IMO among all the S3 tools, s3curl is the simplest to use as it speaks HTTP which is inline with the S3 REST API. It is also lightweight and involves minimum processing of input/output unlike boto or s3cmd.This wiki page provides:


* How to use s3curl against a service that implements AWS S3 API
* s3curl.pl script modified to work with Eucalyptus OSG


## Step-by-step guide

* Download the s3curl.pl script (see above).
* s3curl depends on perl's cryptographic HMAC digest module. To install it: 


```
yum install -y perl-Digest-HMAC




```

* Create .s3curl file in your home directory. This file contains one or more sections/paragraphs of connection information. Each section represents an S3 endpoint and has a nickname. It encapsulates url to the S3 endpoint,AWS Access Key ID and Secret Access Key pairs of the account for S3 operations. There are several styles of S3 Eucalyptus endpoints. Format for the .s3curl file with examples:

%awsSecretAccessKeys = (
  ufshost => {
    url => '10.111.1.1',
    id => 'access-key-id' {typically AKIA...},
    key => 'secret-access-key'  {typically a random-looking 40-character string},
  },
  ufsdns => {
    url => 's3.e-15.autoqa.qa1.eucalyptus-systems.com',
    id => 'access-key-id',
    key => 'secret-access-key',
  },
  riakcs => {
    url => '10.111.5.79',
    id => 'access-key-id',
    key => 'secret-access-key',
  },
  aws => {
    url => 's3.amazonaws.com',
    id => 'access-key-id',
    key => 'secret-access-key',
  }
  awsuswest1 => {
    url => 's3-us-west-1.amazonaws.com',
    id => 'access-key-id',
    key => 'secret-access-key'
  }
);


* You are all set, start executing S3 operations using the below examples


```
./s3curl.pl --id ufshost-- "http://10.111.1.1:8773/services/objectstorage"
```
Or


```
./s3curl.pl --id ufsdns -- "http://s3.e-15.autoqa.qa1.eucalyptus-systems.com:8773"
```


IP address used in the above command after '--' should match the url in the .s3curl file. A mismatch might result in signature computation error. OSG's DNS name can also be used in which case the trailing /services/objectstorage in the script invocation should be removed. You can find that DNS name in a credentials .ini file such as ~/.euca/euca-admin.ini if your cloud is deployed in EQ by Calyptos.






### Optional flags

* A lot of output from the S3 API is in xml format. If you want this output to be nicely organized for easier reading, you can use xmllintto format the output from the s3curl command. If the body is NOT in XML, or if you use any of the debugging flags below, the xmllint will not work and you won't see much of anything.

    


```
./s3curl.pl --id ufshost -- "http://10.111.1.1:8773/services/objectstorage" | xmllint --format -
```

* For debugging the request and signature string created by s3curl, use --debug flag


```
./s3curl.pl --debug --id ufshost -- "http://10.111.1.1:8773/services/objectstorage"
```

* For debugging the request and response headers, use the '-v' curl flag after the '-- ' :


```
./s3curl.pl --id ufshost -- "http://10.111.1.1:8773/services/objectstorage" -i
```



## Example s3curl commands

### Buckets

* List all your buckets:


```
./s3curl.pl --id ufshost -- "http://10.111.1.1:8773/services/objectstorage"
```



* List objects in bucket:


```
./s3curl.pl --id ufshost -- "http://10.111.1.1:8773/services/objectstorage/bucket"
```



* Create bucket:


```
./s3curl.pl --id ufshost --createBucket -- "http://10.111.1.1:8773/services/objectstorage/bucket"
```

* Create bucket with [Canned ACL](http://docs.aws.amazon.com/AmazonS3/latest/dev/acl-overview.html#canned-acl):


```
./s3curl.pl --id ufshost --createBucket -- -H 'x-amz-acl: public-read' "http://10.111.1.1:8773/services/objectstorage/bucket"
```

* Create bucket with ACL:


```
./s3curl.pl --id ufshost --createBucket -- -H 'x-amz-grant-read:uri="http://acs.amazonaws.com/groups/global/AllUsers"' "http://10.111.1.1:8773/services/objectstorage/bucket"
```

* Create bucket with its location constrained to a particular AWS region (only applies to AWS):


```
./s3curl.pl --id ufshost --createBucket us-west-2 -- "http://s3.amazonaws.com/bucket"
```

* Show a bucket's location constraint (only applies to AWS):


```
./s3curl.pl --id ufshost -- "http://s3.amazonaws.com/bucket?location"
```


After you create a bucket with a given location (a.k.a. region) constraint, you can only access the bucket from that region's endpoint. This applies to get/put/delete object and deleting the bucket. (Only applies to AWS)




* Delete bucket:


```
./s3curl.pl --id ufshost --delete -- "http://10.111.1.1:8773/services/objectstorage/bucket"
```



* Get bucket ACL:


```
./s3curl.pl --id ufshost -- "http://10.111.1.1:8773/services/objectstorage/bucket?acl"
```



* Put bucket ACL (ACL as xml body):


```
./s3curl.pl --id ufshost --put <path-to-acl-file> -- "http://10.111.1.1:8773/services/objectstorage/bucket?acl"
```
Example contents of acl-file:

<AccessControlPolicy xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Owner>
    <ID>2662e512f4f222dc374d1614451ab492dd63371b202732564e55483331357174</ID>
    <DisplayName>eucalyptus</DisplayName>
  </Owner>
  <AccessControlList>
    <Grant>
      <Grantee xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="Group">
        <URI>http://acs.amazonaws.com/groups/s3/LogDelivery</URI> 
      </Grantee>
      <Permission>READ_ACP</Permission>
    </Grant>
  </AccessControlList>
</AccessControlPolicy>


* Get bucket versioning status:


```
./s3curl.pl --id ufshost -- "http://10.111.1.1:8773/services/objectstorage/bucket?versioning"
```



* Put (enable/suspend) bucket versioning status:


```
./s3curl.pl --id ufshost --put <path-to-versioning-file> -- "http://10.111.1.1:8773/services/objectstorage/bucket?versioning"
```
Example contents of versioning-file:

<?xml version="1.0" encoding="UTF-8"?>
<VersioningConfiguration>
  <Status>Enabled</Status> 
</VersioningConfiguration>

AWS replies to a "PUT bucket versioning" request before it has applied the setting! If you do a PUT followed immediately by a GET, the GET may return the old value. Waiting 5 seconds or more after a PUT, before a GET, always seems to return the correct setting. (Even 2 seconds is sometimes too short.)




* List all versions in a bucket:


```
./s3curl.pl --id ufshost -- "http://10.111.1.1:8773/services/objectstorage/bucket?versions"
```



* List multipart uploads in progress


```
./s3curl.pl --id ufshost -- "http://10.111.1.1:8773/services/objectstorage/bucket?uploads"
```



* Get CORS configuration


```
./s3curl.pl --id ufshost -- "http://10.111.1.1:8773/services/objectstorage/bucket?cors"
```



* Put a new CORS configuration. Note the --calculateContentMd5option, which sends a required header to AWS containing the MD5 digest of the contents of the XML file.


```
./s3curl.pl --id ufshost --put CORS-config.xml --calculateContentMd5 -- "http://10.111.1.1:8773/services/objectstorage/bucket?cors"
```
Example of CORS configuration file:

<CORSConfiguration>
    <CORSRule>
        <AllowedOrigin>\*</AllowedOrigin>
        <AllowedMethod>GET</AllowedMethod>
        <AllowedMethod>POST</AllowedMethod>
        <AllowedMethod>PUT</AllowedMethod>
        <AllowedMethod>DELETE</AllowedMethod>
        <MaxAgeSeconds>3000</MaxAgeSeconds>
        <AllowedHeader>\*</AllowedHeader>
    </CORSRule>
</CORSConfiguration>


* Delete CORS configuration


```
./s3curl.pl --id ufshost --delete -- "http://10.111.1.1:8773/services/objectstorage/bucket?cors"
```



### Objects

* Put object (upload contents from file):


```
./s3curl.pl --id ufshost --put <path-to-file> -- "http://10.111.1.1:8773/services/objectstorage/bucket/object"
```

* Put object with metadata:


```
./s3curl.pl --id ufshost --put <path-to-file> -- -H 'x-amz-meta-somekey: somevalue' "http://10.111.1.1:8773/services/objectstorage/bucket/object"
```

* Put object with [Canned ACL](http://docs.aws.amazon.com/AmazonS3/latest/dev/acl-overview.html#canned-acl):


```
./s3curl.pl --id ufshost --put <path-to-file> -- -H 'x-amz-acl: public-read' "http://10.111.1.1:8773/services/objectstorage/bucket/object"
```

* Put object with ACL:


```
./s3curl.pl --id ufshost --put <path-to-file> -- -H 'x-amz-grant-read:uri="http://acs.amazonaws.com/groups/global/AllUsers"' "http://10.111.1.1:8773/services/objectstorage/bucket/object"
```



* Head object (get object info and metadata):


```
./s3curl.pl --id ufshost --head -- "http://10.111.1.1:8773/services/objectstorage/bucket/object" 
```



* Get object (download contents to file):


```
./s3curl.pl --id ufshost -- "http://10.111.1.1:8773/services/objectstorage/bucket/object" > <path-to-file>
```

* Get object with range(download contents to file):


```
./s3curl.pl --id ufshost -- -H 'Range: bytes=0-10' "http://10.111.1.1:8773/services/objectstorage/bucket/object" > <path-to-file>
```



* Delete object:


```
./s3curl.pl --id ufshost --delete -- "http://10.111.1.1:8773/services/objectstorage/bucket/object"
```



* Delete version:


```
./s3curl.pl --id ufshost --delete -- "http://10.111.1.1:8773/services/objectstorage/bucket/object?versionId=<version-id>"
```



* Copy object:


```
./s3curl.pl --id ufshost --copySrc sourcebucket/sourceobject -- "http://10.111.1.1:8773/services/objectstorage/destinationbucket/destinationobject"
```

* Copy specific version of object (can be used for restoring deleted objects in versioned buckets):


```
./s3curl.pl --id ufshost --copySrc sourcebucket/sourceobject?versionId=<version-id> -- "http://10.111.1.1:8773/services/objectstorage/destinationbucket/destinationobject"
```

* Get object ACL:


```
./s3curl.pl --id ufshost -- "http://10.111.1.1:8773/services/objectstorage/bucket/object?acl"
```

* Put object ACL (ACL as xml body):


```
./s3curl.pl --id ufshost --put <path-to-acl-file> -- "http://10.111.1.1:8773/services/objectstorage/bucket/object?acl"
```
Example contents of acl-file:

<AccessControlPolicy xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Owner>
    <ID>2662e512f4f222dc374d1614451ab492dd63371b202732564e55483331357174</ID>
    <DisplayName>eucalyptus</DisplayName>
  </Owner>
  <AccessControlList>
    <Grant>
      <Grantee xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="Group">
        <URI>http://acs.amazonaws.com/groups/s3/LogDelivery</URI> 
      </Grantee>
      <Permission>READ_ACP</Permission>
    </Grant>
  </AccessControlList>
</AccessControlPolicy>


* Initiate multipart upload:


```
./s3curl.pl --id ufshost --post -- "http://10.111.1.1:8773/services/objectstorage/bucket/object?uploads"
```



* Upload part(upload contents from file):


```
./s3curl.pl --id ufshost --put <path-to-file> -- "http://10.111.1.1:8773/services/objectstorage/bucket/object?uploadId=<upload-id-from-initiate-mpu-response>&partNumber=<part-number-integer>"
```


NoteMake note of part number and etag from response. They are required for completing the multipart upload.




* Complete multipart upload:


```
./s3curl.pl --id ufshost --post <path-to-mpu-file> -- "http://10.111.1.1:8773/services/objectstorage/bucket/object?uploadId=<upload-id-from-initiate-mpu-response>"
```
Example contents of mpu-file:

<CompleteMultipartUpload>
  <Part>
    <PartNumber>1</PartNumber>
    <ETag>"<etag-from-upload-part-response>"</ETag>
  </Part>
  <Part>
    <PartNumber>2</PartNumber>
    <ETag>"<etag-from-upload-part-response>"</ETag>
  </Part>
</CompleteMultipartUpload>


* Abort upload:


```
./s3curl.pl --id ufshost --delete -- "http://10.111.1.1:8773/services/objectstorage/bucket/object?uploadId=<upload-id-from-initiate-mpu-response>"
```



* List parts (for a multipart upload in progress):


```
./s3curl.pl --id ufshost -- "http://10.111.1.1:8773/services/objectstorage/bucket/object?uploadId=<upload-id-from-initiate-mpu-response>"
```



* Disable s3curl from sending Expect: 100-continue in PUT requests:


```
./s3curl.pl --id ufshost --put <path-to-acl-file> -- -H "Expect:" "http://10.111.1.1:8773/services/objectstorage/bucket/object?acl"
```



### Other

* Security token associated with IAM roles


```
./s3curl.pl --id ufshost -- -H "x-amz-security-token: <security-token>" "http://10.111.1.1:8773/services/objectstorage/bucket"
```



## Related articles
[AWS S3 REST API](http://docs.aws.amazon.com/AmazonS3/latest/API/APIRest.html)

[[S3 API Support in Eucalyptus Object Storage|S3-API-Support-in-Eucalyptus-Object-Storage]]
## Possible To-do's

* Thescript attached to this page has been modified to work against Eucalyptus OSG. It was edited from an earlier version of the [s3curl](https://aws.amazon.com/code/128) script contributed from the Amazon community and hosted on an [AWS public Web page](https://aws.amazon.com/code/128).
* The Eucalyptus version of the script is not tracked in git or github. We should add it to some github repository.
* We could merge into our script the changes made to the AWS-contributed [s3curl](https://aws.amazon.com/code/128) script since the version we modified, if there are fixes/features we want.
* As new fixes/features are added to object storage in Eucalyptus or AWS, this script should be updated to match. Fetch changes from AWS's script when possible and merge them into ours.





*****

[[category.storage]] 
[[category.confluence]] 
