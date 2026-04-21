Build CVMFS server for reference data
=====================================
This section gives a quick overview of the steps needed to create a new cvmfs repository to share reference data and activate it on the clients. The repository name used is ``elixir-italy.galaxy.refdata``, but it can be replaced  with the appropriate name. 

All script needed to deploy a Reference data CernVM-FS Stratum 0 are located `here <https://github.com/indigo-dc/Reference-data-galaxycloud-repository>`_.

Create CernVM-FS Repository
---------------------------
The CernVM-FS (cvmfs) relies on OverlayFS or AUFS as default storage driver. Ubuntu 16.04 natively supports OverlayFS, therefore it is used as default, to create and populate the cvmfs server.

#. Install cvmfs and cvmfs-server packages.

#. Ensure enough disk space in ``/var/spool/cvmfs`` (>50GiB). 

#. For local storage: ensure enough disk space in /srv/cvmfs.

#. Create a repository with ``cvmfs_server mkfs``.


.. Warning::

  - ``/cvmfs`` is the repository mount point, containing read-only union file system mountpoints that become writable during repository updates.

  - ``/var/spool/cvmfs`` hosts the scratch area described here, thus might consume notable disk space during repository updates. When you copy your files to ``/cvmfs/<your_repository_name>/``, they are stored in ``/var/spool/cvmfs``, therefore you have ensure enough space to this directory.

  - ``/srv/cvmfs`` is the central repository storage location. During the ``cvmfs_server publish`` procedure, your files will be moved and stored here. Therefore you have to ensure enough space here, too. This directory needs to have enough space to store all your cvmfs server contents.


.. Note::

  A complete set of reference data takes 100 GB. Our cvmfs server exploits two different volumes, one 100 GB volume mounted on ``/var/spool/cvmfs`` and one 200 GB volume for ``/srv/cvmfs``.

- To **Create** a new repository:

  ::

    cvmfs_server mkfs -w http://<stratum_zero>/cvmfs/elixir-italy.galaxy.refdata -o cvmfs elixir-italy.galaxy.refdata'

  Replace ``<stratum_zero>`` with your domain or ip address.

- **Publish** your contents to the cvfms stratum zero server:

  ::

    cvmfs_server transaction elixir-italy.galaxy.refdata
    touch /cvmfs/elixir-italy.galaxy-refdata/test-content
    cvmfs_server publish elixir-italy.galaxy.refdata

- **Periodically** resign the repository (at least every 30 days): 

  ::

    cvmfs_server resign elixir-italy.galaxy.refdata

  A resign script is located in ``/usr/local/bin/Cvmfs-stratum0-resign`` and the corresponding weekly cron job is set to ``/etc/cron.d/cvmfs_server_resign``.

  Log file is located in ``/var/log/Cvmfs-stratum0-resign.log``.

- Finally **restart** the ``apache2`` daemon.

  ::

    sudo systemctl restart apache2

The public key of the new repository is located in ``/etc/cvmfs/keys/elixir-italy.galaxy.refdata.pub``

Client configuration
--------------------
- Add the public key of the new repository to ``/etc/cvmfs/keys/elixir-italy.galaxy.refdata.pub``

- Repository configuration:

  ::

    $ cat /etc/cvmfs/config.d/elixir-italy.galaxy.refdata.conf 
    CVMFS_SERVER_URL=http://90.147.102.186/cvmfs/elixir-italy.galaxy.refdata 
    CVMFS_PUBLIC_KEY=/etc/cvmfs/keys/elixir-italy.galaxy.refdata.pub
    CVMFS_HTTP_PROXY=DIRECT

Populate a CernVM-FS Repository (with reference data)
-----------------------------------------------------

Content Publishing

#. | cvmfs_server transaction <repository name>
#. | Install content into /cvmfs/<repository name> (see `Reference data download`_ section)
#. | cvmfs_server publish <repository name>

.. Note::

   cvmfs_server publish command will take time to move your contents from ``/cvmfs`` to ``/srv/cvmfs``.

.. _ShortAnchor:

Reference data download
-----------------------
Reference data are available on Openstack Swift for public download. The list of reference data download link is `here <https://raw.githubusercontent.com/indigo-dc/Reference-data-galaxycloud-repository/master/lists/url_list.txt>`_

Furthermore, to automatically download our reference data set it is possible to use python script `refdata_download.py <https://raw.githubusercontent.com/indigo-dc/Reference-data-galaxycloud-repository/master/scripts/refdata_download.py>`_. 

The package python-pycurl is needed to satisfy refdata_download.py requirements: on Ubuntu ``sudo apt-get install python-pycurl``

Script usage
************
This script takes the ``yaml`` files as input located in ``Reference-data-galaxycloud-repository/lists/``  directory.

=======================  =================
Option                   Description
=======================  =================
``-i``, ``--input.``     Input genome list in yaml format
``-o``, ``--outdir``     Destination directory. Default ``/refdata``
``-s``, ``--space``      Subdirectory name (for cvmfs and onedata spaces). Default ``elixir-italy.galaxy.refdata``
=======================  =================

::

  /usr/bin/python refdata_download.py -i sacCer3-list.yml -o /refdata -s elixir-italy.galaxy.refdata

Available Reference data yaml file:

- at10-list.yml
- at9-list.yml
- dm2-list.yml
- dm3-list.yml
- hg18-list.yml
- hg19-list.yml
- hg38-list.yml
- mm10-list.yml
- mm8-list.yml
- mm9-list.yml
- sacCer1-list.yml
- sacCer2-list.yml
- sacCer3-list.yml

It is possible to download automatically all reference data files using the bash script ``refdata_download.sh``, which parse the python script, using as input the list file ``Reference-data-galaxycloud-repository/lists/list.txt``

::

  ./refdata_download.sh list.txt

References
----------
`CernVM-FS stratum Zero documentation <http://cvmfs.readthedocs.io/en/stable/cpt-repo.html>`_

`Nikhef wiki <https://wiki.nikhef.nl/grid/Adding_a_new_cvmfs_repository>`_
