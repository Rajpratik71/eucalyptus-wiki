The release process generally occurs in two steps:  the final compose, and general availability.

# Final Compose (SA)

* Ensure version information in all source code is coherent
  * **./VERSION**
      * Format is X.X.X (e.g., 3.3.0).
  * **./console/eucaconsole/version.py** which is generated when _setup.py_ is run.
  * **./clc/modules/bootstrap/src/main/native/{arguments.ggo,eucalyptus-bootstrap.h}** are generated using _gengetopt_.
  * **./clc/modules/msgs/src/main/java/com/eucalyptus/upgrade/Upgrades.java** should contain an enum value for the current version.
      * Format is vX_X_X (e.g., v3_3_0).
* Build final packages in Jenkins
  * Make sure to use _no_ tarball suffix, as final releases don't need git info.
  * If the release contains embargoed security fixes, make sure to build against the appropriate security fix branch.
* Keep final Jenkins builds ( _all_ jobs in the sequence, not just binaries or tarballs) forever
* Name the final Jenkins builds according to the version number
  * Edit Build Information > DisplayName
* Sign final tarball
  * `gpg --armor --sign --detach-sign`
* Sign final packages
  * Make sure to use sigv3
* Upload to staging server (release-repo)
  * source tarball (*.tar.gz)
     * /releases/$program/$series/source/*.tar.gz
  * source tarball signature (*.tar.gz.asc)
     * /releases/$program/$series/source/*.tar.gz.asc
  * source package (*.src.rpm)
     * /releases/$program/$series/$distro/$releasever/source/*.src.rpm)
  * binary packages (*.rpm _except_ *.src.rpm)
     * /releases/$program/$series/$distro/$releasever/$basearch/*.rpm

# General Availability (GA)

* Upload everything to public download server
  * Releases with security fixes that cause the source release to be delayed should exclude `eucalyptus-*.src.rpm` and `eucalyptus-*.tar.gz` until the embargo is lifted.
* Tag each git repo
  * Pushing tags automatically causes sources to upload to GitHub; if the source release must be delayed then this step must be delayed as well.
* Add n-v-r and commit data to [[List of Packages]] wiki page
* For a new series (e.g. x.y.0) work with IT to update where the mirrorlist points to by default

*****

[[category.releng]]