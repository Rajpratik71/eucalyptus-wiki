The release process generally occurs in two steps:  the _final compose_ (software is packaged, signed and provided to the support team), which is also known as _software availability_; and _general availability_ (software is made available for download to the public).

Software releases of major and minor versions (non-maintenance versions) also incorporate a third step known as _post general availability_ in which steps are taken to prepare for the maintenance of the release.

# Software Availability (SA)

* Ensure version information in all source code is coherent using the [[Eucalyptus Version Checklist]]
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

# Post-GA

* Create a branch from _master_ with the name _maint/\<version\>/testing_, where _version_ is the major and minor version number from the release (i.e., if the release was 3.3.0, then the new branch would be _maint/3.3/testing_).
* Create a branch from _maint/<version>/testing_ called _maint/\<version\>/master_ (i.e., if the testing branch is called _maint/3.3/testing_, then the master branch is called _maint/3.3/master_).

*****

[[category.releng]]