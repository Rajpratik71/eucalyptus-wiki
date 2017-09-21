This document details the IAM/STS service changes for OpenID Connect (OIDC) support in Eucalyptus 4.4.

* [Scope](#scope)
  * [API](#api)
  * [Related Service Changes](#related-service-changes)
* [Design](#design)
  * [Authentication](#authentication)
  * [Authorization](#authorization)
  * [Validation](#validation)
  * [Persistence](#persistence)
* [Administration](#administration)
  * [Quotas](#quotas)
  * [Delegate Account](#delegate-account)
  * [Account Deletion](#account-deletion)
  * [Cloud Properties](#cloud-properties)
* [API Details](#api-details)
  * [Actions](#actions)
  * [Condition keys](#condition-keys)
* [References](#references)



## Scope
This section gives a high-level summary of the scope of the service changes for OpenID Connect (OIDC) support in the 4.4 release.


### API
At a high level, the IAM API actions related management of OIDC providers are implemented along with the STS action for assuming a role with a web identity.


### Related Service Changes
CloudFormation does not support IAM OIDC providers. When AWS adds support we should also implement this functionality.


## Design
This section covers design details for the feature.


### Authentication
IAM API actions use the existing query authentication.

STS is updated to support anonymous requests as required for assuming a role with a web identity where the message contains the credentials (the web id token). The  _TokensQueryPipeline_  now has an _AnonymousRequestHandler_  that is used for the  **_AssumeRoleWithWebIdentity_**  action. The anonymous user has an ephemeral policy permitting any service action to be invoked, this allows the assume role (i.e. resource) policy to later determine if access is permitted.


### Authorization
IAM API actions are authorized in  _Permissions_  as per other IAM actions. This does not support resource specific authorization.

A general purpose  _PolicyEvaluationContext_ is added as a mechanism to provide contextual information required during IAM policy evaluation. This is now used for OIDC information and also in place of the  _ExternalIdKey_ specific context used previously. Resource policies are updated to support federated principals ( _Principal.PrincipalType.Federated_ ) and keys are added for audience ( _OpenIDConnectAudKey_  ) and subject ( _OpenIDConnectSubKey_  )


### Validation
Validation for API requests is performed directly by the service as per other IAM/STS actions.


### Persistence
The following tables are used for persistence of OIDC providers:


* auth_openid_provider : entity
* auth_openid_provider_client_ids : collection table for list property
* auth_openid_provider_thumbprints : collection table for list property

Auxiliary database objects are used to handle creation of delete cascade to the collection tables. An upgrade task  _OpenIdProviderPreUpgrade440_ is added to handle creation of delete cascade constraints on upgrade to 4.4.

The account unique key for an OIDC provider is the "url" which consists of the provider host name and path (excluding any scheme or port)


## Administration
This section covers Eucalyptus specific administrative functionality.


### Quotas
Quotas can be used to limit the number of OIDC providers per account.

The new quota key is "iam:quota-openidconnectprovidernumber".


### Delegate Account
API actions support the usual IAM delegate account functionality that can be used to list the providers in an account and to manage them.


### Account Deletion
Recursive account deletion is updated to delete any identity providers from the account.


### Cloud Properties
The following cloud properties are added for this feature:



| Property | Default Value | Description | 
|  --- |  --- |  --- | 
|  |  |  | 


## API Details

### Actions


| Service | Action | Notes | 
|  --- |  --- |  --- | 
| IAM | AddClientIdToOpenIDConnectProvider |  | 
| IAM | CreateOpenIDConnectProvider |  | 
| IAM | DeleteOpenIDConnectProvider |  | 
| IAM | GetOpenIDConnectProvider |  | 
| IAM | ListOpenIDConnectProviders |  | 
| IAM | RemoveClientIDFromOpenIdConnectProvider |  | 
| IAM | UpdateOpenIDConnectProviderThumbprint |  | 
| IAM | GetAccountSummary | Updated to include number of OIDC identity providers using "Provider" key | 
| STS | AssumeRoleWithWebIdentity | OIDC only, no support for OAuth 2.0 identity providersAdditional policy at assume role time not supported | 


### Condition keys


| Service | Action | Key | Notes | 
|  --- |  --- |  --- |  --- | 
| STS | AssumeRoleWithWebIdentity | $PROVIDER_URL:aud | e.g. auth.globus.org:aud | 
| STS | AssumeRoleWithWebIdentity | $PROVIDER_URL:sub |  | 


## References

* [OAuth 2.0 support (Epic)](https://eucalyptus.atlassian.net/browse/EUCA-12560)
* [[4.4 Identity Federation Architectural Analysis|4.4-Identity-Federation-Architectural-Analysis]]
* [[OIDC Assume Role Notes|OIDC-Assume-Role-Notes]]







*****

[[category.confluence]] 
