Manage CVMFS
============

The CernVM-File System (conversely cvmfs) provides a scalable, reliable and low-maintenance software distribution service. It was developed to assist High Energy Physics (HEP) collaborations to deploy software on the worldwide distributed computing infrastructure used to run data processing applications.

CernVM-FS is implemented as a POSIX read-only file system in user space (a FUSE module). When initially mounted, CVMFS does not consume any local disk space on the client (in this case, your Galaxy server). Instead, as files are accessed, they are pulled from the server to a local disk-based cache of a configurable size. The reference data Files and directories are hosted on standard web servers and mounted on ``/cvmfs`` directory

For example, listing the CVMFS ``elixir-italy.galaxy.refdata`` will results in:

::

  $ ls -l /cvmfs/elixir-italy.galaxy.refdata/
  total 60
  drwxr-xr-x. 5 cvmfs cvmfs 4096 May 21 20:10 at10
  drwxr-xr-x. 5 cvmfs cvmfs 4096 May 21 20:10 at9
  drwxr-xr-x. 3 cvmfs cvmfs 4096 May 21 20:10 dm2
  drwxr-xr-x. 7 cvmfs cvmfs 4096 May 21 20:11 dm3
  drwxr-xr-x. 7 cvmfs cvmfs 4096 May 21 20:15 hg18
  drwxr-xr-x. 7 cvmfs cvmfs 4096 May 21 18:36 hg19
  drwxr-xr-x. 7 cvmfs cvmfs 4096 May 21 20:18 hg38
  drwxr-xr-x. 7 cvmfs cvmfs 4096 May 21 20:22 mm10
  drwxr-xr-x. 3 cvmfs cvmfs 4096 May 21 20:22 mm8
  drwxr-xr-x. 7 cvmfs cvmfs 4096 May 21 20:25 mm9
  -rw-r--r--. 1 cvmfs cvmfs   57 May 21 18:31 new_repository
  drwxr-xr-x. 3 cvmfs cvmfs 4096 May 21 20:25 sacCer1
  drwxr-xr-x. 3 cvmfs cvmfs 4096 May 21 20:25 sacCer2
  drwxr-xr-x. 7 cvmfs cvmfs 4096 May 21 20:25 sacCer3
  -rw-r--r--. 1 cvmfs cvmfs    0 May 21 18:31 test-content

.. note::

   The files hosted on a CVMFS repository are pulled from the server only if required, resulting in an empty directory if the file are not required. For example, just listing the directory content will cause the files to be mounted.

Cvmfs client setup
******************

CVMFS is installed by default on each Galaxy instance (CentOS 7 or Ubuntu 16.04). The public key is installed in ``/etc/cvmfs/keys/``. The ``/etc/cvmfs/default.local`` file is also already configured. The ``cvmfs_config probe`` command mount the cvmfs volume to ``/cvmfs``.

======================  ======================
Description             Command
======================  ======================
check configuration     cvmfs_config chksetup
mount volume            cvmfs_config probe
umount volume           cvmfs_config umount <refdata_repository_name>
reload repository       cvmfs_config reload <refdata_repository_name>
======================  ======================

.. Note::

   If mount fails, try to restart autofs with ``sudo service autofs restart``.

.. Note::

   CVMFS commands require root privileges

The CVMFS repositoy can be mount also using the ``mount`` command to a specific mount point:

::

  $ sudo mount -t cvmfs elixir-italy.galaxy.refdata /refdata/elixir-italy.galaxy.refdata
  CernVM-FS: running with credentials 994:990
  CernVM-FS: loading Fuse module... done

  $ ls /refdata/elixir-italy.galaxy.refdata/
  at10  at9  dm2  dm3  hg18  hg19  hg38  mm10  mm8  mm9  new_repository  sacCer1  sacCer2  sacCer3  test-content

Troubleshooting
***************

After an instance reboot, CVMFS is automatically restarted. If this does not happen:

::

  $ sudo cvmfs_config_probe
  Probing /cvmfs/elixir-italy.galaxy.refdata... Failed!

A reload of the config could be able to fix the `problem <https://wiki.chipp.ch/twiki/bin/view/CmsTier3/IssueCvmfsFailsToMount>`_:

::

  $ sudo cvmfs_config reload elixir-italy.galaxy.refdata
  Connecting to CernVM-FS loader... done
  Entering maintenance mode
  Draining out kernel caches (60s)
  Blocking new file system calls
  Waiting for active file system calls
  Saving inode tracker
  Saving chunk tables
  Saving inode generation
  Saving open files counter
  Unloading Fuse module
  Re-Loading Fuse module
  Restoring inode tracker...  done
  Restoring chunk tables...  done
  Restoring inode generation...  done
  Restoring open files counter...  done
  Releasing saved glue buffer
  Releasing chunk tables
  Releasing saved inode generation info
  Releasing open files counter
  Activating Fuse module

If the file system appears to be hanging, it might have been interrupted during a reload operation. Try to run ``sudo cvmfs_config killall`` and then again ``sudo cvmfs_config_probe``.

References
**********

`CernVM-FS <https://cernvm.cern.ch/portal/filesystem>`_

`CVMFS documentation <http://cvmfs.ireadthedocs.io/en/stable/>`_

`Debugging CVMFS <https://cernvm.cern.ch/portal/filesystem/debugmount>`_
