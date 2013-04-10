Other uses of pbuilder
======================

Using pbuilder for small experiments
------------------------------------

There are cases when some small experimenting is required, and you do
not want to damage the main system, like when installing experimental
library packages, or compiling with experimental compilers. For such
cases, the ``pbuilder login`` command is available.

``pbuilder login `` is a debugging feature for ``pbuilder`` itself, but
it also allows users to have a temporary chroot.

Note that the chroot is cleaned after logging out of the shell, and
mounting file systems inside it is considered harmful.

Running little programs inside the chroot
-----------------------------------------

To facilitate using ``pbuilder`` for other uses, ``pbuilder execute`` is
available. ``pbuilder execute`` will take a script specified in the
command-line argument, and invoke the script inside the chroot.

The script can be useful for sequences of operations such as installing
ssh and adding a new user inside the chroot.
