Eucalyptus has a tool to generate the wsdl files from its APIs in the source tree. To run it, you must have the code checked out and built. See the [INSTALL](https://github.com/eucalyptus/eucalyptus/blob/master/INSTALL) and [README](https://github.com/eucalyptus/eucalyptus/blob/master/README) files for instructions on building Eucalyptus.

To generate the API WSDLS, from the source directory where you cloned the source, run:

`./devel/groovy.sh ./clc/modules/msgs/src/main/java/generate-xsd.groovy`

The output will be a set of WSDL files in the current working directory. These include services that implement AWS APIs as well as internal and administrative service APIs.


*****

[[category.developer]]