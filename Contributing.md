# Contributing to Eucalyptus

If you are interested in contributing to Eucalyptus code, please read the
following instructions and guidelines.

## Legal Prerequisites

Significant contributions (usually those more than about 15 lines of code)
require that you sign our contributor agreement before they can be accepted.
This agreement is located at 
http://www.eucalyptus.com/participate/code/contribute/agreement

## Jira Issue(s)

Before committing any code, please make note of the issue IDs to which your
code relates.  You may also want to comment in the issues to let people
know that you are working on a fix.  If no issue exists yet for the code
you are writing, please create one.  See the REPORTING-BUGS file in the
source tree for more information about this process.

Be aware that if an issue is still in "Unconfirmed" or "Investigating"
state, it may mean that the Eucalyptus engineering team has not yet 
confirmed that they can reproduce the issue, and this is required before a 
patch can be reviewed.  Providing a test case can accelerate this process.

## Commit Messages

If you are developing your patch in git (which we highly recommend), please
ensure that your commit messages include a short summary line (50 chars or
less), followed by a blank line, followed by a description of what the 
patch does (in lines of 72 chars or less).  The description should contain
all issue IDs relevant to the commit.  Ideally, fixes for separate issues
should be in separate commits.

## Sending a Patch

There are three ways to send a patch:

* If you have a github account, commit your patch to your fork of
eucalyptus, and create a pull request.  If you do this, please link the
relevant Jira issue to the pull request.  To do this:
     * Go to the issue at http://eucalyptus.atlassian.net
     * Select more actions -> link
     * Select "web link"
     * Paste the url of the pull request into the URL field
     * Enter "github pull request" in the link text field
     * Click the "Link" button to submit the link
* If you have simply cloned our git repository, you can use "git 
format-patch <commit-id>" to export your patch to a file.  You can then
attach that file to the relevant issue at http://eucalyptus.atlassian.net
* If you are not using git (e.g., you've created a path against a release
tarball or an RPM or DEB source package), you may create a unified diff
and attach it to the relevant issue.  If you do this, it is very important
to add a comment stating exactly which source the diff originated from.

**NOTE:** If the issue for which you are submitting a patch is already in
the "Confirmed" state in Jira, you can use the "Submit Patch" action to
send the issue to "In Review" state.  This will notify the Eucalyptus
developers that the patch is ready for code review.

## The Patch Lifecycle

Once the patch is submitted (following the above guidelines), the following
process occurs:

* The patch is checked for basic integration (apply cleanly, compile cleanly, obvious incompatibilities)
* The patch receives closer inspection and is put through our internal QA system.  This system runs tests on various architectures (32 and 64 bit) and various distributions we currently support (and plan to support)
* If the patch shows regressions or integration problems, suggestions are made to the contributor on what failed and how to fix it, if possible. (Note that once a new version of the patch is submitted, the procedure will start from the beginning)
* Specific tests are produced for the patch, and the patch is tested against them, to ensure behavior of the patch is correct for current and future branches;
* If the patch is accepted, it is integrated into our development branch;
* If the patch is not accepted, a report is issued explaining why it cannot be accepted, and guidelines are provided showing how to remedy the problem for subsequent patch acceptance.