# What is SoapUI?

SoapUI is a tool for interacting with
[SOAP](http://en.wikipedia.org/wiki/SOAP) (Simple Object Access
Protocol) APIs. It allows the user to see the raw XML used in both the
request and response of a SOAP query. This can be useful for lots of
different reasons. In the case of Eucalyptus it's most helpful for
testing APIs for compatibility with AWS.

# Prerequisites

You'll need to install the SoapUI package, which is available for
download
[here](http://sourceforge.net/projects/soapui/files/soapui/4.5.2/). You
should also have a Eucalyptus cloud up and running to interact with.

In order to run SoapUI, you'll also need to have Java installed. Java
also includes the _keytool_ binary that is used to create the
_keystore_. To install Java on CentOS, RHEL or Fedora, run the
following command as the _root_ user:

    $ yum install java-1.7.0-openjdk

# Creating a Keystore

The first thing to do before starting SoapUI is to create a
_keystore_. Your _keystore_ is an archive containing a _keypair_
(certificate and private key) for a Eucalyptus (or AWS) user.

If you already have a _keypair_, you can create a _keystore_ by
running the following commands:

    $ openssl pkcs12 -export -in myuser.crt -inkey myuser.pkey > keypair.p12
    $ keytool -importkeystore -srckeystore keypair.p12 -destkeystore mykeystore.jks -srcstoretype pkcs12

For both commands you'll be prompted to enter a password. Once for the
pkcs12 file you create, and then again for the _keystore_ itself. You
can also use [this](https://gist.github.com/mspaulding06/5667759)
script which will do it all for you. It will automatically enter a
password which defaults to _foobar_.

# Setting up soapUI

Start the soapUI application.

## Importing the API WSDL

Once you've got it open, click **File** and select **New soapUI
Project**. Give your project a name like _EC2 API_ and then paste a
link to the WSDL you would like to use (in our case, an AWS WSDL for
EC2). Your dialog should look something like the one below:

![Project Dialog](https://raw.github.com/mspaulding06/wiki-images/master/soapui/soapui_config_project.png)

## Adding the Keystore

Right-click on the project name in the left panel and a window for the
project should appear. In the new window, click the tab called
**WS-Security Configurations** and then the tab **Keystores**. Click the
plus sign icon and then browse for the keystore file you've
created. It will prompt you for the password that you chose when the
_keystore_ was created.

![Keystore Dialog](https://raw.github.com/mspaulding06/wiki-images/master/soapui/soapui_config_keystore.png)

## Configuring Outgoing Security

Now we need to configure what security information will be included in
the SOAP requests that are sent. Click on the tab that says
**Outgoing WS-Security Configurations**. Click the pluse sign right under
the tab. The tooltip should say **Adds a new Outgoing WSS Configuration**.
Give the configurationa name, like _eucalyptus_. Then click the other plus
sign below the first one. The tooltip should say **Adds a new WSS Entry**.
Select **Timestamp** and set the _Time to Live_ to **300**.

Click the same plus sign again and select **Signature**, filling in the fields as below:

![Outgoing Security Configuration](https://raw.github.com/mspaulding06/wiki-images/master/soapui/soapui_config_outgoing.png)

The _Namespace_ for _Timestamp_ should be:

http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd

and the _Namespace_ for _Body_ should be:

http://schemas.xmlsoap.org/soap/envelope/

## Adding an Endpoint

Right click on _AmazonEC2Binding_ under your project in the left pane
and select **Show Interface Viewer**.  Click the **Service Endpoints**
tab and add an endpoint for your cloud. It should look something like
this:

![Endpoint Configuration](https://raw.github.com/mspaulding06/wiki-images/master/soapui/soapui_config_endpoint.png)

You can also click the **Assign** button and assign the endpoint to
all requests.

*****

[[category.developer]]
[[category.tools]]