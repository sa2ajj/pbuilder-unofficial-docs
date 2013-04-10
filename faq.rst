Frequently asked questions
==========================

Here, known problems and frequently asked questions are documented. This
portion was initially available in README.Debian file, but moved here.

pbuilder create fails
---------------------

It often happens that ``pbuilder`` cannot create the latest chroot. Try
upgrading ``pbuilder`` and debootstrap. It is currently only possible to
create software that handles the past. Future prediction is a feature
which may be added later after we have become comfortable with the past.

There are people who occasionally back port debootstrap to stable
versions; hunt for them.

When there are errors with the debootstrap phase, the debootstrap script
needs to be fixed. ``pbuilder`` does not provide a way to work around
debootstrap.

Directories that cannot be bind-mounted
---------------------------------------

Because of the way ``pbuilder`` works, there are several directories
which cannot be bind-mounted when running ``pbuilder``. The directories
include ``/tmp``, ``/var/cache/pbuilder``, and system directories such
as ``/etc`` and ``/usr``. The recommendation is to use directories under
the user's home directory for bind-mounts.

Logging in to pbuilder to investigate build failure
---------------------------------------------------

It is possible to invoke a shell session after a build failure. Example
hook scripts are provided as ``C10shell`` and ``C11screen`` scripts.
C10shell script will start bash inside chroot, and C11screen script will
start GNU screen inside the chroot.

Logging in to pbuilder to modify the environment
------------------------------------------------

It is sometimes necessary to modify the chroot environment. ``login``
will remove the contents of the chroot after logout. It is possible to
invoke a shell using hook scripts. ``pbuilder update`` executes 'E'
scripts, and a sample for invoking a shell is provided as ``C10shell``.

::

    $ mkdir ~/loginhooks
    $ cp C10shell ~/loginhooks/E10shell
    $ sudo pbuilder update --hookdir ~/loginhooks/E10shell

It is also possible to add ``--save-after-exec`` and/or
``--save-after-login`` options to the ``pbuilder login`` session to
accomplish the goal. It is possible to add the ``--uml-login-nocow``
option to ``pbuilder-user-mode-linux`` session as well.

Setting BUILDRESULTUID for sudo sessions
----------------------------------------

It is possible to set

::

    BUILDRESULTUID=$SUDO_UID

in pbuilderrc to set the proper BUILDRESULTUID when using ``sudo``.

Notes on usage of $TMPDIR
-------------------------

If you are setting $TMPDIR to an unusual value, of other than ``/tmp``,
you will find that some errors may occur inside the chroot, such as
``dpkg-source`` failing.

There are two options, you may install a hook to create that directory,
or set

::

    export TMPDIR=/tmp

in pbuilderrc. Take your pick.

An example script is provided as ``examples/D10tmp`` with ``pbuilder``.

Creating a shortcut for running ``pbuilder`` with a specific distribution
-------------------------------------------------------------------------

When working with multiple chroots, it would be nice to work with
scripts that reduce the amount of typing. An example script
``pbuilder-distribution.sh`` is provided as an example. Invoking the
script as ``pbuilder-squeeze`` will invoke ``pbuilder`` with a squeeze
chroot.

Using environmental variables for running ``pbuilder`` for specific distribution
--------------------------------------------------------------------------------

This section [#]_ describes briefly a way to setup and use multiple
pbuilder setups by creating a pbuilderrc configuration in your home path
(``$HOME/.pbuilderrc``) and using the variable "DIST" when running
pbuilder or pdebuild.

.. [#]
   This part of the documentation contributed by Andres Mejia

   This example was taken from a wiki
   (`https://wiki.ubuntu.com/PbuilderHowto <https://wiki.ubuntu.com/PbuilderHowto>`_).

First, setup ``$HOME/.pbuilderrc`` to look like:

::

    if [ -n "${DIST}" ]; then
            BASETGZ="`dirname $BASETGZ`/$DIST-base.tgz"
            DISTRIBUTION="$DIST"
            BUILDRESULT="/var/cache/pbuilder/$DIST/result/"
            APTCACHE="/var/cache/pbuilder/$DIST/aptcache/"
    fi

Then, whenever you wish to use pbuilder for a particular distro, assign
a value to "DIST" that is one of the distros available for Debian or any
Debian based distro you happen to be running (i.e. whatever is found
under /usr/lib/debootstrap/scripts).

Here's some examples on running pbuilder or pdebuild:

::

    DIST=gutsy sudo pbuilder create

    DIST=sid sudo pbuilder create --mirror http://http.us.debian.org/debian

    DIST=gutsy sudo pbuilder create \
            --othermirror "deb http://archive.ubuntu.com/ubuntu gutsy universe \
            multiverse"

    DIST=gutsy sudo pbuilder update

    DIST=sid sudo pbuilder update --override-config --mirror \
    http://http.us.debian.org/debian \
    --othermirror "deb http://http.us.debian.org/debian sid contrib non-free"

    DIST=gutsy pdebuild

Using special apt sources lists, and local packages
---------------------------------------------------

If you have some very specialized requirements on your apt setup inside
``pbuilder``, it is possible to specify that through the
:option:`--othermirror` option.  Try something like: ``--othermirror "deb
http://local/mirror stable main|deb-src http://local/source/repository ./"``

To use the local file system instead of HTTP, it is necessary to do
bind-mounting. :option:`--bindmounts` is a command-line option useful for such
cases.

It might be convenient to use your built packages from inside the
chroot. It is possible to automate the task with the following
configuration. First, set up pbuilderrc to bindmount your build results
directory.

::

    BINDMOUNTS="/var/cache/pbuilder/result"

Then, add the following hook

::

    # cat /var/cache/pbuilder/hooks/D70results
    #!/bin/sh
    cd /var/cache/pbuilder/result/
    /usr/bin/dpkg-scanpackages . /dev/null > /var/cache/pbuilder/result/Packages
    /usr/bin/apt-get update

This way, you can use ``deb file:/var/cache/pbuilder/result``

To add new apt-key inside chroot:

::

    sudo pbuilder --login --save-after-login
    # apt-key add - <<EOF
    ...public key goes here...
    EOF
    # logout

How to get pbuilder to run apt-get update before trying to satisfy build-dependency
-----------------------------------------------------------------------------------

You can use hook scripts for this. D scripts are run before satisfying
build-dependency.

`This snippet comes from Ondrej
Sury. <http://lists.debian.org/debian-devel/2006/05/msg00550.html>`_

Different bash prompts inside pbuilder login
--------------------------------------------

To make distinguishing bash prompts inside ``pbuilder`` easier, it is
possible to set environment variables such as PS1 inside ``pbuilderrc``

With versions of bash more recent than 2.05b-2-15, the value of the
debian\_chroot variable, if set, is included in the value of PS1 (the
Bash prompt) inside the chroot. In prior versions of bash, [#]_ setting
PS1 in pbuilderrc worked.

.. [#]
   Versions of bash from and before Debian 3.0


example of debian\_chroot

::

        export debian_chroot="pbuild$$"

example of PS1

::

        export PS1="pbuild chroot 32165 # "

Creating a chroot reminder
--------------------------

Bash prompts will help you remember that you are inside a chroot. There
are other cases where you may want other signs of being inside a chroot.
Check out the ``examples/F90chrootmemo`` hook script. It will create a
file called ``/CHROOT`` inside your chroot.

Using /var/cache/apt/archives for the package cache
---------------------------------------------------

For the help of low-bandwidth systems, it is possible to use
``/var/cache/apt/archives`` as the package cache. Just specify it
instead of the default ``/var/cache/pbuilder/aptcache``.

It is however not possible to do so currently with the user-mode-linux
version of ``pbuilder``, because ``/var/cache/apt/archives`` is usually
only writable by root.

Use of dedicated tools such as apt-proxy is recommended, since caching
of packages would benefit the system outside the scope of ``pbuilder``.

pbuilder back ported to stable Debian releases
----------------------------------------------

Currently stable back port of pbuilder is available at backports.org.

Warning about LOGNAME not being defined
---------------------------------------

You might see a lot of warning messages when running ``pbuilder``.

::

        dpkg-genchanges: warning: no utmp entry available and LOGNAME not defined; using uid of process (1234)

It is currently safe to ignore this warning message. Please report back
if you find any problem with having LOGNAME unset. Setting LOGNAME
caused a few problems when invoking ``chroot``. For example, dpkg
requires getpwnam to succeed inside chroot, which means LOGNAME and the
related user information have to be set up inside chroot.

Cannot Build-conflict against an essential package
--------------------------------------------------

``pbuilder`` does not currently allow Build-Conflicts against essential
packages. It should be obvious that essential packages should not be
removed from a working Debian system, and a source package should not
try to force removal of such packages on people building the package.

Avoiding the "ln: Invalid cross-device link" message
----------------------------------------------------

By default, ``pbuilder`` uses hard links to manage the ``pbuilder``
package cache. It is not possible to make hard links across different
devices; and thus this error will occur, depending on your set up. If
this happens, set

::

    APTCACHEHARDLINK=no

in your pbuilderrc file. Note that packages in ``APTCACHE`` will be
copied into chroot local cache, so plan for enough space on
``BUILDPLACE`` device.

Using fakechroot
----------------

It is possible to use ``fakechroot`` instead of being root to run
``pbuilder``; however, several things make this impractical.
``fakechroot`` overrides library loads and tries to override default
libc functions when providing the functionality of virtual ``chroot``.
However, some binaries do no use libc to function, or override the
overriding provided by ``fakechroot``. One example is ``ldd``. Inside
``fakechroot``, ``ldd`` will check the library dependency outside of the
chroot, which is not the expected behavior.

To work around the problem, debootstrap has a ``--variant fakechroot``
option. Use that, so that ldd and ldconfig are overridden.

Make sure you have set your LD\_PRELOAD path correctly, as described in
the fakechroot manpage.

Using debconf inside pbuilder sessions
--------------------------------------

To use debconf inside ``pbuilder``, setting DEBIAN\_FRONTEND to
“readline” in ``pbuilderrc`` should work. Setting it to “dialog” should
also work, but make sure whiptail or dialog is installed inside the
chroot.

nodev mount options hinder pbuilder activity
--------------------------------------------

If you see messages such as this when building a chroot, you are
mounting the file system with the nodev option.

::

        /var/lib/dpkg/info/base-files.postinst: /dev/null: Permission denied

You will also have problems if you mount the file system with the noexec
option, or nosuid. Make sure you do not have these flags set when
mounting the file system for ``/var/cache/pbuilder`` or $BUILDPLACE.

This is not a problem when using ``user-mode-linux``.

See `316135 <http://bugs.debian.org/316135>`_ for example.

pbuilder is slow
----------------

``pbuilder`` is often slow. The slowest part of ``pbuilder`` is
extracting the tar.gz every time ``pbuilder`` is invoked. That can be
avoided by using ``pbuilder-user-mode-linux``.
``pbuilder-user-mode-linux`` uses COW file system, and thus does not
need to clean up and recreate the root file system.

``pbuilder-user-mode-linux`` is slower in executing the actual build
system, due to the usual ``user-mode-linux`` overhead for system calls.
It is more friendly to the hard drive.

``pbuilder`` with cowdancer is also an alternative that improves speed
of pbuilder startup.

Using pdebuild to sponsor package
---------------------------------

To sign a package marking for sponsorship, it is possible to use :option:`--auto-debsign`
and :option:`--debsign-k` options of ``pdebuild``.

::

	pdebuild --auto-debsign --debsign-k XXXXXXXX

Why is there a source.changes file in ../?
------------------------------------------

When running ``pdebuild``, ``pbuilder`` will run dpkg-buildpackage to
create a Debian source package to pass it on to ``pbuilder``. File named
XXXX\_YYY\_source.changes is what remains from that process. It is
harmless unless you try to upload it to the Debian archive.

This behavior is different when running through
``--use-pdebuild-internal``

amd64 and i386-mode
-------------------

amd64 architectures are capable of running binaries in i386 mode. It is
possible to use ``pbuilder`` to run packages, using ``linux32`` and
:command:`debootstrap --arch` option. Specifically, a command-line option like the
following will work.

::

    pbuilder create --distribution sid --debootstrapopts --arch --debootstrapopts i386 \
             --basetgz /var/cache/pbuilder/base-i386.tgz --mirror http://ftp.jp.debian.org/debian
    linux32 pbuilder build --basetgz /var/cache/pbuilder/base-i386.tgz

Using tmpfs for buildplace
--------------------------

To improve speed of operation, it is possible to use tmpfs for pbuilder
build location. Mount tmpfs to :file:`/var/cache/pbuilder/build`, and set

::

    APTCACHEHARDLINK=no

Using svn-buildpackage together with pbuilder
---------------------------------------------

pdebuild command can be used with svn-buildpackage --svn-builder
command-line option.  [#]_

.. [#]
   `Zack has posted an example on his
   blog. <http://upsilon.cc/~zack/blog/posts/2007/09/svn-cowbuilder/>`_

::

    alias svn-cowbuilder="svn-buildpackage --svn-builder='pdebuild --pbuilder cowbuilder"
