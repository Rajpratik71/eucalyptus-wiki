The **Eucalyptus Virtual Cloud** is basically a _CIABIAB_ (Cloud in a box in a box). When you use our CIAB installation from [Silvereye](https://github.com/eucalyptus/silvereye), you get a complete Eucalyptus cloud that can run on a single bare metal machine. With a **Eucalyptus Virtual Cloud** we go one step further. Now you can take that single machine _CIAB_ and install it into a _Virtual Machine_. For something like this to work, the [Hypervisor](https://en.wikipedia.org/wiki/Hypervisor) that is being used on the machine hosting the VM must support [nested virtualization](http://en.wikipedia.org/wiki/Nesting_%28computing%29).

The purpose of this document is to provide the necessary information to set up a **Eucalyptus Virtual Cloud**. It provides the installation and configuration procedures, as well as lists of supported _Hypervisors_ and tools that will be needed to make this work.

*****
[[category.tools]]