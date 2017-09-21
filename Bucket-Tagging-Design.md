

| 4.1.0 | 
| [EUCA-9886](https://eucalyptus.atlassian.net/browse/EUCA-9886) | 
| DRAFT | 
|  | 
|  | 
|  | 
|  | 
|  --- | 
|  --- | 
|  --- | 
|  --- | 
|  --- | 
|  --- | 
|  --- | 
| 4.1.0 | 
| [EUCA-9886](https://eucalyptus.atlassian.net/browse/EUCA-9886) | 
| DRAFT | 
|  | 
|  | 
|  | 
|  | 


## Goals

* Implement PUT bucket tagging
* Implement GET bucket tagging
* Implement DELETE bucket tagging

 **S3 References:** 


* [http://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketPUTtagging.html](http://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketPUTtagging.html)


* [http://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketGETtagging.html](http://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketGETtagging.html)


* [http://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketDELETEtagging.html](http://docs.aws.amazon.com/AmazonS3/latest/API/RESTBucketDELETEtagging.html)


## Schema Changes
Following schema changes have been made to implement bucket tagging.

bucket_tagging_erd


## IAM Dependencies

* PUT: operation depends on s3:PutBucketTagging
* GET: operation depends on s3:GetBucketTagging
* DELETE: operation depends on s3:PutBucketTagging


## Sample XML request

```
<Tagging>
  <TagSet>
    <Tag>
      <Key>Project</Key>
      <Value>Project One</Value>
    </Tag>
    <Tag>
      <Key>User</Key>
      <Value>jsmith</Value>
    </Tag>
  </TagSet>
</Tagging>
```

## Sample PUT response

```
HTTP/1.1 204 No Content
x-amz-id-2: YgIPIfBiKa2bj0KMgUAdQkf3ShJTOOpXUueF6QKo
x-amz-request-id: 236A8905248E5A01
Date: Wed, 01 Oct  2012 12:00:00 GMT
```



## Requirements


| # | Title | Description | Importance | Notes | 
|  --- |  --- |  --- |  --- |  --- | 
| 1complete | Add binding support for bucket tagging | Binding support needs to be added to accept GET/PUT/DELETE requests | Must have |  | 
| 2complete | Change schema for Bucket Tagging | Bucket tagging implementation requires schema changes in eucalyptus_osg | Must have |  | 
| 3complete | Implement SET tagging | Implement SET bucket tagging functionality | Must have |  | 
| 4complete | Implement GET tagging | Implement DELETE bucket tagging functionality | Must have |  | 
| 5complete | Implement DELETE tagging | Implement DELETE bucket tagging functionality | Must have |  | 


## User interaction

* User should be able to use any AWS clients to perform bucket tagging operations.
* Example using AWS Java SDK:[https://gist.github.com/shaon/ad660e6543745268f84e](https://gist.github.com/shaon/ad660e6543745268f84e)
* TODO: add list of s3 clients


## QA
 **Happy Tests** 

6incompleteSet one or more TagSets to a bucket7incompleteshould return 204 with no xml8incompleteGet TagSets from a bucket9incompleteshould return 200 with an AWS compatible xml10incompleteDelete TagSets of a bucket11incompleteshould return 204 with no xml **Negative Tests** 

12incompleteSet bucket tagging with an empty TagSet13incompleteshould throw AWS compatibleMalformedXML exception14incompleteGet bucket tagging when there is no TagSet associated15incompleteshould return 404 withNoSuchTagSet exception in xml16incompleteGet tagging when the bucket does not exists17incompletetypical NoSuchBucket exception18incompleteDelete tags from a bucket when there is not TagSet associated19incompleteDelete bucket removes the tags from DB20incompleteMaximum number of tags per bucket is 10




## Required IAM permission

```
{
  "Id": "Policy1409280973083",
  "Statement": [
    {
      "Action": [
        "s3:CreateBucket",
        "s3:DeleteBucket",
        "s3:PutBucketTagging",
        "s3:GetBucketTagging",
      ],
      "Effect": "Allow",
      "Resource": "*",
    }
  ]
}
```



## Questions
Below is a list of questions to be addressed as a result of this requirements document:



|  | Action | Current State | Assignee | 
|  --- |  --- |  --- |  --- | 
| 1 | How does the schema changes should look like? (possible options are  _Schema Changes_  section) | Resolved | Zach | 
| 2 | Still not sure, how to make OSG response 204 instead of an empty xml | Resolved | Shaon | 



*****

[[category.storage]] 
[[category.confluence]] 
