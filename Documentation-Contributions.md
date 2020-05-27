It’s easy to patch the Eucalyptus documentation, and we’d love your help! This document mostly describes our Git setup. You'll also need to see the [README](https://github.com/eucalyptus/documentation/blob/master/README) for information on how to build the documentation, and if you have questions not answered here, please come find us on our [mailing list](http://lists.eucalyptus.com/cgi-bin/mailman/listinfo/documentation), or on Freenode IRC in #eucalyptus-docs. We'd love to help you get started!

Our repository is available here on Github, and the best way to send us your patches is through a Github pull request. If you’ve never done this before, you’ll need to do a couple of things to get set up:
* Create a [Github account](https://github.com/plans)
* Go to [https://github.com/eucalyptus/documentation/](https://github.com/eucalyptus/documentation/) and click Fork
* Clone the resulting repository by using ``git clone --depth 1 git@github.com:[username]/documentation.git`` on your local machine
* Enter the new directory ``cd documentation``
* We’re going to be using more than one remote git repository, so rename this remote to make it clear what’s what using ``git remote rename origin [forkname]``
* Add the Eucalyptus Github repository as a remote using ``git remote add [remotename] https://github.com/eucalyptus/documentation.git``

If you’re already set up, making sure that your fork is up to date will make it easier to create and work with your pull request.
* Fetch the latest changes from the Eucalyptus Github repository using ``git fetch [remotename]``
* Merge those changes into your repository using ``git merge [remotename]/master``

Once you’re up to date, you can safely start editing!
* Make a new branch for your changes using ``git checkout -b [branchname]``
* Make your changes, committing locally as often as you’d like using ``git commit -a``
* When you’re done, push your changes to your fork on Github using ``git push [remotename] [branchname]``
* Go to the website for your fork (https://github.com/[username]/documentation) and click on Pull Request

As a matter of policy, we ask that every change to our repository be reviewed by at least one person other than the author. Whether or not you have commit access, you can help us with this, by keeping an eye on [incoming pull requests](https://github.com/eucalyptus/documentation/pulls) and commenting on any you’ve reviewed. 

When you get commit access to the Eucalyptus Systems repository, you’ll be able to move from reviewing pull requests on the website, to actually pulling them into the repository. (They still need to be reviewed though--don’t pull anything you haven’t actually read!)

* Before you can commit your first pull request, you’ll have add the Eucalyptus Systems repository as a remote to your own repository using ``git remote add [internalremotename] git+ssh://repo-docs@git.eucalyptus-systems.com/documentation``
* When you’re ready to patch in a pull request, find the patch URL (https://github.com/eucalyptus/documentation/pull/[number].patch)
* To keep things clean, make sure you’re on your master branch before you apply the patch using ``git checkout master``
* Apply the patch to your local repository using ``curl [patchURL] | git am``
* Make sure the docs build, and do any more review that’s required
* Push the changes to the Eucalyptus Systems repository using ``git push [internalremotename]``

*****

[[category.docs-team]]
[[category.contributions]]