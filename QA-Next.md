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
 * (mc) Suggesting a different usage of a 'text based config'. Flexible config on the fly is good. Guessing the syntax, requirements to run, and the fail slow (really slow) nature of QA's memo field today is not acceptable going forward. 
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
8. Leverage all work towards 'The system formally known as QA' for a larger test/dev use case. Dog food. This is a common use case, so any work we do towards it should be easily used/extended outside of Eucalyptus. 
9. Set constraints around the design to ensure work going into our test dev system to make sure work is flexible, shareable, ephemeral, modular, and re-usable. 
10. 'HAVE A DESIGN' ...that allows for easy collaboration. Design and describe the project so people can easily come and go from working on different portions of the project with a clear picture or what needs to be done for each part of the project.
 * The key word here is **modular**! Let's think of this as a suite of tools.
 * Follow the [Unix Philosophy](http://en.wikipedia.org/wiki/Unix_philosophy)

    > Write programs that do one thing and do it well.
11. Use DNS by default in all QA tests.
12. Track and manage resources (machines, san access, network addrs, etc). Allow for easy to extend resource policies. ie:groups, accounts, users, etc.. Policies that may mimic 'leases', limits on length of time a system may be 'leased', count of resources a user/group can use, etc.. Keep statistics on resource consumption. Be able to track resource consumption during different cycles of development. Resources may have 'read only' like perms for non-engineering usage. 
13. Look and feel should mimic our Eucalyptus interfaces where ever possible. 
14. More test metrics are a must. How much, how long, averages, historical data, what env. etc.. Data points for tests, test units, individual Eucalyptus operations, etc.. 
15. Would be nice if we can programmatically interface with our docs for install and setup portions of QA. ...or the reverse produce easy to read sudo code per setup/install step that can be consumed by our docs team. ...or both