
## Overview
This document describes the preferred usage of the entities API. The entities API has some deprecated functionality, this document details the preferred alternatives for deprecated functionality.


## Preferred API usage

### Transactions
The preferred APIs for transaction management using try-with-resources are:


* #transactionFor
* #distinctTransactionFor
* #readOnlyDistinctTransactionFor

for callbacks with automatic transaction retries:


* #asTransaction
* #asDistinctTransaction

The distinct variants control whether transaction nesting (checked at the call site) is permitted. 

Callbacks must be coded so that retries are handled correctly. This generally means they should have no side effects other than changing database state.

Nesting callbacks within other transactions is often an error as it will break retry functionality.




### Querying entities
Single entity lookup:


```java
UserEntity user = Entities.criteriaQuery( Entities.restriction( UserEntity.class ).equal( UserEntity_.userId, userId ) ).uniqueResult( )
```
Optional entity lookup:


```java
Optional<UserEntity> userOption = Entities.criteriaQuery( UserEntity.class ).whereEqual( UserEntity_.userId, userId ).uniqueResultOption( )
```
Simple listing:


```java
List<UserEntity> users = Entities.criteriaQuery( UserEntity.class ).list( );
```
Listing with restrictions:


```java
Entities.criteriaQuery( 
    Entities.restriction( AccountEntity.class ).like( AccountEntity_.name, accountNameLike ).build( )
).list( )
```
Listing with simple joins:


```java
List<UserEntity> users = Entities
    .criteriaQuery( UserEntity.class )
    .join( UserEntity_.groups ).whereEqual( GroupEntity_.userGroup, Boolean.TRUE )
    .join( GroupEntity_.account ).whereEqual( AccountEntity_.name, this.delegate.getName( ) )
    .list( );
```
Listing with disjunct criteria (OR):


```java
final List<CertificateEntity> entities = Entities.criteriaQuery( CertificateEntity.class ).where(
    Entities.restriction( CertificateEntity.class ).any(
        Entities.restriction( CertificateEntity.class ).isTrue( CertificateEntity_.revoked ).build( ),
        Entities.restriction( CertificateEntity.class ).isNull( CertificateEntity_.certificateHashId ).build( )
    )
).list( );
```

### Counting entities
Count with join and criteria:


```java
final long roles = Entities.count( RoleEntity.class )
    .join( RoleEntity_.account )
    .whereEqual( AccountEntity_.name, accountName )
    .uniqueResult( );
```

### Deleting entities
Entity deletion with criteria:


```java
Entities.delete(
    Entities.restriction( ReservedNameEntity.class ).before( ReservedNameEntity_.expiry, new Date( ) ).build( )
).delete( )
```
this example shows a bulk deletion with simple criteria.

Entity deletion with join:


```java
Entities.delete( UserEntity.class )
    .join( UserEntity_.groups ).whereEqual( GroupEntity_.userGroup, Boolean.TRUE )
    .join( GroupEntity_.account ).whereEqual( AccountEntity_.name, accountName )
    .delete( );
```
this example uses the convenience  _whereEqual_ restriction.


## Deprecated functionality
QBE (query by example) is not part of JPA and we do not currently have a direct replacement for QBE functionality.



| Deprecated | Replacement | Comments | 
|  --- |  --- |  --- | 
| #get | #transactionFor or #distinctTransactionFor | Consider changing to callback style with retries for updates :#asTransaction #asDistinctTransaction | 
| #count(Class) | #count(EntityRestriction) |  | 
| #createCriteria #createCriteriaUnique | #criteriaQuery |  | 
| #query | #criteriaQuery( ... )....list( ) |  | 
| #uniqueResult | #criteriaQuery( ... )....uniqueResult( ) |  | 
| #queryOptions | #criteriaQuery( ... )....readonly( ) |  | 
| deleteAll /deleteAllMatching | #delete( ... )....delete( ) |  | 


## References

* [[Persistence update 4.3|Persistence-update-4.3]]
* [The Java EE Tutorial : Using the Criteria API and Metamodel API to Create Basic Typesafe Queries (docs.oracle.com)](http://docs.oracle.com/javaee/7/tutorial/persistence-criteria003.htm)





*****

[[category.confluence]] 
