Experimental or wishlist features of pbuilder
=============================================

There are some advanced features, above that of the basic feature of
``pbuilder``, for some specific purposes.

Using LVM
---------

LVM2 has a useful snapshot function that features Copy-on-write images.
That could be used for ``pbuilder`` just as it can be used for the
user-mode-linux ``pbuilder`` port. lvmpbuilder script in the examples
directory implements such port. The scripts and documentation can be
found under ``/usr/share/doc/pbuilder/examples/lvmpbuilder/``.

Using cowdancer
---------------

``cowdancer`` allows copy-on-write semantics on file system using hard
links and hard-link-breaking-on-write tricks. ``pbuilder`` using
``cowdancer`` seems to be much faster and it is one ideal point for
improvement. ``cowbuilder``, a wrapper for ``pbuilder`` for using
``cowdancer`` is available from ``cowdancer`` package since 0.14

Example command-lines for cowbuilder look like the following.

::

    # cowbuilder --create --distribution sid
    # cowbuilder --update --distribution sid
    # cowbuilder --build XXX.dsc

It is also possible to use cowdancer with pdebuild command. Specify with
command-line option ``--pbuilder`` or set it in PDEBUILD\_PBUILDER
configuration option.

::

    $ pdebuild --pbuilder cowbuilder

Using pbuilder without tar.gz
-----------------------------

The :option:`--no-targz` option of ``pbuilder`` will allow usage of ``pbuilder`` in a
different way from conventional usage. It will try to use an existing
chroot, and will not try to clean up after working on it. It is an
operation mode more like ``sbuild``.

It should be possible to create base chroot images for ``dchroot`` with
the following commands:

::

    # pbuilder create --distribution lenny --no-targz --basetgz /chroot/lenny
    # pbuilder create --distribution squeeze --no-targz --basetgz /chroot/squeeze
    # pbuilder create --distribution sid --no-targz --basetgz /chroot/sid

Using pbuilder in a vserver
---------------------------

It is possible to use ``pbuilder`` in a vserver environment. This
requires either vserver-patches in version 2.1.1-rc14 or higher, or a
Linux kernel version 2.6.16 or higher.

To use ``pbuilder`` in a vserver, you need to set the ``secure_mount``
``CAPS`` in the ``ccapabilities`` of this vserver.

Usage of ccache
---------------

By default ``pbuilder`` will use the C compiler cache ``ccache`` to
speed up repeated builds of the same package (or packages that compile
the same files multiple times for some reason). Using ``ccache`` can
speed up repeated building of large packages dramatically, at the cost
of some disk space and bookkeeping.

To disable usage of ``ccache`` with ``pbuilder``, you should unset
:envvar:`CCACHEDIR` in your :file:`pbuilderrc` file.
