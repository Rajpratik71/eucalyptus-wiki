## Overview

Fog is a Ruby library for controlling clouds. It is the underlying library used by popular cloud provisioning modules for both Puppet and Chef.  Fog provides API hooks for many different cloud providers; Eucalyptus uses the API hooks provided for AWS.

## Installation

Fog is easily installed from a single command on any system that has Ruby and Gems installed: 
`gem install fog `

## Configuration and Usage Examples

The AWS provider libraries should work with Eucalyptus without modifications.

A simple example that lists all images:

```
#!/usr/bin/ruby

require 'rubygems'
require 'fog'

# This script is a slightly modified version of the AWS compute sample script provided at:
#   http://fog.io/compute/

# Insert your AWS credentials into a local credentials.rb file. 
# That file should look like this:
#
# @aws_access_key_id = "XXXXXXXXXXXXXXXXXXXXX"
# @aws_secret_access_key = "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
# @ec2_endpoint = "http://your-euca-cloud.com:8773/services/Eucalyptus"

require './credentials.rb'

# Connect to your Eucalyptus Cloud. Note that the only difference is in "endpoint".
connection = Fog::Compute.new(
    :provider => "AWS",
    :aws_access_key_id => @aws_access_key_id,
    :aws_secret_access_key=> @aws_secret_access_key,
    :endpoint => @ec2_endpoint
)

###################################################################
# Get a list of our images
###################################################################
my_images_raw = connection.describe_images()
my_images = my_images_raw.body["imagesSet"]

#  List image ID, architecture and location
for key in 0...my_images.length
  print my_images[key]["imageId"], "\t" , my_images[key]["architecture"] , "\t\t" , my_images[key]["imageLocation"],  "\n";
end
``` 

## Known Issues

Storage *only* works with DNS on.

## Helpful Resources

* A vagrant file configuration for installing a functional Fog client
* A cloud-init library for installing a functional Fog client

## TODOs

* Provide the simplest functional example for storage: listing buckets
* Provide the simplest functional example for launching an image
