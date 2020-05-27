From John Jiang's email to euca-users, 11 March 2013:

*****

Today I looked into the latest version of AWS SDK for PHP, and managed to make it work with ECC. Here is what I did:

(1) Check out the latest version  of AWS SDK for PHP

git clone --depth 1 https://github.com/amazonwebservices/aws-sdk-for-php.git

(2) Make some minor modifications to aws-sdk-for-php/sdk.class.php. There is only one line you need to add to method set_resource_prefix($prefix). The original version of this method was

        public function set_resource_prefix($prefix)
        {
                $this->resource_prefix = $prefix;
                return $this;
        }

After adding one magic line, this method now becomes:

        public function set_resource_prefix($prefix)
        {
                $this->resource_prefix = $prefix;
                $this->hostname .= $this->resource_prefix;
                return $this;
        }

(3) Try to run a VM instance with the following code. It worked.

require_once 'sdk.class.php';

$ec2 = new AmazonEC2(array('key' => 'EC2_ACCESS_KEY', 'secret' => 'EC2_SECRET_KEY',));
$ec2->set_hostname("eucalyptus.ecc.eucalyptus.com", 8773);
$ec2->set_resource_prefix("/services/Eucalyptus");
$ec2->disable_ssl(); // If you need to disable SSL

$response = $ec2->run_instances("emi-07EE3567", 1, 1, array('InstanceType' => 'm1.small', 'KeyName' => 'MyKey',));

if ($response->isOK()) {
    var_dump($response);
} else {
    var_dump($response);
}

(4) Try to terminate a VM instance. It worked.

$response = $ec2->terminate_instances("i-INSTANCE-ID");

(5) Try to describe images. The response from AWS worked great with the following code, but did not work with ECC.

$response = $ec2->describe_images();
if ($response->isOK()) {
    // `body` is a SimpleXMLElement
    foreach ($response->body->imagesSet->item as $item) {
        $image_id = (string) $item->imageId;
        print_r($image_id . "\n");
    }
}

I dumped the response from both AWS and ECC via var_dump($response), and found that the format of them were quite different. I am afraid that this is potentially an incompatible point with AWS.

(6) Try to describe instances. Again, the response from AWS was quite different from the response from ECC in format.

$response = $ec2->describe_instances();

*****

[[category.tools]]
[[category.aws-compatibility]]