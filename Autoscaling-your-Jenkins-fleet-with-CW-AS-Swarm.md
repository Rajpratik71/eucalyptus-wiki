### Prerequisites
1. Jenkins installed
2. [https://wiki.jenkins-ci.org/display/JENKINS/Swarm+Plugin](Swarm Plugin) installed in Jenkins 
3. Eucalyptus cloud running 3.3.0

### Setting up Swarm Client user-data
In order to use Swarm the only requirement is that your slave instances run the Swarm client Java application.

1. Download [the Swarm client](http://maven.jenkins-ci.org/content/repositories/releases/org/jenkins-ci/plugins/swarm-client/1.9/swarm-client-1.9-jar-with-dependencies.jar)
2. Upload the client jar to a bucket in Walrus with Public Read ACL. 
3. Change the JENKINS_MASTER_URL and SWARM_CLIENT_URL in the following user-data or create your own userdata that installs JRE and then runs the client JAR:
```
Content-Type: multipart/mixed; boundary="===============5423618256409275201=="
MIME-Version: 1.0

--===============5423618256409275201==
Content-Type: text/x-shellscript; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="setup-sabnzbd.sh"

#!/bin/bash
JENKINS_MASTER_URL=http://10.0.1.61:8080
SWARM_CLIENT_URL=http://mycloud:8773/services/Walrus/jenkins-plugins/swarm-client-1.9-jar-with-dependencies.jar
instance_id=`curl http://169.254.169.254/latest/meta-data/instance-id`
instance_ip=`curl http://169.254.169.254/latest/meta-data/local-ipv4`

ntpdate -u pool.ntp.org

wget $SWARM_CLIENT_URL -O /root/swarm-client-1.9-jar-with-dependencies.jar
java -jar /root/swarm-client-1.9-jar-with-dependencies.jar -master $JENKINS_MASTER_URL -name $instance_id & > /root/swarm-client.log


--===============5423618256409275201==
Content-Type: text/cloud-config; charset="us-ascii"
MIME-Version: 1.0
Content-Transfer-Encoding: 7bit
Content-Disposition: attachment; filename="cloud-config.txt"

#cloud-config

disable_root: false
byobu_by_default: system
resize_rootfs: True

packages:
 - ntp
 - openjdk-7-jre

--===============5423618256409275201==--
```

### Creating Eucalyptus Autoscaling and CloudWatch resources

#### Autoscaling Config
1. Create a launch configuration that uses your desired base image, type, keypair etc and uses the user data file you created above
2. Create an autoscaling group that uses the launch config from step 1 and set the desired capacity to the initial number of executors you would like to use.
3. After step 2 you should begin to see instances launching in your cloud and subsequently registering themselves with your Jenkins Master
4. Create a scaling policy for the newly created ASG that has a ChangeInCapacity of 1

#### Monitoring Jenkins
1. In order to monitor your Jenkins free executors (the scaling parameter used in this case) you will need to run the following Eutester script on your Jenkins Master: https://gist.github.com/viglesiasce/6012546
2. The run parameters for the script are as follows:
```
monitor-jenkins.py --credpath /path/to/your/creds 
```
3. Once the script is running you should see output every 10 or so seconds saying that it has entered CloudWatch data and which value it has entered.
4. Check your CloudWatch metrics to ensure that they are being entered correctly into the "Jenkins" namespace and with the metric name "Free Executors"
#### Setting up CloudWatch Alarms
1. Create an Alarm that has the following parameters:
```
AlarmName = <arbitrary name for alarm>
NameSpace = "Jenkins"
MetricName = "Free Executors"
Period = 60
EvaluationPeriods = 5 or greater
ComparisonOperator = LessThanOrEqualToThreshold
Statistic = Average
Threshold = <initial autoscaling group desired capacity>
AlarmActions = <ARN from scaling policy that was created>
```