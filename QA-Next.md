## Codename - Exit Zero (EZ)

#### Brain dump of ideas/wishes/hopes/desires (in priority order)
1. Reduce time to get from 0-running setup 
 * When machines are freed rekickstart them with CentOS so that they are ready to roll (saves us ~12m)
 * DONE - Don't do  lv_extend/resize2fs during BUILD, set the kickstart default volume size to 200G (saves us ~3m) 
 * Any package installations need to be pushed to kickstart
2. Fix configuration test units to be faster
 * pre_test_setup has a 2 minute sleep "for stabilization"
2. Reduce rinse-repeat cycle (rebuilding on same machines)
 * Make Jenkins jobs on MicroQA that can reset a system to many different phases
 * Just after PXE
 * Just after installation of packages or source
 * Just after configuration and load image
3. No more memo field 
 * (mc) Suggesting a different usage of a 'text based config'. Flexible config on the fly is good. QA's current nature of forcing user's to guess the syntax(or look it up on a remote page), and/or guess the requirements to run a test, and then provide a fail slow (really slow) feedback mechanism is not acceptable going forward. 
 * (mc) Provide a mechanism for all test units (or whatever we do going forward) to provide near instant '-h' like feedback on what arguments/configuration that unit can accept, and needs to run. Allow for a sequence to be 'dryrun' to test whatever config mechanisms the system provides going forward. 
 * (mc) Most tests currently use python's configParser libs. This is a really vanilla key=value syntax which allows for multiple contexts via brackets [your name here]. Suggesting that we allow for any config format anyone wants inside these brackets including yaml, xml, key/value, etc.. 
 * Consider using a YAML format consumable by Ansible to define parameters for system builds.
 
4. Make build process automated through a deployment tool (a la Ansible)
 * Ensure that all (third party) deps are installed from packages.
 * Cobbler integration for Ansible host inventory.
 * Investigate use of a _pull_ mechanism for a scalable solution (ansible-pull).
5. run all of our tests (eutester, eutester4j, FOG ..)
6. no more perl
 * Target python 2.7 or newer
 * Code should be compatible with python 3.x as much as possible (remember 2.x series will EOL in 2015!!)
 * Re-use code, see suggestion below about suite of tools. Modules which do one thing, do it really well, and can be re-used in both QA env, but testing, and possibly product. 
7. Stop calling it QA, it's so much more than that
 * What do we call it?
   + Personally I like the name [Mechanistic](http://www.thefreedictionary.com/mechanistic) (mspaulding)
   + Skynet
   + EZ (exitzero)
   + Marsupial
   + Rune
   + Loki
8. Leverage all work towards 'The system formally known as QA' for a larger test/dev use case. Dog food. This is a common use case, so any work we do towards it should be easily used/extended outside of Eucalyptus. 
9. Set constraints around the design to ensure work going into our test dev system to make sure work is flexible, shareable, ephemeral, modular, and re-usable. 
10. 'HAVE A DESIGN' ...that allows for easy collaboration. Design and describe the project so people can easily come and go from working on different portions of the project with a clear picture of what needs to be done for each part of the project.
 * The key word here is **modular**! Let's think of this as a suite of tools.
 * Follow the [Unix Philosophy](http://en.wikipedia.org/wiki/Unix_philosophy)

    > Write programs that do one thing and do it well.
11. Use DNS by default in all QA tests.
12. Track and manage resources (machines, san access, network addrs, etc). Allow for easy to extend resource policies. ie:groups, accounts, users, etc.. Policies that may mimic 'leases', limits on length of time a system may be 'leased', count of resources a user/group can use, etc.. Keep statistics on resource consumption. Be able to track resource consumption during different cycles of development. Resources may have 'read only' like perms for non-engineering usage. 
13. Look and feel should mimic our Eucalyptus interfaces where ever possible. 
14. More test metrics are a must. How much, how long, averages, historical data, what env. etc.. Data points for tests, test units, individual Eucalyptus operations, etc.. 
15. Would be nice if we can programmatically interface with our docs for install and setup portions of QA. ...or the reverse produce easy to read sudo code per setup/install step that can be consumed by our docs team. ...or both

# Notes from first design session
## Constraints/Assumptions
1. An OS installed
2. SSH access (via password or key)
3. No IT intervention
4. Public IPs / Network is working for them
5. No conflicting DHCP server (or DHCP server blocks requests from MAC_PREFIX)
6. Config management system will not conflict with installer/provisioner

## Use Cases
1. Basic Community User
    - Little config as possible
    - No enterprise bits
    - Push button deploy
    - Failure messages must be clear and allow remediation
2. Advanced PoC type customer
    - Enterprise bits
    - More customization of parameters (repo locations, network configs, DNS config, etc)
    - Possibility to redeploy on another VM or bare metal machine (maybe all or parts of the system)
    - Deploy in all supported topologies/configurations
    - Expand the deployment
3. Internal QA System
    - Manage many many installs
    - Multi User

https://eucalyptus.atlassian.net/wiki/display/EUCA/Repeatable+Deployment+Plan

### Phases of deployment
#### Phase 1 - Installation of MicroQA
1. Go to Eucalyptus Website
2. Download image in appropriate (KVM RAM, VirtualBox, VMDK), Ansible version
3. Start VM or deploy via Ansible
4. Prompt with passwords and URLs for VNC, SSH, and HTTP
5. Ncurses display of URL to go to for landing
6. Landing web page that outlines all the features and URLs

#### Phase 2 - Deployment
1. Create config step (small number of params), validate config, create artifact URI
2. Start job based off config artifact URI
4. Verify Infrastructure based on configuration
3. Create Eucalyptus install based off config

#### Phase 3 - Testing
1. Pass config to test sequences
2. Run test sequences

#### Phase 4 - Maintanence
1. Deploy Nagios/Ganglia
2. Ensure sosreports installed 
3. Verify config using Ansible

#### Phase 5 - Rinse & Repeat 
1. Restart from bare metal
2. Restart a particular config
3. Clean the system of Euca resources using LVM snapshots and reinstall 
4. Upgrade packages/source
5. Rerun validator/testing scripts


### Resource Manager (controls allocation and reservation of limited resources)
Resources
- IPs
    - Public IPs
       - Owner
       - Lease Time
    - Private IPs
       - Owner
       - Lease Time
    - Available Machines
       - State of the Box
       - Get information (IP, distro)
       - Whether its bare metal or VM
       - Owner
       - Lease Time
    - Accessibility to SAN (internal use case) 
       - Owner
       - Lease Time
- Also can manage configuration options with service API
- Externalizable database 
- Lightweight UI/API
- Quotas for resources allocation
- Ability to requests certain resources (allows customer user case to choose which machines will act as a particular role)

### TODO
Rebranding
SOS Report to Design
Adding Resource Management to design
Video showing deployment
Deployment Mechanisms:
- KVM
- VirtualBox
- VMDK
- Vagrant
- Ansible
Downstream Build view display in run order
Link cases to JIRA
Programmatically create params of which tests should run
Check on state of Validator plugins (agrimm)
Offline Mode (i.e. all things packaged without need for external repos)
Write script that cleans out Eucalyptus install

