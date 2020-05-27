# Monitoring Eucalyptus IaaS Availability

This article describes one way to monitor your Eucalyptus cloud with Eutester, a framework for testing clouds and Nagios, a popular open source infrastructure monitoring tool.

We use a simple instance testcase from Eutester to spin up an instance in Eucalyptus with a custom ssh key and security group, ping the instance, ssh into it and then terminate the instance. We then report to Nagios on the overall success of the test.

This test simulates a simple Cloud User's interaction with Eucalyptus to ensure the service is working for users.

## System Requirements
* CentOS 6
* NTP running and syncing time correctly
* EPEL repository

## Install some helpful tools
    yum -y install git unzip euca2ools

## Install Nagios

Nagios-core and lots of Nagios plugins are part of the EPEL repository.

### Install packages
    yum install -y nagios nagios-plugins nagios-plugins-nrpe nagios-plugins-all

### Set nagiosadmin htaccess access password
    htpasswd /etc/nagios/passwd nagiosadmin

### Turn on services at boot start them
    chkconfig nagios on
    chkconfig httpd on
    service nagios start
    service httpd start

### Test Nagios

Open your browser to test the webui is working, enter the user 'nagiosadmin' and the password you set above.
    
    http://<your-nagios-server>/nagios/

More information on configuring Nagios can be found on the Nagios website:

    http://go.nagios.com/nagioscore/docs


## Create a Eucalyptus account for Eutester

This is useful for tracking eutester usage with Eucalyptus' reporting and makes it possible to restrict its resources using IAM policies.

Run the following commands from an environment where your Eucalyptus admin credentials are sourced.

### Create Account
    euare-accountcreate -a eutester

### Create key
    euare-useraddkey --delegate=eutester -u admin

### Changing pwd
    euare-useraddloginprofile -u admin -p [PASSWORD] --delegate=eutester

### Generate Certificate
    euare-usercreatecert -u admin --delegate=eutester

### Download credentials for admin user in eutester account
    euca-get-credentials -a eutester -u admin eutester-admin.zip

### Copy the zipfile to your Nagios server
    scp eutester-admin.zip root@<your-nagios-server>:~

### Login to your nagios server and source the credentials for use with Eucalyptus
    ssh root@<your-nagios-server>
    unzip eutester-admin.zip -d ~/.eucarc
    source ~/.eucarc/eucarc

### To ensure your credentials are working, try a euca-describe-images command
    euca-describe-images

_This command should list the available images in your cloud_

## Install Eutester

### Clone the latest eutester code from Github
    cd /root && git clone --depth 1 git://github.com/eucalyptus/eutester.git && cd eutester
    python setup.py install

### Install some python dependancies
    yum -y install python-argparse python-paramiko

### Download Custom testcase for Eutester

This testcase is a modified version of instancetest.py and is the [process](https://github.com/eucalyptus/eutester/pull/99) of being submitted to Eutester upstream.
For the meantime, let's just remove the upstream version and replace it with the custom version that can output in the format we require for Nagios.

    rm -f /root/eutester/testcases/cloud_user/instances/instancetest.py
    wget -c https://raw.github.com/tomellis/eutester/master/testcases/cloud_user/instances/instancetest.py -O /root/eutester/testcases/cloud_user/instances/instancetest.py

## Install Nagios plugin and helper scripts

### Download Nagios eutester check scripts
    cd /root && git clone --depth 1 git://github.com/monolive/nagios-eucalyptus.git

### Copy Nagios plugin to plugins dir
    cp nagios-eucalyptus/nagios/check_eutester_test.sh /usr/lib64/nagios/plugins/

### Add script to run Eutester every hour to cron

This script requires a Eucalyptus Machine Image ID of a usable Linux instance in your cloud that it can spin up, ping and ssh to. It also requires the path to your Eucalyptus credentials.
If you followed the example from above, you shouldn't need to modify anything except for supplying a valid Linux based EMI.

**Edit the script and update the EMI to a valid EMI** 

    vim /root/nagios-eucalyptus/nagios/run_eutester_testcase.sh

On the line with **EMI=**, add a valid EMI from your cloud. See _euca-describe-images_ for a list of EMI's.

**Link the script to /etc/cron.hourly - This will run the script once an hour**

    ln -s /root/nagios-eucalyptus/nagios/run_eutester_testcase.sh /etc/cron.hourly/

### Test the script

This should pass the test and the results file should show PASS.

    /root/nagios-eucalyptus/nagios/run_eutester_testcase.sh
    cat /tmp/results/test-BasicInstanceChecks-result.txt

### Add the Nagios Check

Finally, we need to add our Nagios check which will look at the Eutester result and throw a critical alert if the test failed, a warning if the results file does not exist.

This nagios check has a simple command and service check that is performed from the local nagios server.

Append the following to /etc/nagios/objects/commands.cfg:
    ```
    # Eutester check command
    define command {
            command_name    check_eutester_test
            command_line    $USER1$/check_eutester_test.sh
            }
    ```
Append the following to /etc/nagios/objects/localhost.cfg:
    ```
    # Define Eutester service check
    define service{
            use                             local-service         ; Name of service template to use
            host_name                       localhost
            service_description             Eutester Basic Instance Test
            check_command                   check_eutester_test
            notification_interval           0
            }
    ```
### Restart Nagios
    /etc/init.d/nagios restart

### Done!

Now go checkout the Nagios WebUI to see the status of the check:
    ```
    http://<your-nagios-server>/nagios/
    ```

*****

[[category.monitoring]]
