Using pbuilder
==============

There are several simple commands for operation. ``pbuilder create``,
``pbuilder update``, and ``pbuilder build`` commands are the typical commands
used. Let us look at the commands one by one.

Creating a base chroot image tar-ball
-------------------------------------

``pbuilder create`` will create a base chroot image tar-ball (base.tgz).
All other commands will operate on the resulting base.tgz If the Debian
release to be created within chroot is not going to be "sid" (which is
the default), the distribution code-name needs to be specified with the
:option:`--distribution` command-line option.

``debootstrap``  [#]_ is used to create the bare minimum Debian
installation, and then build-essential packages are installed on top of
the minimum installation using ``apt-get`` inside the chroot.

For fuller documentation of command-line options, see the pbuilder.8
manual page. Some configuration will be required for ``/etc/pbuilderrc``
for the mirror site  [#]_ to use, and proxy configuration may be
required to allow access through HTTP. See the pbuilderrc.5 manual page
for details.

Updating the base.tgz
---------------------

``pbuilder update`` will update the base.tgz. It will extract the
chroot, invoke ``apt-get update`` and ``apt-get dist-upgrade`` inside
the chroot, and then recreate the base.tgz (the base tar-ball).

It is possible to switch the distribution which the base.tgz is targeted
at at this point. Specify `` `` to change the distribution to sid.  [#]_

For fuller documentation of command-line options, see the pbuilder.8
manual page

Building a package using the base.tgz
-------------------------------------

To build a package inside the chroot, invoke :samp:`pbuilder build {whatever.dsc}`.
``pbuilder`` will extract the base.tgz to a temporary working directory,
enter the directory with chroot, satisfy the build-dependencies inside
chroot, and build the package. The built packages will be moved to a
directory specified with the :option:`--buildresult` command-line option.

The :option:`--basetgz` option can be used to specify which base.tgz to use.

``pbuilder`` will extract a fresh base chroot image from base.tgz.
(base.tgz is created with ``pbuilder create``, and updated with
``pbuilder update``). The chroot is populated with build-dependencies by
parsing debian/control and invoking ``apt-get``.

For fuller documentation of command-line options, see the pbuilder.8
manual page

Facilitating Debian Developers' typing, pdebuild
------------------------------------------------

``pdebuild`` is a little wrapper script that does the most frequent of
all tasks. A Debian Developer may try to do ``debuild``, and build a
package, inside a Debian source directory. ``pdebuild`` will allow
similar control, and allow package to be built inside the chroot, to
check that the current source tree will build happily inside the chroot.

``pdebuild`` calls ``dpkg-source`` to build the source packages, and then
invokes ``pbuilder`` on the resulting source package. However, unlike debuild,
the resulting deb files will be found in the :option:`--buildresult` directory.

See the :manpage:`pdebuild(1)` manual page for more details.

There is a slightly different mode of operation available in ``pdebuild`` since
version 0.97. ``pdebuild`` usually runs ``debian/rules clean`` outside of the
chroot; however, it is possible to change the behavior to run it inside the
chroot with the :option:`--use-pdebuild-internal`. It will try to bind mount
the working directory inside chroot, and run ``dpkg-buildpackage`` inside. It
has the following characteristics, and is not yet the default mode of
operation.

-  Satisfies build-dependency inside the chroot before creating source
   package. (which is a good point that default ``pdebuild`` could not
   do).

-  The working directory is modified from inside the chroot.

-  Building with ``pdebuild`` does not guarantee that it works with
   ``pbuilder``.

-  If making the source package fails, the session using the chroot is
   wasted (chroot creation takes a bit of time, which should be improved
   with cowdancer).

-  Does not work in the same manner as it used to; for example, :option:`--buildresult`
   does not have any effect.

-  The build inside chroot is ran with the current user outside chroot.

Configuration Files
-------------------

It is possible to specify all settings by command-line options. However,
for typing convenience, it is possible to use a configuration file.

``/etc/pbuilderrc`` and ``${HOME}/.pbuilderrc`` are read in when
``pbuilder`` is invoked. The possible options are documented in the
pbuilderrc.5 manual page.

It is useful to use ``--configfile`` option to load up a preset
configuration file when switching between configuration files for
different distributions.

Please note ``${HOME}/.pbuilderrc`` supersede system settings. Caveats
is that if you have some configuration, you may need to tweak the
configuration to work with new versions of pbuilder when upgrading.

Building packages as non-root inside the chroot
-----------------------------------------------

``pbuilder`` requires full root privilege when it is satisfying the
build-dependencies, but most packages do not need root privilege to
build, or even refused to build when they are built as root.
``pbuilder`` can create a user which is only used inside ``pbuilder``
and use that user id when building, and use the ``fakeroot`` command
when root privilege is required.

BUILDUSERID configuration option should be set to a value for a user id
that does not already exist on the system, so that it is more difficult
for packages that are being built with ``pbuilder`` to affect the
environment outside the chroot. When BUILDUSERNAME configuration option
is also set, ``pbuilder`` will use the specified user name and fakeroot
for building packages, instead of running as root inside chroot.

Even when using the fakerooting method, ``pbuilder`` will run with root
privilege when it is required. For example, when installing packages to
the chroot, ``pbuilder`` will run under root privilege.

To be able to invoke ``pbuilder`` without being root, you need to use
user-mode-linux, as explained in ?.

Using pbuilder for back-porting
-------------------------------

``pbuilder`` can be used for back-porting software from the latest
Debian distribution to the older stable distribution, by using a chroot
that contains an image of the older distribution, and building packages
inside the chroot. There are several points to consider, and due to the
following reasons, automatic back-porting is usually not possible, and
manual interaction is required:

-  The package from the unstable distribution may depend on packages or
   versions of packages which are only available in unstable. Thus, it
   may not be possible to satisfy Build-Depends: on stable (without
   additional backporting work).

-  The stable distribution may have bugs that have been fixed in
   unstable which need to be worked around.

-  The package in the unstable distribution may have problems building
   even on unstable.

Mass-building packages
----------------------

``pbuilder`` can be automated, because its operations are
non-interactive. It is possible to run ``pbuilder`` through multiple
packages non-interactively. Several such scripts are known to exist.
Junichi Uekawa has been running such a script since 2001, and has been
filing bugs on packages that fail the test of ``pbuilder``. There were
several problems with auto-building:

-  Build-Dependencies need to install non-interactively, but some
   packages are so broken that they cannot install without interaction
   (like postgresql).

-  When a library package breaks, or gcc/gcj/g++ breaks, or even bison,
   a large number of build failures are reported. (gcj-3.0 which had no
   "javac", bison which got more strict, etc.)

-  Some people were quite hostile against build failure reports.

Most of the initial bugs have been resolved in the ``pbuilder`` sweep
done around 2002, but these transitional problems which affect a large
portion of Debian Archive do arise from time to time. Regression tests
have their values.

A script that was used by Junichi Uekawa in the initial run is now
included in the ``pbuilder`` distribution, as ``pbuildd.sh``. It is
available in ``/usr/share/doc/pbuilder/examples/pbuildd/`` and its
configuration is in ``/etc/pbuilder/pbuildd-config.sh``. It should be
easy enough to set up for people who are used to ``pbuilder``. It has
been running for quite a while, and it should be possible to set the
application up on your system also. This version of the code is not the
most tested, but should function as a starter.

To set up pbuildd, there are some points to be aware of.

-  A file ``./avoidlist`` needs to be available with the list of
   packages to avoid building.

-  It will try building anything, even packages which are not aimed for
   your architecture.

-  Because you are running random build scripts, it is better to use the
   fakeroot option of ``pbuilder``, to avoid running the build under
   root privilege.

-  Because not all builds are guaranteed to finish in a finite time,
   setting a timeout is probably necessary, or pbuildd may stall with a
   bad build.

-  Some packages require a lot of disk space, around 2GB seems to be
   sufficient for the largest packages for the time being. If you find
   otherwise, please inform the maintainer of this documentation.

Auto-backporting scripts
------------------------

There are some people who use ``pbuilder`` to automatically back-port a
subset of packages to the stable distribution.

I would like some information on how people are doing it, I would
appreciate any feedback or information on how you are doing, or any
examples.

Using pbuilder for automated testing of packages
------------------------------------------------

``pbuilder`` can be used for automated testing of packages. It has the
feature of allowing hooks to be placed, and these hooks can try to
install packages inside the chroot, or run them, or whatever else that
can be done. Some known tests and ideas:

-  Automatic install-remove-install-purge-upgrade-remove-upgrade-purge
   test-suite (distributed as an example, ``B91dpkg-i``), or just check
   that everything installs somewhat (``execute_installtest.sh``).

-  Automatically running lintian (distributed as an example in
   ``/usr/share/doc/pbuilder/examples/B90lintian``).

-  Automatic debian-test of the package? The debian-test package has
   been removed from Debian. A ``pbuilder`` implementation can be found
   as debian/pbuilder-test directory, implemented through B92test-pkg
   script.

To use B92test-pkg script, first, add it to your hook directory.  [#]_.
The test files are shell scripts placed in
``debian/pbuilder-test/NN_name`` (where NN is a number) following
run-parts standard [#]_ for file names. After a successful build,
packages are first tested for installation and removal, and then each
test is ran inside the chroot. The current directory is the top
directory of the source-code. This means you can expect to be able to
use ./debian/ directory from inside your scripts.

Example scripts for use with pbuilder-test can be found in
``/usr/share/doc/pbuilder/examples/pbuilder-test``.

Using pbuilder for testing builds with alternate compilers
----------------------------------------------------------

Most packages are compiled with ``gcc`` or ``g++`` and using the default
compiler version, which was gcc 2.95 for Debian GNU/Linux 3.0 (i386).
However, Debian 3.0 was distributed with other compilers, under package
names such as ``gcc-3.2`` for gcc compiler version 3.2. It was therefore
possible to try compiling packages against different compiler versions.
``pentium-builder`` provides an infrastructure for using a different
compiler for building packages than the default gcc, by providing a
wrapper script called gcc which calls the real gcc. To use
``pentium-builder`` in ``pbuilder``, it is possible to set up the
following in the configuration:

::

    EXTRAPACKAGES="pentium-builder gcc-3.2 g++-3.2"
    export DEBIAN_BUILDARCH=athlon
    export DEBIAN_BUILDGCCVER=3.2

It will instruct ``pbuilder`` to install the ``pentium-builder`` package
and also the GCC 3.2 compiler packages inside the chroot, and set the
environment variables required for ``pentium-builder`` to function.

.. [#] debootstrap or cdebootstrap can be chosen

.. [#]
   The mirror site should preferably be a local mirror or a cache
   server, so as not to overload the public mirrors with a lot of
   access. Use of tools such as apt-proxy would be advisable.

.. [#] Only upgrading is supported. Debian does not generally support
       downgrading (yet?).

.. [#] It is possible to specify ``--hookdir /usr/share/doc/pbuilder/examples``
       command-line option to include all example hooks as well.

.. [#] See run-parts(8). For example, no '.' in file names!
