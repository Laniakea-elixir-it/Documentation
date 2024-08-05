Galaxycloud-os
==============
This role provides advanced storage options for Galaxy instances.

.. Warning::

   Run indigo-dc.galaxycloud-os before indigo-dc.galaxycloud, setting the variable ``enable_storage_advanced_options`` to ``true``.

It is possible to select three different storage options using the ``os_storage`` ansible role variable.

====================  =========================
Storage provider      Description
====================  =========================
Iaas                  IaaS block storage volume is attached to the instance and Galaxy is configured.
onedata               Onedata space is mounte through oneclient and Galaxy is configured.
encryption            IaaS block storage volume is encrypted with aes-xts-plain64 algorithm using LUKS.
====================  =========================

Path configuration for Galaxy is then correctly set, depending on the storage solution selected, replacing the indigo-dc.galaxycloud path recipe (with the ``enable_storage_advanced_options`` set to ``true``).

The role exploits the ``galaxyctl_libs`` (see :doc:`script_galaxyctl_libs`) for LUKS and onedata volumes management .

LUKS encryption
---------------
For a detailed description of LUKS encryption used and scripts, see section :doc:`FS_encryption`.

Dependencies
------------

For LUKS encryption the ansible role install ``cryptsetup``.

For onedata reference data provider, the role depends on indigo-dc.oneclient role, to install oneclient:

::

  - hosts: servers
    roles:
      - role: indigo-dc.oneclient
        when: os_storage == 'onedata'

Variables
---------
The Galaxy path variables are the same of indigo-dc.galaxycloud.

Path
****
``galaxy_user``: set linux user to launch the Galaxy portal (default: ``galaxy``).

``GALAXY_UID``: set user UID (default: ``4001``).

``galaxy_FS_path``: path to install Galaxy (default: ``/home/galaxy``).

``galaxy_directory``: Galaxy directory (usually galaxy or galaxy-dist, default ``galaxy``).

``galaxy_install_path``: Galaxy installation directory (default: ``/home/galaxy/galaxy``).

``export_dir``: Galaxy userdata are stored here (defatult: ``/export``).

``galaxy_custom_config_path``: Galaxy custom configuration files path (default: ``/etc/galaxy``).

``galaxy_custom_script_path``: Galaxy custom script path (defautl: ``/usr/local/bin``).

``galaxy_log_path``: log file directory (default: ``/var/log/galaxy``).

``galaxy_instance_key_pub``: instance ssh public key to configure <galaxy_user> access.

``galaxy_lrms``: enable  Galaxy virtual elastic cluster support. Currently supported local and slurm (default: ``local``, possible values: ``local, slurm``).

Main options
************
``GALAXY_ADMIN_EMAIL``: Galaxy administrator e-mail address

Isolation specific vars
***********************
``os_storage``: takes three possible values:

  #. ``IaaS``: standard IaaS block storage volume.
  #. ``onedata``: Onedata space is mounted for user data.
  #. ``download``: IaaS block storage volume encrypted with ``aes-xts-plain64`` is mounted.

Onedata
*******

``onedata_dir``: onedata mountpoint. (default: ``/onedata``).

.. Note::

  Once onedata space is mounted, files existing before mount operation, will not be available until volume umount. For this reason we set it to ``/onedata`` to a differet path.

``onedatactl_config_file``: set onedatactl config file (default: ``{{ galaxy_custom_config_path }}/onedatactl.ini``).

``userdata_oneprovider``: set onedata oneprovider.

``userdata_token``: set onedata access token.

``userdata_space``: set space name.

Encryption
**********

``luks_lock_dir``: set luks lock file directory (default: ``/var/run/fast_luks``).

``luks_success_file``: set success file. It signals to ansible to proceed (default: ``/var/run/fast-luks.success``).

``luks_log_path``: set LUKS log path (default: ``/var/log/galaxy``).

``luks_config_file``: set luksctl configuration file (default: ``/etc/galaxy/luks-cryptdev.ini``).

``wait_timeout``: time to waint encryption password (default: 5 hours).

``mail_from``: set mail from field (default: ``GalaxyCloud@elixir-italy.org``).

``mail_subject``:  with the instructions to access and encrypt the volume is sent to the user (default: ``[ELIXIR-ITALY] GalaxyCloud encrypt password``).

LUKS specific variables
***********************

``cipher_algorithm``: set cipher algorithm (default: ``aes-xts-plain64``).

``keysize``: set key size (default: ``256``).

``hash_algorithm``: set hash algorithm (default: ``sha256``).

``device``: set device to mount (default: ``/dev/vdb``)

``cryptdev``: set device mapper name (default:  ``/dev'crypt``).

``mountpoint``: set mount point. Usually the same of ``export_dir`` (default:  ``{{ export_dir }}``).

``filesystem``: set file system (default: ``ext4``).

Create block file:

#. https://wiki.archlinux.org/index.php/Dm-crypt/Device_encryption
#. https://wiki.archlinux.org/index.php/Dm-crypt/Drive_preparation
#. https://wiki.archlinux.org/index.php/Disk_encryption#Preparing_the_disk

Before encrypting a drive, it is recommended to perform a secure erase of the disk by overwriting the entire drive with random data.

To prevent cryptographic attacks or unwanted file recovery, this data is ideally indistinguishable from data later written by dm-crypt.

``paranoic_mode``: to enable block storage low level deletion set to true (default: ``false``).

Example Playbook
----------------

IaaS configuration:

::

  - hosts: servers
    roles:
      - role: indigo-dc.galaxycloud-os
        os_storage: 'IaaS'
        GALAXY_ADMIN_EMAIL: "admin@server.com"
        galaxy_instance_key_pub: '<your_ssh_public_key>'

      - role: indigo-dc.galaxycloud
        GALAXY_ADMIN_EMAIL: "admin@server.com"
        GALAXY_ADMIN_USERNAME: "admin"
        GALAXY_VERSION: "release_17.05"
        galaxy_instance_key_pub: "<your_ssh_public_key>"
        enable_storage_advanced_options: true

Onedata configuration:

::

  - hosts: servers
    roles:
      - role: indigo-dc.galaxycloud-os
        os_storage: 'onedata'
        GALAXY_ADMIN_EMAIL: "admin@server.com"
        userdata_provider: 'oneprovider2.cloud.ba.infn.it'
        userdata_token: '<your_access_token>'
        userdata_space: '<your_onedata_space>'
        galaxy_instance_key_pub: '<your_ssh_public_key>'

      - role: indigo-dc.galaxycloud
        GALAXY_ADMIN_EMAIL: "admin@server.com"
        GALAXY_ADMIN_USERNAME: "admin"
        GALAXY_VERSION: "release_17.05"
        galaxy_instance_key_pub: "<your_ssh_public_key>"
        enable_storage_advanced_options: true

LUKS configuration:

::

  - hosts: servers
    roles:
      - role: indigo-dc.galaxycloud-os
        os_storage: 'encryption'
        GALAXY_ADMIN_EMAIL: "admin@server.com"
        galaxy_instance_key_pub: '<your_ssh_public_key>'

      - role: indigo-dc.galaxycloud
        GALAXY_ADMIN_EMAIL: "admin@server.com"
        GALAXY_ADMIN_USERNAME: "admin"
        GALAXY_VERSION: "release_17.05"
        galaxy_instance_key_pub: "<your_ssh_public_key>"
        enable_storage_advanced_options: true

References
----------
Galaxy: https://galaxyproject.org/

Apache licence: http://www.apache.org/licenses/LICENSE-2.0

