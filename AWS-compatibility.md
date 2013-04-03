Eucalyptus strives for 100% API compatibility with all AWS services that we currently support.  Our goal is to be a "drop-in replacement AWS region", or as close to that as humanly possible.

To that end, we want to test against all AWS tools in the open source ecosystem, identify those places where we are compatible, and find and fix all places where we are not compatible. All issues should be filed against the [AWS Compatibility component](https://eucalyptus.atlassian.net/browse/EUCA/component/10201).

## <font color="gold">Golden: Just Works</font>

Golden means a service "just works". Pointing region and credential variables of the service to a properly-configured Eucalyptus results in a near-perfectly working service.

* _**Vagrant AWS Plug-in**_
 * Project URL: https://github.com/mitchellh/vagrant-aws
 * Known bugs: None
 * Notes: On 16 Mar 2013, Github user "carlsverre" submitted a [https://github.com/mitchellh/vagrant-aws/pull/6](patch) to the new Vagrant AWS plug-in to allow it to work against EMIs as well as AMIs.  On 21 Mar 2013, Github user "svanzoest" +1'd it.  The patch was merged by Github user "mitchellh" (upstream) on 22 Mar 2013.
 * Last update: 27 Mar 2013

* _**s3curl**_
 * Project URL: http://aws.amazon.com/code/128
 * Known bugs: None
 * Notes: The most basic tool for talking to Walrus. For more info, see: http://www.eucalyptus.com/eucalyptus-cloud/tools/s3curl
 * Last update: 29 May 2013

* _**s3cmd**_
 * Project URL: http://s3tools.org/s3cmd and https://github.com/s3tools/s3cmd
 * Known bugs: None (with latest 1.5.0 build)
 * Notes: Recommended tool for talking to Walrus. For more info, see: 
http://www.eucalyptus.com/eucalyptus-cloud/tools/s3cmd
 * Last update: 02 April 2013

## <font background="black" color="green">Green: Works with Tweaks</font>

Green means a service works well enough to be useful, though code tweaks are necessary and fully equivalent functionality may not yet be available.  Problems with tools in the "green" state are considered bugs, and will be handled accordingly.

* _**AWS Toolkit for Eclipse**_
 * Project URL: https://github.com/aws/aws-toolkit-eclipse
 * Driver: **[[Colby Dyess]]**
 * Known bugs: FIXME
 * Notes: EC2 functionality mostly "just works", some known issues around S3 functionality. More details here: [[Eclipse Plugin]]
 * Last update: 12 Mar 2013

* _**AWS SDK for PHP**_
 * Project URL: FIXME
 * Driver: **John Jiang**
 * Known bugs: FIXME
 * Notes: [[AWS SDK for PHP]]
 * Last update: 11 Mar 2013

* _**AWS SDK for Ruby**_ 
 * Project URL: https://github.com/aws/aws-sdk-ruby
 * Driver: **jeevan_ullas**
 * Known bugs: FIXME
 * Notes: FIXME
 * Last update: none

* _**AWS SDK for Java**_ 
 * Project URL: http://aws.amazon.com/sdkforjava/
 * Driver: **Tony Beckham**
 * Known bugs: FIXME
 * Notes: known issues with Walrus should be fixed as of 3.3m4, needs retesting
 * Last update: 11 Mar 2013

* _**Fog**_
 * Project URL: http://fog.io
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
  <tr><td>Stackato</td><td>&nbsp;</td><td>[[Harold Spencer Jr.]]</td><td><font color="green">Green</font></td><td>Image runs, but the Stackato services do not yet work. More info: [[Stackato-Image]]</td></tr>
  <tr><td>AWS .Net SDK</td><td>&nbsp;</td><td>open</td><td>&nbsp;</td><td>&nbsp;</td></tr>
  <tr><td>AWS SDK for Android</td><td>&nbsp;</td><td>open</td><td>&nbsp;</td><td>&nbsp;</td></tr>
  <tr><td>AWS toolkit for iOS</td><td>&nbsp;</td><td>open</td><td>&nbsp;</td><td>&nbsp;</td></tr>
  <tr><td>JetS3t</td><td>&nbsp;</td><td>open</td><td>&nbsp;</td><td>&nbsp;</td></tr>
  <tr><td>RightScale AWS Gems</td><td>&nbsp;</td><td>open</td><td>&nbsp;</td><td>&nbsp;</td></tr>
</table>

*****

[[category.aws-compatibility]]