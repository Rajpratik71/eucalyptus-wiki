
### Problem Statement
Current (v4.3.0) implementation of the EBS and S3 systems for Eucalyptus are plug-in based and thus require significant configuration to get the system bootstrapped and configured initially.The current process is "register" service, then configure with some ordering dependencies and config changes necessary to get the component to ENABLED status. The design also makes adding active-active components with different configurations/backend problematic because differentiating on a per service basis is a configuration mess.

EBS

Select the provider per cluster. Then ordered config of provider (SANs have a couple of ordering constraints).

euserv-register-service -t storage -H <ip> -p <some partition> <sc_name>

euctl <some partition>.storage.providerclient="3par"

..wait for a bit for 3par to initialize its values...

euctl <some partition>.storage.sanuser=<user>...



OSG:

Must configure which provider to use globally and config any provider specific options.




### Proposed Solution
Rather than registering the generic component and then configuring, the admin should register a specificimplementation–e.g. 3PARController, RiakCS_OSG, Walrus_OSG, etc.

The configuration values are then keyed to the specific component and the dependency between selection and config is relaxed because registration includes selection and the specific implementation config options can be part of the component configuration rather than decoupled and managed independently. That would could config initialization with registration and cleanup the process.

Example flow:

euserv-register-service -t 3parcontroller -H <ip> -p <some partition> <name>

euctl <some partition>.storage.san_host=<ip of 3par>

euctl ...

 **Impact:** docs changes, changes to the ComponentID class and component management systems in Euca directly to support multiple components with same parent type (e.g. storage, or objectstorage). Right now there is no way to differentiate the component implementation from the componentID.

Add "@ComponentType" annotation or as a property to the ComponentID as needed. Must be able to register specific implementations but have msg routing done to any member of that group if using the componentType (e.g. storage, rather than cephcontroller).





*****

[[category.storage]] 
[[category.confluence]] 
