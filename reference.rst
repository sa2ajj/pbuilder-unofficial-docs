Reference materials
===================

Directory structure outside the chroot
--------------------------------------

+--------------------------------------+--------------------------------------+
| Directory                            | Meaning                              |
+======================================+======================================+
| ``/etc/pbuilderrc``                  | configuration file                   |
+--------------------------------------+--------------------------------------+
| ``/usr/share/pbuilder/pbuilderrc``   | Default configuration                |
+--------------------------------------+--------------------------------------+
| ``/var/cache/pbuilder/base.tgz``     | Default location pbuilder uses for   |
|                                      | base.tgz, the tar-ball containing a  |
|                                      | basic Debian installation with only  |
|                                      | the build-essential packages.        |
+--------------------------------------+--------------------------------------+
| ``/var/cache/pbuilder/build/PID/``   | Default location pbuilder uses for   |
|                                      | chroot                               |
+--------------------------------------+--------------------------------------+
| ``/var/cache/pbuilder/aptcache``     | Default location ``pbuilder`` will   |
|                                      | use as apt cache, to store deb       |
|                                      | packages required during             |
|                                      | ``pbuilder`` build.                  |
+--------------------------------------+--------------------------------------+
| ``/var/cache/pbuilder/ccache``       | Default location ``pbuilder`` will   |
|                                      | use as cache location                |
+--------------------------------------+--------------------------------------+
| ``/var/cache/pbuilder/result``       | Default location ``pbuilder`` puts   |
|                                      | the deb files and other files        |
|                                      | created after build                  |
+--------------------------------------+--------------------------------------+
| ``/var/cache/pbuilder/pbuilder-umlre | Default location                     |
| sult``                               | ``pbuilder-user-mode-linux`` puts    |
|                                      | the deb files and other files        |
|                                      | created after build                  |
+--------------------------------------+--------------------------------------+
| ``/var/cache/pbuilder/pbuilder-mnt`` | Default location                     |
|                                      | ``pbuilder-user-mode-linux`` uses    |
|                                      | for mounting the COW file system,    |
|                                      | for chrooting.                       |
+--------------------------------------+--------------------------------------+
| ``/tmp``                             | ``pbuilder-user-mode-linux`` will    |
|                                      | mount tmpfs for work.                |
+--------------------------------------+--------------------------------------+
| ``${HOME}/tmp/PID.cow``              | ``pbuilder-user-mode-linux`` use     |
|                                      | this directory for location of COW   |
|                                      | file system.                         |
+--------------------------------------+--------------------------------------+
| ``${HOME}/uml-image``                | ``pbuilder-user-mode-linux`` use     |
|                                      | this directory for user-mode-linux   |
|                                      | full disk image.                     |
+--------------------------------------+--------------------------------------+

Table: Directory Structure outside the chroot

Directory structure inside the chroot
-------------------------------------

+--------------------------------------+--------------------------------------+
| Directory                            | Meaning                              |
+======================================+======================================+
| ``/etc/mtab``                        | symlink to ``/proc/mounts``.         |
+--------------------------------------+--------------------------------------+
| ``/tmp/buildd``                      | Default place used in ``pbuilder``   |
|                                      | to place the Debian package to be    |
|                                      | processed.                           |
|                                      | ``/tmp/buildd/packagename-version/`` |
|                                      | will be the root directory of the    |
|                                      | package being processed. HOME        |
|                                      | environment variable is set to this  |
|                                      | value inside chroot by               |
|                                      | pbuilder-buildpackage.               |
|                                      | ``--inputfile`` will place files     |
|                                      | here.                                |
+--------------------------------------+--------------------------------------+
| ``/runscript``                       | The script passed as an argument to  |
|                                      | ``pbuilder`` execute is passed on.   |
+--------------------------------------+--------------------------------------+
| ``/tmp/hooks``                       | The location of hooks.               |
+--------------------------------------+--------------------------------------+
| ``/var/cache/apt/archives``          | ``pbuilder`` copies the content of   |
|                                      | this directory to and from the       |
|                                      | aptcache directory of outside        |
|                                      | chroot.                              |
+--------------------------------------+--------------------------------------+
| ``/var/cache/pbuilder/ccache``       | ``pbuilder`` bind-mounts this        |
|                                      | directory for use by ccache.         |
+--------------------------------------+--------------------------------------+
| ``/tmp/XXXX``                        | ``pbuilder-user-mode-linux`` uses a  |
|                                      | script in ``/tmp`` to bootstrap into |
|                                      | user-mode-linux                      |
+--------------------------------------+--------------------------------------+

Table: Directory Structure inside the chroot
