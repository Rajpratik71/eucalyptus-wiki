## Introduction

The AWS Ruby SDK is Amazon's official Ruby library for AWS services. The Ruby SDK works with Eucalyptus as well, with some minor patches, for those services that Eucalyptus supports.  For more information on using the AWS Ruby SDK, see the [official site](http://aws.amazon.com/sdkforruby/).

## Installation

The following command will install a patched version of the AWS Ruby SDK, which supports both AWS and Eucalyptus style of endpoint addressing: 'gem install aws-sdk-euca'

## Configuration and Usage

A very simple S3-style script that lists all Walrus buckets:

'''
require 'aws-sdk'
 
storage_walrus = AWS::S3.new({
	:access_key_id => 'XXXXXXXXXXXXXXXXXXXXX',
	:secret_access_key => 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',
	:s3_endpoint => 'http://your-eucalyptus-server.com',
	:s3_port => 8773,
	:s3_service_path => '/services/Walrus',
	:use_ssl => false,
	:s3_force_path_style => false,
	:ssl_verify_peer => false,
	:http_wire_trace => true
})
 
storage_walrus.buckets.map(&:name)
'''

And a very simple EC2-style script that shows available images:
'''
require 'aws-sdk'
 
conn = AWS::EC2.new({
	:access_key_id => 'XXXXXXXXXXXXXXXXXXXXX',
	:secret_access_key => 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',
	:ec2_endpoint => 'http://eucalyptus.your-eucalyptus-server.com'
        :ec2_service_path => '/services/Eucalyptus/',
        :ec2_port => 8773,
        :ssl_verify_peer => false,
        :use_ssl => false,
        :http_wire_trace => true
})

conn.images.map(&:name)
'''

## Known Issues

FIXME

## Questions/Issues

If you have any questions about using Fog with Eucalyptus, please first check the [knowledgebase](https://engage.eucalyptus.com/customer/portal/articles/search?q=Ruby).  

If you do not find an answer in the knowledgebase, please [ask a question](https://engage.eucalyptus.com/customer/portal/questions/new?q=Ruby).

***
[[category.HOWTO]]