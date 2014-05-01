# Deltaclouds about Deltacloud

> Deltacloud provides the API server and drivers necessary for connecting to cloud providers.

> Deltacloud maintains long-term stability for scripts, tools and applications and backward compatibility across different versions.

> Deltacloud enables management of resources in different clouds by the use of one of three supported APIs. The supported APIs are the Deltacloud classic API, the DMTF CIMI API or even the EC2 API.

From http://deltacloud.apache.org/about.html

# Connecting to Eucalyptus

To start the Deltacloud service with the eucalyptus driver you should use the provider option to pass in your eucalyptus host information:
```
  deltacloudd -i eucalyptus -P 'ec2=10.111.1.16;s3=10.111.1.16'
```
You can use a ":PORT" suffix if you run eucalyptus on ports other than the default (8773)

# Example use

## Describing images
```
$ curl  --user "AKIOFSL6GNWHAEXAMPLE:.....secret.here....." "http://localhost:3001/api/images?format=xml"
127.0.0.1 - - [01/May/2014 06:43:54] "GET /api/images?format=xml HTTP/1.1" 200 1491 0.1487
<?xml version='1.0' encoding='utf-8' ?>
<images>
  <image href='http://localhost:3001/api/images/emi-8dcb47bc' id='emi-8dcb47bc'>
    <name>preciseservercloudimgamd64disk1img-1398899605X</name>
    <description>precise-server-cloudimg-amd64-disk1img/precise-server-cloudimg-amd64-disk1.img.manifest.xml</description>
    <owner_id>150564846067</owner_id>
    <architecture>x86_64</architecture>
    <state>available</state>
    <hardware_profiles>
      <hardware_profile href='http://localhost:3001/api/hardware_profiles/m1.small' id='m1.small' rel='hardware_profile'></hardware_profile>
      <hardware_profile href='http://localhost:3001/api/hardware_profiles/c1.medium' id='c1.medium' rel='hardware_profile'></hardware_profile>
      <hardware_profile href='http://localhost:3001/api/hardware_profiles/m1.large' id='m1.large' rel='hardware_profile'></hardware_profile>
      <hardware_profile href='http://localhost:3001/api/hardware_profiles/m1.xlarge' id='m1.xlarge' rel='hardware_profile'></hardware_profile>
      <hardware_profile href='http://localhost:3001/api/hardware_profiles/c1.xlarge' id='c1.xlarge' rel='hardware_profile'></hardware_profile>
    </hardware_profiles>
    <root_type>transient</root_type>
    <actions>
      <link href='http://localhost:3001/api/instances;image_id=emi-8dcb47bc' method='post' rel='create_instance' />
      <link href='http://localhost:3001/api/images/emi-8dcb47bc' method='delete' rel='destroy_image' />
    </actions>
  </image>
</images>
```

## Run instance
```
$ curl -X POST -F "keyname=root" -F "image_id=emi-8dcb47bc" --user "AKIOFSL6GNWHAEXAMPLE:.....secret.here....." "http://localhost:3001/api/instances?format=xml"
I, [2014-05-01T06:46:47.681560 #18039]  INFO -- : Launching instance of image emi-8dcb47bc, key: root, groups: 
127.0.0.1 - - [01/May/2014 06:46:48] "POST /api/instances?format=xml HTTP/1.1" 201 1204 0.6364
<?xml version='1.0' encoding='utf-8' ?>
<instance href='http://localhost:3001/api/instances/i-3a982ba6' id='i-3a982ba6'>
  <name>emi-8dcb47bc</name>
  <owner_id>150564846067</owner_id>
  <image href='http://localhost:3001/api/images/emi-8dcb47bc' id='emi-8dcb47bc'></image>
  <realm href='http://localhost:3001/api/realms/PARTI00' id='PARTI00'></realm>
  <state>PENDING</state>
  <hardware_profile href='http://localhost:3001/api/hardware_profiles/m1.small' id='m1.small'>
  </hardware_profile>
  <actions>
    <link href='http://localhost:3001/api/instances/i-3a982ba6/stop' method='post' rel='stop' />
    <link href='http://localhost:3001/api/instances/i-3a982ba6/run;id=i-3a982ba6' method='post' rel='run' />
  </actions>
  <launch_time>2014-05-01T06:46:48.227Z</launch_time>
  <public_addresses><address type='unavailable'></address></public_addresses>
  <private_addresses><address type='unavailable'></address></private_addresses>
  <firewalls><firewall href='http://localhost:3001/api/firewalls/default' id='default'></firewall></firewalls>
  <storage_volumes></storage_volumes>
  <authentication type='key'>
    <login>
      <keyname>root</keyname>
    </login>
  </authentication>
</instance>
```

*****

[[category.tools]]