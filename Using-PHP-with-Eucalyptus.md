If you are developing your own Web based tool for Eucalyptus in PHP, you may want to interact with Eucalyptus using an SDK.

## Requirements

To use these tools you'll need:

* A Linux system running a LAMP stack
* A running Eucalyptus cloud, with user credentials (This guide was tested on Eucalyptus 3.1.2)
* git revision control tool installed

## AWS SDK for PHP

The AWS SDK for PHP (http://aws.amazon.com/sdkforphp/) provides a great tool kit for interacting with the Amazon Web Services APIs, including EC2 and S3, which we can re-use in Eucalyptus. 

**The SDK has not been fully validated on Eucalyptus and is not supported by Eucalyptus Systems but it does seem to work without modification, your mileage may vary.**

Steps:

1. Download SDK and ensure you are using version 1.5.3 - this is the last SDK before introduction of the HMAC4 signing code, which Eucalyptus does not yet support:

    ```
    git clone --depth 1 https://github.com/amazonwebservices/aws-sdk-for-php.git /var/www/aws-sdk-for-php
    cd /var/www/aws-sdk-for-php
    git checkout 1.5.3
    ```

2. Create a test page to make sure the SDK works (/var/www/aws-sdk-for-php/eucalyptus-example.php):

     Replace '_your-access-key_', '_your-secret-access-key_' and '_eucalyptus.yourdomain.com_', with your Eucalyptus Access Key, Secret Key and DNS name found within your Eucalyptus 'eucarc' credentials.

    ```
    <?php

    require_once 'sdk.class.php';

    $ec2 = new AmazonEC2(array(
        'key' => 'your-access-key',
        'secret' => 'your-secret-access-key',
    ));

    // http://docs.amazonwebservices.com/AWSSDKforPHP/latest/#m=AmazonEC2/set_hostname
    $ec2->set_hostname("eucalyptus.yourdomain.com", 8773);

    // http://docs.amazonwebservices.com/AWSSDKforPHP/latest/#m=AmazonEC2/set_resource_prefix
    $ec2->set_resource_prefix("/services/Eucalyptus");

    // http://docs.amazonwebservices.com/AWSSDKforPHP/latest/#m=CFRuntime/disable_ssl
    $ec2->disable_ssl(); // If you need to disable SSL

    $response = $ec2->describe_images();

    if ($response->isOK()) {
        // `body` is a SimpleXMLElement
        foreach ($response->body->imagesSet->item as $item) {
            $image_id = (string) $item->imageId;
            print_r($image_id . "\n");
        }
    } else {
        // error--dump the whole request
        print_r($response);
    }
    ```

3. Run the PHP test page via the command-line:

    ```
    php /var/www/aws-sdk-for-php/eucalyptus-example.php
    ```

4. Adapt the code based on examples provided on the AWS site, to use it within your own website code.

### Example: Simple page to launch an instance

This page provides a simple form to launch and instance and uses a few different API queries : https://gist.github.com/3946086

## PHP-AWS

PHP-AWS (https://github.com/tylerhall/php-aws/) is a collection of small PHP scripts that use PHP-Curl to query the Eucalyptus API.


1. Download the scripts:

    ```
    git clone --depth 1 https://github.com/tylerhall/php-aws.git /var/www/php-aws
    cd /var/www/php-aws
    ```

2. Modify class.ec2.php to point to your Eucalyptus server.

    Replace:
    ```
    var $_server     = "http://ec2.amazonaws.com";
    ```

    with:
    ```
    var $_server     = "http://eucalyptus.yourdomain.com:8773/services/Eucalyptus";
    ```

3. Create a test page to ensure the scripts are working (/var/www/php-aws/euca-test.php)

    Replace '_your-access-key_', '_your-secret-access-key_' and '_put-an-emi-id-here', with your credentials and a Eucalyptus Machine Image to run.

    ```
    <?PHP  
        // ec2 sdk code
        require_once("../class.ec2.php");

        // Credentials for ec2 function
        $ec2    = new EC2('your-access-key', 'your-secret-access-key');

        $run = $ec2->runInstances('put-an-emi-id-here', 1, 1, 'key_name');
    ?>
    ```

4. Run the test script via the command line:

    ````
    php /var/www/php-aws/euca-test.php
    ````

5. Some more code samples to describe images and instances, as well as terminate instances:


    ````
    // list all images
    $images = $ec2->getImages();
    foreach ($images as $image_id=>$image)
    {
        $image_location = $image["location"];
        print_r($image_id . ” ” . $image_location . “\n”);
    }

    // start an instance
    $ec2->runInstances(“emi-XXXXXXXX”, 1, 1, “key_name”);

    // list all instances
    $instances = $ec2->getInstances();
    foreach ($instances as $instance_id=>$instance)
    {
        $image_id = $instance["imageId"];
        $state = $instance["state"];
        $dns = $instance["dns"];
        print_r($instance_id . ” ” . $image_id . ” ” . $state . ” ” . $dns . “\n”);
    }

    // terminate an instance
    $ec2->terminateInstances(“i-XXXXXXXX″);
    ````


*****

[[category.docs]]