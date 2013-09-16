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

# Command Line Tools

## arado-describe-build

This command will describe the content of a build. You can use this to check that the build you are about to promote is the one that you want. You must pass both the ``--project`` option and the ``--commit`` option. The commit can be any reference that git understands. Here's an example:

``arado-describe-build --project eucalyptus --commit testing``

This will show packages built from the **testing** branch of **eucalyptus**. If there are no packages built at the **HEAD**, then you will get back packages from the latest build associated with the reference.

## arado-describe-commit

Again, you need to pass both ``--project`` and ``--commit`` when using this command. Instead of showing you the packages in the build, it will return the commit hash for the build you've specified. For instance, if you ran:

``arado-describe-commit --project euca2ools --commit 3.0.1``

You would get the commit hash ``0f134a3250babd043f903512564ec237ce8db5f7``, which is the commit tagged for the 3.0.1 release.

## arado-promote-build

This command allows you to promote a single CI build from your ``source`` repository to a ``destination`` repository.

Options:
* ``--project`` - The project name to promote.
* ``--commit`` - The git commit reference to promote.
* ``--release`` - The major and minor release number (e.g., 3.3). This is used to determine the location to promote to on the release repository. For instance, if the project is **eucalyptus** and the release is **3.3**, then since the eucalyptus project is mapped to the eucalyptus repository, the build will be promoted to **eucalyptus/3.3/** (or in the case of a nightly, **eucalyptus/nightly/3.3**).
* ``--type`` - The type of the build to be promoted can dictate the location that it is promoted to. The possible values to pass are: release, prerelease and nightly.
* ``--key`` - The signing key to use for the promoted packages. The value can again be: release, prerelease or nightly.
* ``--gpgdir`` - You can specify the location where your GPG keys are kept to do signing, otherwise the GNUPGHOME environment variable will be used.
* ``--merge`` - Merge is very helpful if you are promoting multiple projects that are mapped to the same repository. For instance if you wanted to promote 2 different projects that are mapped to the same repository, you can do something like this:

``arado-promote-build --project eucalyptus --release 3.3 --type release --key release``
``arado-promote-build --project load-balancer-image --release 3.3 --type release --key release --merge``

Notice that we ``--merge`` with the second command so that the first set of packages promoted do not get overwritten.

## arado-rebuild-repo

This is usually used in situations where you simply want to rebuild the metadata for a repository, or resign the packages it contains. This takes the same arguments as the ``arado-promote-build`` tool.

## Tool Usage

Currently, these tools can be used manually, but they are usually called from a shell script. A build promotion consists of running a number of these commands to complete a release.

*****

[[category.releng]]