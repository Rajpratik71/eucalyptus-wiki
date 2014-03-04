# Installation
Follow the instructions here:
https://github.com/aws/aws-cli/blob/develop/README.rst#installation

# Configuration
Follow the instructions here:
https://github.com/aws/aws-cli/blob/develop/README.rst#getting-started

# Setting up a custom region
In order for the AWS CLI to work with Eucalyptus you need to add custom regions to botocore (the underlying AWS api library).

Your desired region must be added in 2 places of the botocore library.
* On MacOS these are installed in a directory similar to:  `/Library/Python/2.7/site-packages/botocore-0.33.0-py2.7.egg/botocore/`
* On Linux it will be in: `/usr/lib/python/site-packages/botocore-0.33.0-py2.7.egg/botocore/`

1. First add a custom region in the `botocore/data/aws/_regions.json` file and add a name for your new region as follows:
`"vic-home": {
        "description": "Vic home cloud"
    },`
2. Add the region and url to each service implementation json in the metadata field as follows. For example in the file `botocore/data/aws/ec2.json` for the EC2 service:

> "metadata": {
>         "regions": {
>             "us-east-1": null,
>             "ap-northeast-1": null,
>             "sa-east-1": null,
>             "ap-southeast-1": null,
>             "ap-southeast-2": null,
>             "us-west-2": null,
>             "us-west-1": null,
>             "eu-west-1": null,
>             "us-gov-west-1": null,
>             **"vic-home":"http://10.0.1.91:8773/services/Eucalyptus",**
>             "cn-north-1": "https://ec2.cn-north-1.amazonaws.com.cn"
>         },
>         "protocols": [
>             "https"
>         ]
>     },