# Release Build Promotion Tools

This page is an outline of the basic commands used when promoting builds. These are all part of [arado tools](https://github.com/mspaulding06/arado).

# Installation and configuration

In order to install the tools, you'll need to create a python virtual environment:

``virtualenv toolsenv``

Then enable it:

``. toolsenv/bin/activate``

Install required dependencies (on CentOS 6 systems where we run this, the setup.py install_requires option is not recognized):

``pip install argparse requests beautifulsoup jinja2 pexpect``

Install the tools:

``python setup.py build``

``python setup.py install``

Now you can use the Arado tools for promoting releases.

# Configuration File

You will also need to have a file arado.conf that provides information about how your repository is structured and where builds will be promoted to. Here's an example of the file content.

    [general]
    api-url = http://packages.release.eucalyptus-systems.com/api/1/genrepo
    uid = nobody
    gid = release-team
    fileperms = 664
    dirperms = 2775

    [paths]
    source = /srv/ci-builds
    destination = /srv/releases
    repotemp = /var/tmp

    [projects]
    eucalyptus = repo-euca@git.eucalyptus-systems.com:eucalyptus
    euca2ools = https://github.com/eucalyptus/euca2ools.git
    eucalyptus-console = https://github.com/eucalyptus/eucalyptus-console.git
    load-balancer-image = https://github.com/eucalyptus/load-balancer-image.git
    load-balancer-servo = https://github.com/eucalyptus/load-balancer-servo.git

    [mappings]
    eucalyptus = eucalyptus
    load-balancer-image = eucalyptus
    eucalyptus-console = eucalyptus
    euca2ools = euca2ools
 
## Configuration Options

* general - These are general options needed for doing build promotions
* paths
    * source - The source repository where ci builds live
    * destination - The release repository where builds are promoted to
    * repotemp - Location for building repositories before pushing them to the destination
* projects - This is a list of the projects that are available for release promotion. The key is the project name and the value is the git repository where it lives
* mappings - Mappings provide the name of the release repository that the project will be promoted to. The key is the project and the value is the release repository for promotion

*****

[[category.releng]]