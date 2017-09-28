This document describes some updates to the cloudformation service in the 5.0 release.

# Functionality Changes

## CFN_V1 authn/authz
Support is added for the internal authentication/authorization approach used for signals and retrieval of resource metadata with cloudformation instance initialization scripts [EUCA-12211](https://eucalyptus.atlassian.net/browse/EUCA-12211).

There is a new  _CloudFormationCfnAuthQueryPipeline_ that handles requests with an **_Authorization_**  http header using the  **_CFN_V1_** scheme.

The  _CloudFormationCfnAuthQueryPipeline_  extends the regular pipeline, providing its own authentication handler. A new query pipeline request check is added:

```java
boolean QueryPipeline#validAuthForPipeline( MappingHttpRequest request )
```

this is implemented by  _CloudFormationCfnAuthQueryPipeline_  and  _CloudFormationQueryPipeline_ for checking whether to accept a request based on the authentication in use.

The  _CloudFormationCfnAuthQueryPipeline_  uses the _CloudFormationCfnAuthenticationStage_ to provision the  _CloudFormationCfnAuthenticationHandler_  which handles authentication.

The  **_CFN_V1_** scheme relies on an instance providing instance identity document value from the instance meta-data for authentication of an instance. The value for authorization consists of the base64 encoded instance identity document concatenated with it's related (base64) signature value with whitespace removed. The identity document contains the instance identifier and the signature can be verified for the document to ensure that it was created by the compute service. 

The handler verifies the signature and for valid requests it populates the context with the accounts "admin" principal but without any iam permissions. There is a new  _CfnIdentityDocumentCredential_ added to the subject that can be later used by the service to specially authorize a request. A cache is used for processed authorization headers as these are expensive to process.

The  _CloudFormationService_ is updated so that the actions:

* DescribeStackResource
* SignalResource

will authorize a request if iam permissions allow or if the requesting subject contains a  _CfnIdentityDocumentCredential_  where the originating instance is a resource of the stack being accessed.

# References

* [EC2 instance identity documents (docs.aws.amazon.com)](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-identity-documents.html)

[EUCA-12994 JIRA (eucalyptus.atlassian.net)](https://eucalyptus.atlassian.net/browse/EUCA-12994)

*****

[[category.confluence]] 
