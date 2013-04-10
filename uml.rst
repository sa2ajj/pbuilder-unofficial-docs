Using User-mode-linux with pbuilder
===================================

It is possible to use user-mode-linux by invoking
``pbuilder-user-mode-linux`` instead of ``pbuilder``.
``pbuilder-user-mode-linux`` doesn't require root privileges, and it
uses the copy-on-write (COW) disk access method of ``User-mode-linux``
which typically makes it much faster than the traditional ``pbuilder``.

``User-mode-linux`` is a somewhat less proven platform than the standard
Unix tools which ``pbuilder`` relies on (``chroot``, ``tar``, and
``gzip``) but mature enough to support ``pbuilder-user-mode-linux``
since its version 0.59. And since then, ``pbuilder-user-mode-linux`` has
seen a rapid evolution.

The configuration of ``pbuilder-user-mode-linux`` goes in three steps:

-  Configuration of user-mode-linux

-  Configuration of rootstrap

-  Configuration of pbuilder-uml

Configuring user-mode-linux
---------------------------

user-mode-linux isn't completely trivial to set up. It would probably be
useful to acquaint yourself with it a bit before attempting to use
``rootstrap`` or ``pbuilder-user-mode-linux``. For details, read
``/usr/share/doc/uml-utilities/README.Debian`` and the
``user-mode-linux`` documentation. (It's in a separate package,
user-mode-linux-doc.)

``user-mode-linux`` requires the user to be in the uml-net group in
order to configure the network unless you are using slirp.

If you compile your own kernel, you may want to verify that you enable
TUN/TAP support, and you might want to consider the SKAS patch.

Configuring rootstrap
---------------------

``rootstrap`` is a wrapper around debootstrap. It creates a Debian disk
image for use with UML. To configure rootstrap, there are several
requirements.

-  Install the rootstrap package.

-  TUN/TAP only: add the user to the uml-net group to allow access to
   the network

   ::

       adduser dancer uml-net

-  TUN/TAP only: Check that the kernel supports the TUN/TAP interface,
   or recompile the kernel if necessary.

-  Set up ``/etc/rootstrap/rootstrap.conf``. For example, if the current
   host is 192.168.1.2, changing following entries to something like
   this seems to work.

   ::

       transport=tuntap
       interface=eth0
       gateway=192.168.1.1
       mirror=http://192.168.1.2:8081/debian
       host=192.168.1.198
       uml=192.168.1.199
       netmask=255.255.255.0

   Some experimentation with configuration and running
   ``rootstrap ~/test.uml`` to actually test it would be handy.

   Using slirp requires less configuration. The default configuration
   comes with a working example.

Configuring pbuilder-uml
------------------------

The following needs to happen:

-  Install the pbuilder-uml package.

-  Set up the configuration file ``/etc/pbuilder/pbuilder-uml.conf`` in
   the following manner. It will be different for slirp.

   ::

       MY_ETH0=tuntap,,,192.168.1.198
       UML_IP=192.168.1.199
       UML_NETMASK=255.255.255.0
       UML_NETWORK=192.168.1.0
       UML_BROADCAST=255.255.255.255
       UML_GATEWAY=192.168.1.1
       PBUILDER_UML_IMAGE="/home/dancer/uml-image"

   Also, it needs to match the rootstrap configuration.

-  Make sure BUILDPLACE is writable by the user. Change BUILDPLACE in
   the configuration file to a place where the user has access.

-  Run ``pbuilder-user-mode-linux`` to create the image.

-  Try running ``pbuilder-user-mode-linux build``.

Considerations for running pbuilder-user-mode-linux
---------------------------------------------------

``pbuilder-user-mode-linux`` emulates most of ``pbuilder``, but there
are some differences.

-  ``pbuilder-user-mode-linux`` does not support all options of
   ``pbuilder`` properly yet. This is a problem, and will be addressed
   as specific areas are discovered.

-  :file:`/tmp` is handled differently inside ``pbuilder-user-mode-linux``. In
   ``pbuilder-user-mode-linux``, :file:`/tmp` is mounted as tmpfs inside UML,
   so accessing files under :file:`/tmp` from outside user-mode-linux does not
   work. It affects options like :option:`--configfile`, and when trying to
   build packages placed under :file:`/tmp`.

Parallel running of pbuilder-user-mode-linux
--------------------------------------------

To run ``pbuilder-user-mode-linux`` in parallel on a system, there are a
few things to bear in mind.

-  The create and update methods must not be run when a build is in
   progress, or the COW file will be invalidated.

-  If you are not using slirp, user-mode-linux processes which are
   running in parallel need to have different IP addresses. Just trying
   to run the ``pbuilder-user-mode-linux`` several times will result in
   failure to access the network. But something like the following will
   work:

   ::

       for IP in 102 103 104 105; do
         xterm -e pbuilder-user-mode-linux build --uml-ip 192.168.0.$IP \
           20030107/whizzytex_1.1.1-1.dsc &
       done

   When using slirp, this problem does not exist.

Using pbuilder-user-mode-linux as a wrapper script to start up a virtual machine
--------------------------------------------------------------------------------

It is possible to use ``pbuilder-user-mode-linux`` for other uses than just
building Debian packages. ``pbuilder-user-mode-linux`` will let a user use a
shell inside the user-mode-linux ``pbuilder`` base image, and
``pbuilder-user-mode-linux`` will allow the user to execute a script inside the
image.

You can use the script to install ssh and add a new user, so that it is
possible to access inside the user-mode-linux through ssh.

Note that it is not possible to use a script from ``/tmp`` due to the
way ``pbuilder-user-mode-linux`` mounts a tmpfs at ``/tmp``.

The following example script may be useful in starting a sshd inside
user-mode-linux.

::

    #!/bin/bash

    apt-get install -y ssh xbase-clients xterm
    echo "enter root password"
    passwd
    cp /etc/ssh/sshd_config{,-}
    sed 's/X11Forwarding.*/X11Forwarding yes/' /etc/ssh/sshd_config- > /etc/ssh/sshd_config

    /etc/init.d/ssh restart
    ifconfig
    echo "Hit enter to finish"
    read
