Troubleshooting and development
===============================

Reporting bugs
--------------

To report bugs, it would be important to have a log of what's going
wrong. Most of the time, adding a :option:`--debug` option and re-running the session
should do the trick. Please send the log of such session along with your
problem to ease the debugging process.

Mailing list
------------

There is a mailing list for ``pbuilder`` on alioth
(pbuilder-maint@lists.alioth.debian.org). You can subscribe through the
`alioth web interface <http://alioth.debian.org/mail/?group_id=30778>`_.

IRC Channel
-----------

For coordination and communication, IRC channel #pbuilder on
irc.oftc.net is used. Please log your intent there when you are going to
start doing some changes and committing some change.

Information for pbuilder developers
-----------------------------------

This section tries to document current development practices and how
things generally operate in development.

``pbuilder`` is co-maintained with resources provided by Alioth. There is an
Alioth project page at http://alioth.debian.org/projects/pbuilder.  Home page
is also available, at http://pbuilder.alioth.debian.org/ which shows this text.
git repository is available through http, git, or, if you have an account on
alioth, ssh.

::

    git-clone git://git.debian.org/git/pbuilder/pbuilder.git
    git-clone http://git.debian.org/git/pbuilder/pbuilder.git
    git-clone ssh://git.debian.org/git/pbuilder/pbuilder.git

Git commit message should have the first one line describing what the
commit does, formatted in the way debian/changelog is formatted because
it is copied verbatim to changelog via git-dch. The second line is
empty, and the rest should describe the background and extra information
related to implementation of the commit.

Test-suites are available in ``./testsuite/`` directory. Changes are
expected not to break the test-suites. ``./run-test.sh`` is a basic
test-suite, which puts a summary in ``run-test.log``, and
``run-test-cdebootstrap.log``. ``./run-test-regression.sh`` is a
regression test-suite, which puts the result in
``run-test-regression.log``. Currently, run-test.sh is ran automatically
daily to ensure that pbuilder is working.

+-----------------------------------------+--------------------------------------+
| Directory                               | Meaning                              |
+=========================================+======================================+
| ``./testsuite/``                        | Directory for testsuite              |
+-----------------------------------------+--------------------------------------+
| ``./testsuite/run-test.sh``             | Daily regression test to test        |
|                                         | against Debian Archive changes       |
|                                         | breaking pbuilder.                   |
+-----------------------------------------+--------------------------------------+
| ``./testsuite/run-test.log``            | A summary of testsuite               |
+-----------------------------------------+--------------------------------------+
| ``./testsuite/normal/``                 | Directory for testsuite results of   |
|                                         | running pbuilder with debootstrap    |
+-----------------------------------------+--------------------------------------+
| ``./testsuite/cdebootstrap/``           | Directory for testsuite results of   |
|                                         | running pbuilder with cdebootstrap   |
+-----------------------------------------+--------------------------------------+
| ``./testsuite/run-regression.sh``       | Regression testsuite, ran every time |
|                                         | change is made to pbuilder to make   |
|                                         | sure there is no regression.         |
+-----------------------------------------+--------------------------------------+
| ``./testsuite/run-regression.log``      | Summary of test result               |
+-----------------------------------------+--------------------------------------+
| ``./testsuite/regression/BugID-*.sh``   | Regression tests, exit 0 for         |
|                                         | success, exit 1 for failure          |
+-----------------------------------------+--------------------------------------+
| ``./testsuite/regression/BugID-*``      | Files used for the regression        |
|                                         | testsuite.                           |
+-----------------------------------------+--------------------------------------+
| ``./testsuite/regression/log/BugID-*    | Output of the regression test,       |
| .sh.log``                               | output from the script is redirected |
|                                         | by run-regression.sh                 |
+-----------------------------------------+--------------------------------------+

Table: Directory structure of the testsuite

When making changes, changes should be documented in the Git commit log.
git-dch will generate debian/changelog from the commit log. Make the
first line of your commit log meaningful, and add any bug-closing
information available. debian/changelog should not be edited directly
unless when releasing a new version.

A TODO file is available in ``debian/TODO``. It's mostly not
well-maintained, but hopefully it will be more up-to-date when people
start using it. emacs todoo-mode is used in editing the file.

When releasing a new version of ``pbuilder``, the version is tagged with
the git tag X.XXX (version number). This is done with ``./git-tag.sh``
script available in the source tree.
