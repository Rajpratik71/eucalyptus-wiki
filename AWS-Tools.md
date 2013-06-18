Eucalyptus strives for 100% API compatibility with all currently supported AWS services. Our goal is to be a "drop-in replacement AWS region", or as close to that as humanly possible.  We support the following open source AWS tools.

## Recommended tools

**[s3cmd](https://github.com/eucalyptus/eucalyptus/wiki/HowTo-use-s3cmd-with-Eucalyptus)**.  s3cmd is the preferred command-line tool for working with Eucalyptus Walrus and AWS S3.  

## Other tools

**[http://www.eucalyptus.com/eucalyptus-cloud/tools/s3curl](s3curl)**. S3 Curl is a tool that allows users to interact with Eucalyptus Walrus and AWS S3.

**[https://github.com/mitchellh/vagrant-aws](Vagrant AWS Plug-In)**.  Vagrant is a tool for managing virtual machines on a local system, and also provides a plug-in that allows users to manage their AWS instances with Vagrant configuration files.  The plug-in also works for Eucalyptus.

*****

* _**EucaLobo**_
 * Project URL: https://github.com/viglesiasce/EucaLobo
 * Versions tested: master
 * Driver: **[[Vic Iglesias]]**
 * Known bugs: https://github.com/viglesiasce/EucaLobo/issues
 * Notes: https://github.com/viglesiasce/EucaLobo
 * Last update: 9 April 2013

* _**Cyberduck**_
 * Project URL: http://cyberduck.ch/
 * Versions tested: master
 * Driver: **[[Tony Beckham]]**
 * Known bugs: None
 * Notes: Show hidden files preference allows to see versioned files
 * Last update: 9 April 2013

## <font background="black" color="green">Green: Works with Tweaks</font>

Green means a service works well enough to be useful, though code tweaks are necessary and fully equivalent functionality may not yet be available.  Problems with tools in the "green" state are considered bugs, and will be handled accordingly.

* _**AWS Toolkit for Eclipse**_
 * Project URL: https://github.com/aws/aws-toolkit-eclipse
 * Versions tested:  Eclipse: Juno Service Release 1 --  AWS Java SDK: 1.4.0 -- Eucalyptus: 3.3 Milestone 3
 * Driver: **[[Colby Dyess]]**
 * Known bugs: ECUA-5275
 * Notes: EC2 functionality mostly "just works", some known issues around S3 functionality. More details here: [[AWS Toolkit for Eclipse]]
 * Last update: 12 Mar 2013


* _**AWS SDK for PHP**_
 * Project URL: FIXME
 * Versions tested: FIXME
 * Driver: **John Jiang**
 * Known bugs: FIXME
 * Notes: [[AWS SDK for PHP]]
 * Last update: 11 Mar 2013

* _**AWS SDK for Ruby**_ 
 * Project URL: https://github.com/aws/aws-sdk-ruby
 * Versions tested: FIXME
 * Driver: **jeevan_ullas**
 * Known bugs: FIXME
 * Notes: FIXME
 * Last update: none

* _**AWS SDK for Java**_ 
 * Project URL: http://aws.amazon.com/sdkforjava/
 * Versions tested: [AWS JAVA SDK 1.4.1, Euca 3.3 m5]
 * Testplan: https://github.com/eucalyptus/eucalyptus/wiki/Aws-SDK-for-Java-Testplan
 * Driver: **[[Tony Beckham]]**
 * Known bugs: [EUCA-5515](https://eucalyptus.atlassian.net/browse/EUCA-5515) but reported fixed in 3.3 testing branch
 * Notes: known issues with Walrus should be fixed as of 3.3m4, needs retesting
 * Last update: 09 April 2013

* _**Fog**_
 * Project URL: http://fog.io
 * Versions tested: FIXME
 * Driver: **[[Harold Spencer Jr.]]**
 * Known bugs: FIXME
 * Notes: [[Fog]]. **Notable: Fog with Walrus requires Euca DNS to be set up?**
 * Last update: 16 Mar 2013

* _**NetflixOSS Edda**_
 * Project URL: FIXME
 * Driver: Dan Nurmi, [[grze]]
 * Known bugs: FIXME
 * Notes: Demoed at Netflix, 13 Mar 2013. Blog post describing setup: http://nurmiblog.wordpress.com/2013/01/22/inspired-by-netflix/
 * Last update: 13 Mar 2013

* _**Netflix Asgard**_
 * Project URL: FIXME
 * Driver: [[grze]]
 * Known bugs: FIXME
 * Notes: demoed at Netflix, 13 Mar 2013.
 * Last update: 13 Mar 2013

* _**Netflix Chaos Monkey**_
 * Project URL: FIXME
 * Driver: [[grze]]
 * Known bugs: FIXME
 * Notes: demoed at Netflix, 13 Mar 2013.
 * Last update: 13 Mar 2013

* _**AWS CLI**_
 * Project URL: http://aws.amazon.com/cli/
 * Driver: Andy Grimm
 * Known bugs: FIXME
 * Notes: http://agrimmsreality.blogspot.com/2013/01/using-aws-cli-with-eucalyptus.html
 * Last update: 3 Jan 2013

* _**Jenkins for EC2**_
 * Project URL: https://wiki.jenkins-ci.org/display/JENKINS/Amazon+EC2+Plugin
 * Driver: [[Lester Wade]]
 * Known bugs: FIXME
 * Notes: Version 1.17 of the EC2 plugin does not work. Jenkins tries to get endpoints via the register URL which fails unless endpoint is https://cloudIP:8443/register. Once past this point, connectivity test yields "login failure: all modules ignored". Version 1.14 works however.
 * Last update: FIXME

## <font color="red">Red: Not working / Unknown</font>

Red means that a service cannot be made to work without substantial modification, or at all -- or that status is completely unknown.

*****

## Other Projects

For reference and later testing.

<table>
  <tr><td>AppScale</td><td>&nbsp;</td><td>[[shaon]]</td><td><font color="green">Green</font></td><td>http://mdshaonimran.wordpress.com/2013/03/01/run-appscale-on-eucalyptus/</td></tr>
  <tr><td>Scalr</td><td>&nbsp;</td><td>[[Vic Iglesias]]</td><td><font color="green">Green</font></td><td>http://testingclouds.wordpress.com/2013/01/23/using-scalr-for-automation-of-your-eucalyptus-cloud/</td></tr>
  <tr><td>Stackato</td><td>&nbsp;</td><td>[[Harold Spencer Jr.]]</td><td><font color="green">Green</font></td><td>Image runs, and Stackato services work. More info: [[Stackato-Image]]</td></tr>
  <tr><td>AWS .Net SDK</td><td>&nbsp;</td><td>open</td><td>&nbsp;</td><td>&nbsp;</td></tr>
  <tr><td>AWS SDK for Android</td><td>&nbsp;</td><td>open</td><td>&nbsp;</td><td>&nbsp;</td></tr>
  <tr><td>AWS toolkit for iOS</td><td>&nbsp;</td><td>open</td><td>&nbsp;</td><td>&nbsp;</td></tr>
  <tr><td>JetS3t</td><td>&nbsp;</td><td>open</td><td>&nbsp;</td><td>&nbsp;</td></tr>
  <tr><td>RightScale AWS Gems</td><td>&nbsp;</td><td>open</td><td>&nbsp;</td><td>&nbsp;</td></tr>
</table>

*****

[[category.aws-compatibility]]