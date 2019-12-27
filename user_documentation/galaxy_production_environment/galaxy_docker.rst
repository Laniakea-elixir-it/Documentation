|galaxy_docker| instance
========================

The |project_name| Galaxy Docker application run a Galaxy Docker container inside a Centos 7 virtual machine. The `Official Galaxy Docker image <https://github.com/bgruening/docker-galaxy-stable>`_ is used. Currently, |project_name| supports the following Docker images:

- `bgruening/galaxy-stable <https://hub.docker.com/r/bgruening/galaxy-stable/tags>`_
- `laniakeacloud/galaxy-covacs <https://hub.docker.com/r/laniakeacloud/galaxy-covacs/tags>`_
- `laniakeacloud/galaxy-gdc_somatic_variant <https://hub.docker.com/r/laniakeacloud/galaxy-gdc_somatic_variant/tags>`_
- `bgruening/galaxy-rna-workbench <https://hub.docker.com/r/bgruening/galaxy-rna-workbench/tags>`_
- `laniakeacloud/galaxy-epigen <https://hub.docker.com/r/laniakeacloud/galaxy-epigen/tags>`_

.. note::

   Docker is configured to install all docker-engine files on ``/export``, i.e. in the external storage.

Configuration files
-------------------

The Docker configuration is slighty customized to make the Galaxy experience as similar as possible to the one on the virtual machine.

- ``/etc/galaxy/.myenv.sh``: file with the environment variables of the Docker container.

  The customized variables are:

  ``GALAXY_CONFIG_TOOL_DATA_TABLE_CONFIG_PATH``: tool_data_table_conf.xml specific for the galaxy flavour (see section :doc:`/user_documentation/galaxy/galaxy_flavours`)

  ``GALAXY_CONFIG_ADMIN_USERS``: admin_user - the email selected in the laniakea dashboard

  ``GALAXY_CONFIG_BRAND``: Galaxy brand - the **Instance description** inserted in the laniakea dashboard

  ``GALAXY_CONFIG_REQUIRE_LOGIN``: true - avoid anonymous login.

  ``GALAXY_CONFIG_ALLOW_USER_CREATION``: true - allow user creation.

  ``GALAXY_CONFIG_ALLOW_USER_IMPERSONATION``: false - allow user impersonation.

  ``GALAXY_CONFIG_NEW_USER_DATASET_ACCESS_ROLE_DEFAULT_PRIVATE``: true - By default, users' data will be public, but setting this to True will cause it to be private.

  ``GALAXY_CONDA_PREFIX``: path to _conda prefix

  ``GALAXY_CONFIG_CONDA_AUTO_INIT``: true - conda auto-start

  ``GALAXY_CONFIG_CONDA_AUTO_INSTALL``: true - conda auto-install


- ``/etc/galaxy/tool_data_tables``: directory with the tool_data_table_conf.xml files. A detailed description of Laniakea Galaxy flavours configuration for the reference data is here: :doc:`/user_documentation/galaxy/galaxy_flavours`.


CVMFS configuration
-------------------

The CVMFS repository selected in the Lanikaea dashboard is automatically configured and mounted inside the docker directory ``/cvmfs``. The corresponding configuration files are in the directory ``/etc/cvmfs``.  

Galaxy docker usage
-------------------

Galaxy docker logs
******************

SSH login in the virtual machine and type:

::

  $ sudo docker logs --tail 200 -f galaxydocker

Enter in the Docker
*******************

In order to access to the Galaxy container, SSH login in the virtual machine and execute the following command:

::

  $ sudo docker exec -it galaxydocker bash

Main directories in the Docker
******************************

Main Galaxy directories inside the Docker container are in ``/export``:

- ftp: ``/export/ftp``
- database: ``/export/database``
- conda: ``/export/tool_deps/_conda``

Check Galaxy configuration
**************************

In order to see the Galaxy Docker configuration, SSH login in the virtual machine and execute the following command:

::

  $ sudo docker exec -it galaxydocker echo $GALAXY_CONFIG


Galaxy Docker usage tutorial
----------------------------

.. raw:: html

   <a href="https://asciinema.org/a/273497" target="_blank"><img src="https://asciinema.org/a/273497.svg" /></a>
