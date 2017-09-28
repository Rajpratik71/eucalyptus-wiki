
## Tracking
* Version: 4.1.0
* Issue: [EUCA-9886](https://eucalyptus.atlassian.net/browse/EUCA-9886)

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
| 1 complete | Add binding support for bucket tagging | Binding support needs to be added to accept GET/PUT/DELETE requests | Must have |  | 
| 2 complete | Change schema for Bucket Tagging | Bucket tagging implementation requires schema changes in eucalyptus_osg | Must have |  | 
| 3 complete | Implement SET tagging | Implement SET bucket tagging functionality | Must have |  | 
| 4 complete | Implement GET tagging | Implement DELETE bucket tagging functionality | Must have |  | 
| 5 complete | Implement DELETE tagging | Implement DELETE bucket tagging functionality | Must have |  | 
| 6 incomplete | Set one or more TagSets to a bucket | | | |
| 7 incomplete | Should return 204 with no xml | | | |
| 8 incomplete | Get TagSets from a bucket | | | |
| 9 incomplete | Should return 200 with an AWS compatible xml  | | | |
| 10 incomplete | Delete TagSets of a bucket | | | |
| 11 incomplete | Should return 204 with no xml | | | |
| 12 incomplete | Set bucket tagging with an empty TagSet | | | |
| 13 incomplete | Should throw AWS compatibleMalformedXML exception | | | |
| 14 incomplete | Get bucket tagging when there is no TagSet associated | | | |
| 15 incomplete | Should return 404 withNoSuchTagSet exception in xml | | | |
| 16 incomplete | Get tagging when the bucket does not exists | | | |
| 17 incomplete | Typical NoSuchBucket exception | | | |
| 18 incomplete | Delete tags from a bucket when there is not TagSet associated | | | |
| 19 incomplete | Delete bucket removes the tags from DB | | | |
| 20 incomplete | Maximum number of tags per bucket is 10 | | | |

## User interaction

* User should be able to use any AWS clients to perform bucket tagging operations.
* Example using AWS Java SDK:[https://gist.github.com/shaon/ad660e6543745268f84e](https://gist.github.com/shaon/ad660e6543745268f84e)
* TODO: add list of s3 clients


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


*****

[[category.storage]] 
[[category.confluence]] 
[[category.objectstorage]]
