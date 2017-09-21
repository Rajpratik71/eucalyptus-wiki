* [Overview](#overview)
* [Implementation](#implementation)
  * [Configuration](#configuration)
  * [Threading](#threading)
  * [Authorization / Request Context Propagation](#authorization-/-request-context-propagation)
  * [Service Endpoints](#service-endpoints)
    * [Method Lookup](#method-lookup)
* [References](#references)



# Overview
This document covers the design and implementation details for replacing mule with spring integration in the 4.4 release.


# Implementation

## Configuration
There is a main configuration file for spring integration. Individual beans can be identified by (pre 4.4) component scanning for classes annotated with  _@ComponentNamed_ (and optionally  _@Inject_ )

Each component can provide a  _$COMPONENT$-component-context.xml_  spring bean configuration; these component specific configurations are imported into the main configuration.


## Threading
Threading is handled from  _ServiceContext_ which now has the following configuration properties:


*  **_bootstrap.servicebus.common_thread_pool_size_**  = 256
*  **_bootstrap.servicebus.component_thread_pool_size_**  = 64
*  **_bootstrap.servicebus.dispatch_thread_pool_size_**  = 256

with default values as shown.

There is a common (shared) thread pool that is used for dispatch and all components (services) by default. To use separate thread pools for dispatching and for each component the common thread pool size can be set to zero. The dispatch/component thread pool sizes are not used unless the common thread pool is disabled (size set to zero)


## Authorization / Request Context Propagation
The current  _Context_ information is propagated within the spring integration framework using the  _ThreadStatePropagationChannelInterceptor_ functionality.

We propagate the context within an  _Optional_ so that the absence of a context can also be propagated. The  _ContextPropagationChannelInterceptor_ is responsible for context propagation and is registered in the main XML configuration for spring integration.


## Service Endpoints
The configuration for each component typically defines the  _@ServiceActivator_ (s) (service implementation(s)) and any additional channels used for the component. A simple configuration example is:


```xml
<beans ...>
  <int:service-activator input-channel="autoscaling-request" ref="autoScalingService"/>
</beans>
```
 Default error handling behaviour can be used (as per above) or the component can have a custom error handler:


```xml
<beans ...>
  <int:channel id="autoscaling-error"/>

  <int:chain id="autoscaling-request-chain" input-channel="autoscaling-request">
    <int:header-enricher>
      <int:error-channel ref="autoscaling-error"/>
    </int:header-enricher>
    <int:service-activator ref="autoScalingService"/>
  </int:chain>

  <int:service-activator input-channel="autoscaling-error" ref="autoScalingErrorHandler"/>
</beans>
```
In this example the  **_autoScalingService_** and  **_autoScalingErrorHandler_** would be located by component scanning (annotated with  _@ComponentNamed_ )


### Method Lookup
Service activators determine the target method on service beans by parameter type. Some beans define public methods that are not related to the service implementation, such as lifecycle methods. When there are duplicate methods on service beans the configuration is invalid.

Solutions for duplicate methods are:


* Annotate the service implementation methods with  _@ServiceActivator_ 
* Use a service interface that has only the implementation methods
* Relocate methods not related to service implementation

Using  _@ServiceActivator_  annotations is simple but means details of spring integration leaks into service beans.

An example of using a service interface is shown below, this may be the best approach when there is an existing service interface as it ensures that only methods defined on the interface can be used:


```xml
<beans ...>
  <int:service-activator input-channel="objectstorage-request">
    <bean class="org.springframework.aop.framework.ProxyFactoryBean">
      <property name="targetSource">
        <bean class="org.springframework.aop.target.SimpleBeanTargetSource">
          <property name="targetClass" value="com.eucalyptus.objectstorage.ObjectStorageService"/>
          <property name="targetBeanName" value="objectStorageGateway"/>
        </bean>
      </property>
    </bean>
  </int:service-activator>
</beans>
```
The  _BlockStorageController_ and  _WalrusControl_ are updated to use the new  _BlockStorageService_ and _WalrusService_ interfaces respectively.


# References

* [[4.4 Message bus architectural analysis|4.4-Message-Bus-Architectural-Analysis]]
* [Spring integration reference manual (docs.spring.io)](https://docs.spring.io/spring-integration/docs/4.3.1.RELEASE/reference/htmlsingle/)



*****

[[category.confluence]] 
