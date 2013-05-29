## Introduction

The AWS Ruby SDK is Amazon's official Ruby library for AWS services. The Ruby SDK works with Eucalyptus as well, with some minor patches, for those services that Eucalyptus supports.  For more information on using the AWS Ruby SDK, see the [official site](http://aws.amazon.com/sdkforruby/).

## Installation

The following installation instructions should work for any Linux distribution.

1. Install the AWS Ruby SDK itself.  Eucalyptus supports version 1.8.5, which can be installed as follows: 'gem install aws-sdk -v 1.8.5' 
1. Verify that the right version of the gem is installed by running 'gem list --local'.  
1. Retrieve the patch from Eucalyptus. 'wget https://raw.github.com/eucalyptus/Eucalyptus-Scripts/master/AWS-Ruby-SDK-Patches/euca-aws-ruby-sdk.1.8.5.patch -O /tmp/euca-aws-ruby-sdk.1.8.5.patch'
1. Apply the patch. Modify the path in the following command to the location of the ruby gems on your own system.  'cd /usr/lib/ruby/gems/1.8/gems/aws-sdk-1.8.5 && patch -p1 < /tmp/euca-aws-ruby-sdk.1.8.5.patch'

The resulting patched Ruby gem should work for both Eucalyptus and AWS. 

## Configuration and Usage

FIXME

## Known Issues

FIXME

## Questions/Issues

If you have any questions about using Fog with Eucalyptus, please first check the [knowledgebase](https://engage.eucalyptus.com/customer/portal/articles/search?q=Ruby).  

If you do not find an answer in the knowledgebase, please [ask a question](https://engage.eucalyptus.com/customer/portal/questions/new?q=Ruby).

***
[[category.HOWTO]]