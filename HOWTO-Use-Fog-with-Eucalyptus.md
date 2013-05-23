## Overview

Fog is a Ruby library for controlling clouds. It is the underlying library used by popular cloud provisioning modules for both Puppet and Chef.  Fog provides API hooks for many different cloud providers; Eucalyptus uses the API hooks provided for AWS.  Learn more about Fog at http://fog.io.

## Installation

Fog is easily installed from a single command on any system that has Ruby and Gems installed: 
`gem install fog `

## Configuration and Usage Examples

The AWS provider libraries for Fog should work with Eucalyptus without modifications.

A simple script that lists all images on your Eucalyptus cloud:

```
#!/usr/bin/ruby

require 'rubygems'
require 'fog'

# Insert your Eucalyptus credentials into a local credentials.rb file. 
# That file should look like this:
#
# @aws_access_key_id = "XXXXXXXXXXXXXXXXXXXXX"
# @aws_secret_access_key = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
# @ec2_endpoint = "http://your-euca-cloud.com:8773/services/Eucalyptus"

require './credentials.rb'

# Connect to your Eucalyptus Cloud.
connection = Fog::Compute.new(
    :provider => "AWS",
    :aws_access_key_id => @aws_access_key_id,
    :aws_secret_access_key=> @aws_secret_access_key,
    :endpoint => @ec2_endpoint
)

# List images
my_images_raw = connection.describe_images()
my_images = my_images_raw.body["imagesSet"]

# Print formatted list of images
for key in 0...my_images.length
  print my_images[key]["imageId"], "\t" , my_images[key]["architecture"] , "\t\t" , my_images[key]["imageLocation"],  "\n";
end
``` 

## Known Issues

* It should be noted that storage calls will work *only* with Eucalyptus installations that have DNS properly configured; by convention, S3 requires bucket names to contain fully-qualified domain names, and Eucalyptus enforces this convention.

* Fog 1.11.0 contained changes that don't currently work well with Walrus. For users interacting with Walrus, Fog 1.10.0 is currently recommended.  Details of this issue can be found at: https://eucalyptus.atlassian.net/browse/EUCA-6208

## Questions/Issues

If you have any questions about using Fog with Eucalyptus, please first check the [knowledgebase](https://engage.eucalyptus.com/customer/portal/articles/search?q=Fog).  

If you do not find an answer in the knowledgebase, please [ask a question](https://engage.eucalyptus.com/customer/portal/questions/new?q=Fog).

## TODOs

* Provide the simplest functional example for storage: listing buckets
* Provide the simplest functional example for launching an image

***
[[category.HOWTO]]