Minor archaeological details
============================

Documentation history
---------------------

This document was started on 28 Dec 2002 by Junichi Uekawa, trying to
document what is known about ``pbuilder``.

This documentation is available from the ``pbuilder`` source tar-ball,
and from the git repository of ``pbuilder`` (web-based access is
possible). A copy of this documentation can be found on the `Alioth
project page for
pbuilder <http://pbuilder.alioth.debian.org/pbuilder-doc.html>`_. `There
is also a PDF
version <http://pbuilder.alioth.debian.org/pbuilder-doc.pdf>`_. The
homepage for ``pbuilder`` is
`http://pbuilder.alioth.debian.org/ <http://pbuilder.alioth.debian.org/>`_
hosted by alioth project.

Documentation is written using DocBook XML, with emacs PSGML mode, and
using wysidocbookxml for live previewing.

Possibly inaccurate Background History of pbuilder
--------------------------------------------------

The following is a most possibly inaccurate account of how ``pbuilder``
came to happen, and other attempts to make something like ``pbuilder``
happen. This part of the document was originally in the AUTHORS file, to
give credit to those who existed before ``pbuilder``.

The Time Before pbuilder
~~~~~~~~~~~~~~~~~~~~~~~~

There was once dbuild, which was a shell script to build Debian packages
from source. Lars Wirzenius wrote that script, and it was good, short,
and simple (probably). There was nothing like build-depends then (I
think), and it was simple. It could have been improved, I could only
find references and no actual source.

debbuild was probably written by James Troup. I don't know it because I
have never seen the actual code, I could only find some references to it
on the net, and mailing list logs.

sbuild is a perl script to build Debian packages from source. It parses
Build-Depends, and performs other miscellaneous checks, and has a lot of
hacks to actually get things building, including a table of what package
to use when virtual packages are specified (does it do that still?). It
supports the use of a local database for packages which do not have
build-dependencies. It was written by Ronan Hodek, and I think it was
patched and fixed and extended by several people. It is part of
wanna-build, and used extensively in the Debian buildd system. I think
it was maintained mostly by Ryan Murray.

Birth of pbuilder
~~~~~~~~~~~~~~~~~

wanna-build (sbuild) was (at the time of year 2001) quite difficult to
set up, and it was never a Debian package. dbuild was something that
predated Build-Depends.

Building packages from source using Build-Depends information within a
chroot sounded trivial; and ``pbuilder`` was born. It was initially a
shell script with only a few lines, which called debootstrap and chroot
and dpkg-buildpackage in the same run, but soon, it was decided that
that's too slow.

Yes, and it took almost an year to get things somewhat right, and in the
middle of the process, Debian 3.0 was released. Yay. Debian 3.0 wasn't
completely buildable with ``pbuilder``, but the amount of packages which
are not buildable is steadily decreasing (I hope).

And the second year of its life
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Someone wanted ``pbuilder`` to not run as root, and as User-mode-linux
has become more useful as time passed, I've started experimenting with
``pbuilder-user-mode-linux``. ``pbuilder-user-mode-linux`` has not
stayed functional as much as I would have liked, and bootstrapping
``user-mode-linux`` environment has been pretty hard, due to the quality
of user-mode-linux code or packaging at that time, which kept on
breaking network support in one way or the other.

Fifth year of pbuilder
~~~~~~~~~~~~~~~~~~~~~~

``pbuilder`` is now widely adopted as a 'almost standard' tool for
testing packages, and building packages in a pristine environment. There
are other similar tools that do similar tasks, but they do not share the
exact same goal. To commemorate this fact, ``pbuilder`` is now
co-maintained with several people.

``sbuild`` is now a well-maintained Debian package within Debian, and
with ``pbuilder`` being such a slow monster, some people prefer the
approach of sbuild. Development to use LVM-snapshots, cowloop, or
cowdancer is hoped to improve the situation somewhat.
