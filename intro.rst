Introducing pbuilder
====================

Aims of pbuilder
----------------

``pbuilder`` stands for Personal Builder, and it is an automatic Debian
Package Building system for personal development workstation
environments. ``pbuilder`` aims to be an easy-to-setup system for
auto-building Debian packages inside a clean-room environment, so that
it is possible to verify that a package can be built on most Debian
installations. The clean-room environment is achieved through the use of
a base chroot image, so that only minimal packages will be installed
inside the chroot.

The Debian distribution consists of free software accompanied with
source. The source code within Debian's "main" section must build within
Debian "main", with only the explicitly specified build-dependencies
installed.

The primary aim of ``pbuilder`` is different from other auto-building
systems in Debian in that its aim is not to try to build as many
packages as possible. It does not try to guess what a package needs, and
in most cases it tries the worst choice of all if there is a choice to
be made.

In this way, ``pbuilder`` tries to ensure that packages tested against
``pbuilder`` will build properly in most Debian installations, hopefully
resulting in a good overall Debian source-buildability.

The goal of making Debian buildable from source is somewhat
accomplished, and has seen good progress. In the past age of Debian 3.0,
there were many problems when building from source. More recent versions
of Debian is much better.
