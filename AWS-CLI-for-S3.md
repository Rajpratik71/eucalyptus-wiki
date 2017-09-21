The AWS CLI has a few bits of configuration that are required in order for it to work with Eucalyptus for S3 calls.

To configure AWS CLI to sign requests with V2 signatures, add the following to your .aws/conf file:


```
s3 = 

 signature_version = s3
```
Note the indentation is necessary.

To create V4 signatures:


```
s3 =

  signature_version = s3v4
```
It is necessary to explicitly configure the signature_version in order to use AWS CLI with Eucalyptus for S3.





*****

[[category.storage]] 
[[category.confluence]] 
