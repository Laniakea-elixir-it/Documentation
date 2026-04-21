Custom types
============

GalaxyPortal
------------

Galaxy portal installation and configuration is entrusted to the GalaxyPortal custom type.

::

  tosca.nodes.indigo.GalaxyPortal:
    derived_from: tosca.nodes.WebServer

It is composed by the following sections:

--------------
``properties``
--------------

Galaxy input parameters are listed in the **properties** section:

::

  properties:
    admin_email:
      type: string
      description: email of the admin user
      default: admin@admin.com
      required: false
    admin_api_key:
      type: string
      description: key to access the API with admin role
      default: not_very_secret_api_key
      required: false
    user:
      type: string
      description: username to launch the galaxy daemon
      default: galaxy
      required: false
    install_path:
      type: string
      description: path to install the galaxy tool
      default: /home/galaxy/galaxy
      required: false
    export_dir:
      type: string
      description: path to store galaxy data
      default: /export
      required: false
    version:
      type: string
      description: galaxy version to install
      default: master
      required: false
    instance_description:
      type: string
      description: galaxy instance description
      default: "INDIGO Galaxy test"
    instance_key_pub:
      type: string
      description: galaxy instance ssh public key
      default: your_ssh_public_key
    flavor:
      type: string
      description: name of the Galaxy flavor
      required: false
      default: galaxy-no-tools
    reference_data:
      type: boolean
      description: Install Reference data
      default: true
      required: false

.. Note::

   The ``export_dir`` property is able to set Galaxy storage location. On single VMs it is set to ``/export``, while on Cluster it has to be set to ``/home/export``, allowing for data sharing.

----------------
``requirements``
----------------

The LRMS, e.g.  local, torque, slurm, sge, condor, mesos, is specified in the **requirements** section:

::

  requirements:
    - lrms:
        capability: tosca.capabilities.indigo.LRMS
        node: tosca.nodes.indigo.LRMS.FrontEnd
        relationship: tosca.relationships.HostedOn

-------------
``artifacts``
-------------

The needed Ansible roles, installed using ansible-galaxy, are listed in the **artifacts** section:

::

  artifacts:
    nfs_role:
      file: indigo-dc.nfs
      type: tosca.artifacts.AnsibleGalaxy.role
    galaxy_role:
      file: mtangaro.galaxycloud,master
      type: tosca.artifacts.AnsibleGalaxy.role

--------------
``interfaces``
--------------

The Ansible role is called with its input parameters:

::

  interfaces:
    Standard:
      configure:
        implementation: https://raw.githubusercontent.com/indigo-dc/tosca-types/v3.0.1/artifacts/galaxy/galaxy_install.yml
        inputs:
          galaxy_install_path: { get_property: [ SELF, install_path ] }
          galaxy_user: { get_property: [ SELF, user ] }
          galaxy_admin: { get_property: [ SELF, admin_email ] }
          galaxy_admin_api_key: { get_property: [ SELF, admin_api_key ] }
          galaxy_lrms: { get_property: [ SELF, lrms, type ] }
          galaxy_version: { get_property: [ SELF, version ] }
          galaxy_instance_description: { get_property: [ SELF, instance_description ] }
          galaxy_instance_key_pub:  { get_property: [ SELF, instance_key_pub ] }
          export_dir: { get_property: [ SELF, export_dir ] }
          galaxy_flavor: { get_property: [ SELF, flavor ] }
          get_refdata: { get_property: [ SELF, reference_data ] }

The artifact, called in the ``implementation`` line, is located on github `tosca-types/artifacts/galaxy/galaxy_install.yml <https://raw.githubusercontent.com/indigo-dc/tosca-types/v3.0.1/artifacts/galaxy/galaxy_install.yml>`_

::

 ---
 - hosts: localhost
   connection: local
   roles:
     - role: indigo-dc.galaxycloud
       GALAXY_VERSION: "{{ galaxy_version }}"
       GALAXY_ADMIN_EMAIL: "{{ galaxy_admin }}"
       GALAXY_ADMIN_API_KEY: "{{ galaxy_admin_api_key }}"


GalaxyPortalAndStorage
----------------------

GalaxyPortalAndStorage custom type inherits its properties from GalaxyPortal and extends its functionalities for the **storage encryption**:

::

  tosca.nodes.indigo.GalaxyPortalAndStorage:
    derived_from: tosca.nodes.indigo.GalaxyPortal

--------------
``properties``
--------------

The inputs needed to enable the storage encryption and the Hashicorp Vault key management are:

::

  properties:
    storage_encryption:
      type: boolean
      description: Enable storage encryption using Vault to store secrets and LUKS to encrypt
      default: false
      required: true
    vault_url:
      type: string
      description: Hashicorp Vault server url
      default: vault_url
      required: false
    vault_wrapping_token:
      type: string
      description: Vault Wrapping token to write secret
      default: not_a_valid_token
      required: false
    vault_secret_path:
      type: string
      description: Vault path to store secret
      default: path_to_secret
      required: false
    vault_secret_key:
      type: string
      description: Vault secret key name
      default: secret_key_name
      required: false
    wn_ips:
      type: list
      entry_schema:
        type: string
      description: List of IPs of the WNs
      required: false
      default: []

-------------
``artifacts``
-------------

Here the indigo-dc.galaxycloud-os is the ansible role entrusted of file system encryption:

::

  artifacts:
    nfs_role:
      file: indigo-dc.nfs
      type: tosca.artifacts.AnsibleGalaxy.role
    galaxy_os_role:
      file: indigo-dc.galaxycloud-os
      type: tosca.artifacts.AnsibleGalaxy.role
    galaxy_role:
      file: mtangaro.galaxycloud
      type: tosca.artifacts.AnsibleGalaxy.role

--------------
``interfaces``
--------------

The Ansible role is called with its input parameters:

::

  interfaces:
    Standard:
      configure:
        implementation: https://raw.githubusercontent.com/indigo-dc/tosca-types/v3.0.1/artifacts/galaxy/galaxy_os_install.yml
        inputs:
          storage_encryption: { get_property: [ SELF, storage_encryption ] }
          vault_url: { get_property: [ SELF, vault_url ] }
          vault_wrapping_token: { get_property: [ SELF, vault_wrapping_token ] }
          vault_secret_path: { get_property: [ SELF, vault_secret_path ] }
          vault_secret_key: { get_property: [ SELF, vault_secret_key ] }
          wn_ips: { get_property: [ SELF, wn_ips ] }
          galaxy_install_path: { get_property: [ SELF, install_path ] }
          galaxy_user: { get_property: [ SELF, user ] }
          galaxy_admin: { get_property: [ SELF, admin_email ] }
          galaxy_admin_api_key: { get_property: [ SELF, admin_api_key ] }
          galaxy_lrms: { get_property: [ SELF, lrms, type ] }
          galaxy_version: { get_property: [ SELF, version ] }
          galaxy_instance_description: { get_property: [ SELF, instance_description ] }
          galaxy_instance_key_pub:  { get_property: [ SELF, instance_key_pub ] }
          export_dir: { get_property: [ SELF, export_dir ] }
          galaxy_flavor: { get_property: [ SELF, flavor ] }
          get_refdata: { get_property: [ SELF, reference_data ] }

The artifact includes indigo-dc.galaxycloud-os and indigo-dc.galaxycloud call. 

::

   ---
   - hosts: localhost
     connection: local
     roles:
       - role: indigo-dc.galaxycloud-os
         GALAXY_ADMIN_EMAIL: "{{ galaxy_admin }}"

       - role: indigo-dc.galaxycloud
         GALAXY_VERSION: "{{ galaxy_version }}"
         GALAXY_ADMIN_EMAIL: "{{ galaxy_admin }}"
         GALAXY_ADMIN_API_KEY: "{{ galaxy_admin_api_key }}"
         enable_storage_advanced_options: true # true only with indigo-dc.galaxycloud-os


.. Note::

   The option ``enable_storage_advanced_options`` has to be set to ``true``, leaving storage configuration to indigo-dc.galaxycloud-os.

GalaxyShedTool
--------------

This custom type is used to install tools on Galaxy.

::

  tosca.nodes.indigo.GalaxyShedTool:
    derived_from: tosca.nodes.WebApplication

--------------
``properties``
--------------

The inputs needed to install tools on Galaxy are:

::

  properties:
    flavor:
      type: string
      description: name of the Galaxy flavor
      required: true
      default: galaxy-no-tools
    admin_api_key:
      type: string
      description: key to access the API with admin role
      default: not_very_secret_api_key
      required: false
    version:
      type: string
      description: galaxy version installed
      default: master
      required: false
    reference_data:
      type: boolean
      description: Install Reference data
      default: true
      required: false

----------------
``requirements``
----------------

This custom types requires to be run on a Host with Galaxy already installed before tools installation.

::

  requirements:
    - host:
        capability: tosca.capabilities.Container
        node: tosca.nodes.indigo.GalaxyPortal
        relationship: tosca.relationships.HostedOn

-------------
``artifacts``
------------

Then the Indigo-dc.galaxy-tools role is installed:

::

  artifacts:
    galaxy_role:
      file: indigo-dc.galaxy-tools,master
      type: tosca.artifacts.AnsibleGalaxy.role

--------------
``interfaces``
--------------

Finally, ansible is called:

::

  interfaces:
    Standard:
      configure:
        implementation: https://raw.githubusercontent.com/indigo-dc/tosca-types/v3.0.1/artifacts/galaxy/galaxy_tools_configure.yml
        inputs:
          galaxy_flavor: { get_property: [ SELF, flavor ] }
          galaxy_admin_api_key: { get_property: [ HOST, admin_api_key ] }
          galaxy_version: { get_property: [ SELF, version ] }
          get_refdata: { get_property: [ SELF, reference_data ] }

to install tools:

::

  ---
  - hosts: localhost
    connection: local
    roles:
      - { role: indigo-dc.galaxycloud-tools, GALAXY_VERSION: '{{ galaxy_version }}', when: galaxy_flavor != 'galaxy-no-tools' }

GalaxyReferenceData
-------------------

The ReferenceData custom type configure Galaxy to retrieve the reference data from a CernVM-FS repository.

::

  tosca.nodes.indigo.GalaxyReferenceData:
    derived_from: tosca.nodes.WebApplication

--------------
``properties``
--------------

The ReferenceData input parameters are:

::

  properties:
    reference_data:
      type: boolean
      description: Install Reference data
      default: true
      required: true
    refdata_cvmfs_configuration:
      type: string
      description: Configure cvmfs or load preconfigured repository
      default: 'cvmfs_preconfigured'
      required: false
    refdata_cvmfs_repository_name:
      type: string
      description: CernVM-FS repository name
      default: 'elixir-italy.galaxy.refdata'
      required: false
    refdata_cvmfs_server_url:
      type: string
      description: CernVM-FS server, replica or stratum-zero
      default: 'server_url'
      required: false
    refdata_cvmfs_key_file:
      type: string
      description: CernVM-FS public key
      default: 'not_a_key'
      required: false
    refdata_cvmfs_proxy_url:
      type: string
      description: CernVM-FS proxy url
      default: 'DIRECT'
      required: false
    refdata_cvmfs_proxy_port:
      type: integer 
      description: CernVM-FS proxy port
      default: 80
      required: false
    refdata_dir:
      type: string
      description: path to store galaxy reference data
      default: /cvmfs
      required: false
    flavor:
      type: string
      description: name of the Galaxy flavor
      required: true
      default: galaxy-no-tools

If ``refdata_cvmfs_configuration`` is set to ``cvmfs`` all the parameters are required to setup the CVMFS repository.

On the contrary, if ``refdata_cvmfs_configuration`` is set to ``cvmfs_preconfigured`` only ``refdata_cvmfs_repository_name``, i.e. the name of the repository is needed, since all the needed parameters are retrieved from `GitHub <https://github.com/indigo-dc/Reference-data-galaxycloud-repository>`_.

----------------
``requirements``
----------------

Also in this case, Galaxy is required to install and configure reference data:

::

  requirements:
    - host:
        capability: tosca.capabilities.Container
        node: tosca.nodes.indigo.GalaxyPortal
        relationship: tosca.relationships.HostedOn

-------------
``artifacts``
-------------

The role is used to install cvmfs client.

::

  artifacts:
    cvmfs_role:
      file: indigo-dc.cvmfs-client
      type: tosca.artifacts.AnsibleGalaxy.role
    galaxy_role:
      file: indigo-dc.galaxycloud-refdata
      type: tosca.artifacts.AnsibleGalaxy.role

--------------
``interfaces``
--------------

The Ansible role is called with the paramteres:

::

  interfaces:
    Standard:
      configure:
        implementation: https://raw.githubusercontent.com/indigo-dc/tosca-types/v3.0.1/artifacts/galaxy/galaxy_redfata_configure.yml
        inputs:
          get_refdata: { get_property: [ SELF, reference_data ] }
          refdata_cvmfs_configuration: { get_property: [ SELF, refdata_cvmfs_configuration ] }
          refdata_cvmfs_repository_name: { get_property: [ SELF, refdata_cvmfs_repository_name ] }
          refdata_cvmfs_server_url: { get_property: [ SELF, refdata_cvmfs_server_url ] }
          refdata_cvmfs_key_file: { get_property: [ SELF, refdata_cvmfs_key_file ] }
          refdata_cvmfs_proxy_url: { get_property: [ SELF, refdata_cvmfs_proxy_url ] }
          refdata_cvmfs_proxy_port: { get_property: [ SELF, refdata_cvmfs_proxy_port ] }
          refdata_dir: { get_property: [ SELF, refdata_dir ] }
          galaxy_flavor: { get_property: [ SELF, flavor ] }


The role download from the `GitHub <https://github.com/indigo-dc/Reference-data-galaxycloud-repository>`_ repository all needed information to mount the CVMFS repository:

::

  ---
  - hosts: localhost
    connection: local
    pre_tasks:
      - set_fact:
          galaxy_flavor: 'galaxy-no-tools'
        when: galaxy_flavor == 'galaxy-minimal'
      - name: Get reference data cvmfs key for on-the-fly configuration
        get_url:
          url: 'https://raw.githubusercontent.com/indigo-dc/Reference-data-galaxycloud-repository/master/cvmfs_server_keys/{{ refdata_cvmfs_key_file }}'
          dest: '/tmp'
        when: refdata_cvmfs_configuration == 'cvmfs'
      - name: Get reference data cvmfs key for preconfigured repository
        get_url:
          url: 'https://raw.githubusercontent.com/indigo-dc/Reference-data-galaxycloud-repository/master/cvmfs_server_keys/{{ refdata_cvmfs_repository_name }}.pub'
          dest: '/tmp'
        when: refdata_cvmfs_configuration == 'cvmfs_preconfigured'
      - name: Get reference data cvmfs configuration for preconfigured repository
        get_url:
          url: 'https://raw.githubusercontent.com/indigo-dc/Reference-data-galaxycloud-repository/master/cvmfs_server_config_files/{{ refdata_cvmfs_repository_name }}.conf'
          dest: '/tmp'
        when: refdata_cvmfs_configuration == 'cvmfs_preconfigured'
    roles:
      - role: indigo-dc.galaxycloud-refdata

GalaxyPortalDocker
------------------

The role to deploy the Galaxy Official Docker is derived again from the GalaxyPortalAndStorage, allowing to configure the same options and to perform, also, the storage encryption.

::

  tosca.nodes.indigo.GalaxyPortalDocker:
    derived_from: tosca.nodes.indigo.GalaxyPortalAndStorage

--------------
``properties``
--------------

The reference data are automatically configured, using CVMFS. Therefore the repository name is needed between the inputs.

::

  properties:
    refdata_cvmfs_repository_name:
      type: string
      description: CernVM-FS repository name
      default: 'elixir-italy.galaxy.refdata'
      required: false

-------------
``artifacts``
-------------

The Docker engine has to be installed, alongside with the role to configure the Docker and the storage encryption.

::

  artifacts:
    nfs_role:
      file: indigo-dc.nfs
      type: tosca.artifacts.AnsibleGalaxy.role
    galaxy_os_role:
      file: indigo-dc.galaxycloud-os
      type: tosca.artifacts.AnsibleGalaxy.role
    docker_role:
      file: indigo-dc.docker
      type: tosca.artifacts.AnsibleGalaxy.role
    galaxy_role_docker:
      file: indigo-dc.galaxycloud_docker
      type: tosca.artifacts.AnsibleGalaxy.role

--------------
``interfaces``
--------------

The Ansible role is called with the paramteres:

::

  interfaces:
    Standard:
      configure:
        implementation: https://raw.githubusercontent.com/indigo-dc/tosca-types/v3.0.1/artifacts/galaxy/galaxy_docker.yml
        inputs:
          storage_encryption: { get_property: [ SELF, storage_encryption ] }
          vault_url: { get_property: [ SELF, vault_url ] }
          vault_wrapping_token: { get_property: [ SELF, vault_wrapping_token ] }
          vault_secret_path: { get_property: [ SELF, vault_secret_path ] }
          vault_secret_key: { get_property: [ SELF, vault_secret_key ] }
          galaxy_install_path: { get_property: [ SELF, install_path ] }
          galaxy_user: { get_property: [ SELF, user ] }
          galaxy_admin: { get_property: [ SELF, admin_email ] }
          galaxy_admin_api_key: { get_property: [ SELF, admin_api_key ] }
          galaxy_lrms: { get_property: [ SELF, lrms, type ] }
          galaxy_version: { get_property: [ SELF, version ] }
          galaxy_instance_description: { get_property: [ SELF, instance_description ] }
          galaxy_instance_key_pub:  { get_property: [ SELF, instance_key_pub ] }
          export_dir: { get_property: [ SELF, export_dir ] }
          galaxy_flavor: { get_property: [ SELF, flavor ] }
          get_refdata: { get_property: [ SELF, reference_data ] }
          refdata_cvmfs_repository_name: { get_property: [ SELF, refdata_cvmfs_repository_name ] }

Finally, the galaxycloud_docker ansible role download and run the Galaxy Docker image.

::

  ---
  - hosts: localhost
    connection: local
    roles:
      - role: indigo-dc.galaxycloud-os
        GALAXY_ADMIN_EMAIL: "{{ galaxy_admin }}"
        application_virtualization_type: 'docker'
        enable_reboot_scripts: false
        enable_customization_scripts: false
  
      - role: indigo-dc.galaxycloud_docker
        GALAXY_VERSION: "{{ galaxy_version }}"
        GALAXY_ADMIN_EMAIL: "{{ galaxy_admin }}"
        GALAXY_ADMIN_API_KEY: "{{ galaxy_admin_api_key }}"
