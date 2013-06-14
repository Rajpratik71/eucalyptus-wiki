# QA system renovation 2013

#### Brain dump of ideas/wishes/hopes/desires (in priority order)
1. Reduce time to get from 0-running setup 
 * When machines are freed rekickstart them with CentOS so that they are ready to roll (saves us ~12m)
 * Don't do  lv_extend/resize2fs during BUILD, set the kickstart default volume size to 200G (saves us ~3m)
2. Reduce rinse-repeat cycle (rebuilding on same machines)
 * Make Jenkins jobs on MicroQA that can reset a system to many different phases
 * Just after PXE
 * Just after installation of packages or source
 * Just after configuration and load image
3. No more memo field
4. run all of our tests (eutester, eutester4j, FOG ..)
5. no more perl
6. Stop calling it QA, it's so much more than that
7. Leverage all work towards 'The system formally known as QA' for a larger test/dev use case. Dog food. This is a common use case, so any work we do towards it should be easily used/extended outside of Eucalyptus. 
8. Set constraints around the design to ensure work going into our test dev system to make sure work is flexible, shareable, ephemeral, modular, and re-usable. 
9. 'HAVE A DESIGN' ...that allows for easy collaboration. Design and describe the project so people can easily come and go from working on different portions of the project with a clear picture or what needs to be done for each part of the project. 
