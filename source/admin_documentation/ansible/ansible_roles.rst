Laniakea Ansible Roles
======================

Ansible automates Galaxy installation and configuration using Ansible roles. These roles make extensive use of Ansible Modules, which are the ones that do the actual work in ansible, they are what gets executed in each playbook task. Furthermore, a python scripts collection for galaxy advanced configuration is used (run by ansible).

.. note::

   All roles can be easily installed through ``ansible-galaxy``.

-------------------------
``indigo-dc.galaxycloud``
-------------------------

:Description:
	Install Galaxy Production environment, i.e. Galaxy with all needed software, PostgreSQL, NGINX, Proftpd and uWSGI. The role also installs `Galaxyctl and its API <https://github.com/Laniakea-elixir-it/galaxyctl>`_ for Galaxy management.

:Installation:
	::

	  # ansible-galaxy install indigo-dc.galaxycloud 

:Documentation:
	https://github.com/indigo-dc/ansible-role-galaxycloud

----------------------------
``indigo-dc.galaxycloud-os``
----------------------------

:Description:
	This role provides storage encryption with aes-xts-plain64 algorithm using LUKS for Galaxy instances. The role installs and run `fast-luks <https://github.com/Laniakea-elixir-it/fast-luks>`_ for storage encryption, and `LUKSctl <https://github.com/Laniakea-elixir-it/luksctl>`_ and `LUKSctl APIs <https://github.com/Laniakea-elixir-it/luksctl_api>`_ for storage management.

:Installation:
        ::

          ansible-galaxy install indigo-dc.galaxycloud-os

:Documentation:
	https://github.com/indigo-dc/ansible-role-galaxycloud-os

-------------------------------
``indigo-dc.galaxycloud-tools``
-------------------------------

:Description:
	Automated installation of tools from a Tool Shed into Galaxy. The role use the path scheme from the `indigo-dc.galaxycloud <https://github.com/indigo-dc/ansible-role-galaxycloud>`_ role. It creates a virtual environment, install ephemeris and invoke the install script to tools into Galaxy. The script stop Galaxy (if running), start a local Galaxy instance on http://localhost:8080 and install tools. The list of tools to install is provided in files/tool_list.yaml file, hosted in the `external repository <https://github.com/indigo-dc/Galaxy-flavors-recipes>`_. Workflows are also installed.


:Installation:
        ::

          ansible-galaxy install indigo-dc.galaxycloud-tools

:Documentation:
	https://github.com/indigo-dc/ansible-role-galaxycloud-tools

----------------------------------
``indigo-dc.galaxycloud-refdata``
----------------------------------

:Description:
	The role provides reference data using the CernVM File System and the corresponding Galaxy configuration.

:Installation:
        ::

          ansible-galaxy install indigo-dc.galaxycloud-refdata

:Documentation:
	https://github.com/indigo-dc/ansible-role-galaxycloud-refdata

------------------------------------
``indigo-dc.galaxycloud-fastconfig``
------------------------------------

:Description:
	Ansible role for Galaxy fast configuration on Virtual Machines with Galaxy and tools already inside, created using indigo.dc-galaxycloud role. The documentation on Galaxy Express services, which explotis this role, is: :doc:`/admin_documentation/indigo_paas_deploy/galaxy_vm`.

:Installation:
        ::

          ansible-galaxy install indigo-dc.galaxycloud-fastconfig

:Documentation:
	https://github.com/indigo-dc/ansible-role-galaxycloud-fastconfig

--------------------------------
``indigo-dc.galaxycloud_docker``
--------------------------------

:Description:
	Run Galaxy Docker containers on a Centos7 (Ubuntu 16.04) virtual machine, creating Galaxy administrator user and mounting specific Cern VM file system. The Docker engine is installed and stored with docker images on the external volume (/export).

:Installation:
        ::

          ansible-galaxy install indigo-dc.galaxycloud_docker

:Documentation:
	https://github.com/indigo-dc/ansible-role-galaxycloud-docker

-------------------------
``indigo-dc.cvmfs-client``
-------------------------

:Description:
	Ansible role to install CernVM-FS Client.

:Installation:
        ::

          ansible-galaxy install indigo-dc.cvmfs-client

:Documentation:
	https://github.com/indigo-dc/ansible-role-cvmfs-client

-------------------------
``indigo-dc.cvmfs-server``
-------------------------

:Description:
	Ansible role to install CernVM FS Server.

:Installation:
        ::

          ansible-galaxy install indigo-dc.cvmfs-server

:Documentation:
	https://github.com/indigo-dc/ansible-role-cvmfs-server
