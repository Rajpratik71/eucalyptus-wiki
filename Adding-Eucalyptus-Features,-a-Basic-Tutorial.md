This tutorial will walk you through adding a new API call to a Eucalyptus service and testing it. We'll follow a test-driven-development flow so we'll build a test first, then run it against AWS, then develop the feature in Eucalyptus, followed by verification with the new test pointed at a new Eucalyptus install.

 **Basic Overview** 


1. Get Eutester code
1. Add a test to Eutester for the call to be added in Euca
1. Verify the test against AWS
1. Get the Eucalyptus code
1. Build it in a QA Lifecycle Managed system
1. Add the new feature code
1. Test with Eutester pointed at the Eucalyptus deployment instead of AWS

    

    


### Prerequisites. Getting the environment set up.
Assumes you have QA system credentials, ssh key on the internal git server so you can checkout the code, and have received AWS credentials for API access (access key and secret key) from the QA team to use the Eucalyptus AWS account for validation testing.


1. Get a QA-build Eucalyptus system. Follow instructions in:[[Euca QA|Deploy-for-Development]]
1. On your development system (can be one of the QA systems deployed in step 1 or a laptop/desktop), checkout the [eutester](https://github.com/eucalyptus/eutester.git) code.
    1. 
```
git clone https://github.com/eucalyptus/eutester.git
```

    1. Follow build instructions in README in eutester/
    1. To run eutester against AWS instead of Eucalyptus you'll need to create a _eucarc_  file with the following content:
    1. Assuming you want to test against the us-east region of AWS use the following or substitute the ec2, sts, and s3 urls for the appropriate region


```
EC2_ACCESS_KEY='<yourkey>'
EC2_SECRET_KEY='<yoursecretkey>'
EC2_URL='http://ec2.us-east-1.amazonaws.com'
S3_URL='http://s3.amazonaws.com'
TOKENS_URL='http://iam.amazonaws.com'
EUARE_URL='http://sts.us-east-1.amazonaws.com'
EC2_ACCOUNT_NUMBER=fakeaccountnumber
EC2_USER_ID=fakeuserid
 
```
Note that you can use fake account and user-id names here if you only want to test S3 API, but for EC2 tests you may need the correct information.


    1. Run a quick test against AWS, for example S3, to verify configuration correctness:
    1. From the base eutester directory run (picking test_bucket_acl tests as an example)


```
python testcases/cloud_user/s3/bucket_tests.py --region us-east --tests test_bucket_acl --credpath . --endpoint s3.amazonaws.com
```
This should output some information about doing ACL tests against a bucket. (As I type this there is a bug in the test for the 'bucket-owner-read' canned-acl that needs to be fixed to make the test pass.



    

Next up...adding a test for a new Eucalyptus service feature....


### Add a Eutester Test Case for a new Feature
As an example, let's add a test for S3 Bucket location:eutester/testcases/cloud_user/s3/bucket_tests.py


### Write the Feature Code
For this example, let's add a 'HEAD' HTTP operation on the object storage service that simply returns HTTP 200 with a header'x-amz-euca-response: hello-world'.


1. Add message type class
    1. InObjectStorage.groovy, add a new class:


```
class HeadServiceType extends ObjectStorageRequestType {}
 
class HeadServiceResponseType extends ObjectStorageResponseType {
	List<MetadataEntry> metaData = new ArrayList<MetadataEntry>()
}
```


    
1. Add the class's binding toosg-binding.xml :
    1. 
```xml
    <!-- HEAD OSG SERVICE -->

    <mapping name="HeadService"
             class="com.eucalyptus.objectstorage.msgs.HeadServiceType"
             extends="com.eucalyptus.objectstorage.msgs.ObjectStorageRequestType">
        <collection field="metaData"
                    factory="org.jibx.runtime.Utility.arrayListFactory" usage="optional">
            <structure map-as="com.eucalyptus.storage.msgs.s3.MetaDataEntry"/>
        </collection>
    </mapping>

    <mapping name="HeadServiceResponse"
             class="com.eucalyptus.objectstorage.msgs.HeadServiceResponseType">
    </mapping>
```


    
1. Add the operation in the bindings for OSG
    1. ObjectStorageHEADBinding.java
    1. Add to _populateOperationMap()_ 


```
newMap.put(SERVICE + HttpMethod.HEAD.toString(), "HeadService");
```


    

    
1. Add imports to both ObjectStorageGateway.java andObjectStorageService.java, retaining the alphabetical order of the existing imports:

    

    


```java
import com.eucalyptus.objectstorage.msgs.HeadServiceResponseType;
import com.eucalyptus.objectstorage.msgs.HeadServiceType;


```

1. In ObjectStorageService.java, add the abstract method declaration:

    

    


```java
  public abstract HeadServiceResponseType HeadService(HeadServiceType request) throws S3Exception;
```

1. Add method in service implementation: ObjectStorageGateway.java

    
    1. Add a new service operation method:


```
public HeadServiceResponseType headService(HeadServiceType request) throws S3Exception {
	HeadServiceResponseType reply = request.getReply();
	reply.setStatus(HttpResponseStatus.OK);
	reply.setStatusMessage("OK");
	reply.setMetadata(Lists.newArrayList(new MetaDataEntry("euca-test", "hello world")));
	return reply;
}
```


    
1. Compile fromeucalyptus/clc directory usingmake
1. Install new version fromeucalyptus/clc directory using make install
1. Start/restart the JVM

    
    * 
```
service eucalyptus-cloud restart 
                        (or start)
```


    
1. Test using curl:
    * 
```
curl -X HEAD http://localhost:8773/services/objectstorage/ -debug -verbose
```


    



*****

[[category.confluence]] 
