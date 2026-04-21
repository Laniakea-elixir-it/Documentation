Cvmfs_client
============

Ansible role to install CernVM-FS Client.

Requirements
------------

Python is required on host to run ansible.

The apt ansible module requires the following packages on host to run:

- python-apt (python 2)

Variables
---------
``server_url``: set cvmfs server url (e.g. ip address or domain).

``repository_name``: set cvmfs server repository name (default: ``elixir-italy.galaxy.refdata``).

``cvmfs_server_url``: sert cvmfs server complete url (default: ``'http://{{ server_url }}/cvmfs/{{ repository_name }}``).

``cvmfs_public_key_path``: set path for cvmfs keys (default: ``/etc/cvmfs/keys``).

``cvmfs_public_key``: set cvfms public key, usually `<repository_name.pub>` (defatul: ``{{ repository_name }}.pub``).

``cvmfs_public_key_list_files``:  list of ``*.pub`` files with the key to the repository to be mounted.

``public_key_src_path``: set cvmfs public key temporary path (default: ``/tmp``).

``proxy_url``: set proxy name (default: ``DIRECT``).

``proxy_port``: set proxy port (default: ``80``).

``cvmfs_http_proxy``: set proxy complete url (default: ``http://{{ proxy_url }}:{{ proxy_port }}``).

``cvmfs_mountpoint``: set cvmfs mount point (default: ``/cvmfs``, for reference data ``/refdata``). If set to ``/cvmfs`` the role will use ``cvmfs_config probe`` to mount the repository.

``add_fstab_entry``: add fstab entry to automatically mount the repository (default: ``true``).

Example Playbook
----------------
The role takes as input parameters the CernVM-FS server location details (stratum 0 address, public key and mount point).

::

  - hosts: servers
    roles:
      - role: indigo-dc.cvmfs-client
        server_url: '90.147.102.186'
        repository_name: 'elixir-italy.galaxy.refdata'
        cvmfs_public_key: 'elixir-italy.galaxy.refdata.pub'
        proxy_url: 'DIRECT'
        proxy_port: '80'
        cvmfs_mountpoint: '/refdata'
        when:  refdata_provider_type == 'cvmfs'

References
----------

Official cvmfs documentation: http://cvmfs.readthedocs.io/en/stable/cpt-repo.html

NIKHEF documentation: https://wiki.nikhef.nl/grid/Adding_a_new_cvmfs_repository
