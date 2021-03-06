Some project ideas for GSoC (or implementation in general).  Eucalyptians, please free free to add to the list!

*Recipes.* The goal of the Recipes project is to produce pre-built "recipes" that make it trivial for any Eucalyptus user to create and configure their own application stack.  Code lives at: http://github.com/eucalyptus/recipes

*EucaUI.* There are many ways to interact with a Eucalyptus cloud via the EC2 and S3 APIs, such as using Euca2ools, the Boto library, HybridFox, and others. However, having an basic GUI within eucalyptus may help users to get started. A HTML5 / Javascript UI that can run in a web browser would be helpful for many types of users. _(Skills: HTML5 / Javascript / Ajax)_

*World Map.* Investigate and implement the best method for reporting on Eucalyptus installations worldwide (including generic info such as country, size, and version). To include a means for visualizing this information, for example a world map or a running table showing specific stats, such as latest installation, largest installation of the month, etc. +(Skills: scripts / Java / web presentation)+

*EucaDroid.* Android is an increasingly important open-source user-facing graphical operating system, running in smart phones, in handheld and wallmounted devices, and on tablet computers. It would be useful to have a well written Android application that can manage and control storage and compute resources on AWS and in Eucalyptus clouds. _(Skills: Android development)_

*Cloud Desktop.* This project targets the development of a Web UI to display remote desktops of VMs running in a Eucalyptus cloud. Using this Web UI, administrators can debug their VMs or extend the interface into a “Desktop in the Cloud.” HTML5/Javascript are expected to implement this feature (to avoid cumbersome Flash or Java applet). Remoting technology can be RDP, VNC or SPICE (if there is enough time). _(Skills: HTML5 / Javascript / VNC / RDP / SPICE)_

*Euca2ools Challenge.* The Boto library and Euca2ools are open-source libraries and command line tools that use the published AWS API to interact with Eucalyptus, AWS, and OpenStack resources. A comprehensive framework to do correctness and conformance testing of the tools against the published API is the goal of this project. The AWS API is ever expanding, so the framework must be able to handle multiple versions of the API. _(Skills: Python)_

*Euca2ools NG.* The Boto library and the Euca2ools scripts are written in Python 2, and work well in the Python interpreter up to version 2.7.1. This project will focus on porting Euca2tools and Boto to Python 3.x interpreters. _(Skills: Python 3.x)_ 

*RoboUI.* The roboto project uses JSON data structures to fully describe the requests and responses of HTTP-based distributed systems like Eucalyptus. We then use those JSON descriptions to automatically generate command line interfaces for new services. It is believed that these JSON descriptions could also be useful in helping to automatically generate user interfaces for these services. This project would explore these possibilities with a goal of auto-generating admin-level web user interfaces for a subset of Eucalyptus services. _(Skills: JSON / Java / Javascript)_

*Cluster GPU.* GPUs can now be used to accelerate the performance of many general purpose computing problems. The goal of this project is to explore and implement a mechanism for exposing the GPUs present on the system to the instances. An application needs to be identified that showcases implementation. _(Skills: libvirt / kvm - xen / scripting language)_

*DropBukkit.* Working with many customers, partners and contractors means we often need to share small/medium/large files for various reasons.  Sometimes we use sftp, sometimes we use DropBox, sometimes we use other popular tools.  It would be great if we could work on a cool client for our own Walrus, to be able to provide users access to a bukkit for file sharing purposes.  It would be great to have clients for all platforms (including mobile devices). _(Skills: Android development; Java. Native; perhaps Python + Qt)_

**********

(original source: https://projects.eucalyptus.com/redmine/projects/gsoc2012/)
[[category.gsoc]]
