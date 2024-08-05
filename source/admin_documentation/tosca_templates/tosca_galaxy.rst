Galaxy template
===============

The orchetrator interprets the TOSCA template and orchestrate the Galaxy deployment on the virtual machine.

Galaxy template is located `here <https://raw.githubusercontent.com/Laniakea-elixir-it/laniakea-dashboard-config/master/tosca-templates/galaxy.yaml>`_.

Input parameters are needed for each custom type used in the template:

   - Virtual hardware parameters:

     ::

       number_cpus:
         type: integer
         description: number of cpus required for the instance
         default: 1
       memory_size:
         type: string
         description: ram memory required for the instance
         default: 1 GB
       storage_size:
         type: string
         description: storage memory required for the instance
         default: 10 GB

   - Galaxy input paramters:

     ::

       admin_email:
         type: string
         description: email of the admin user
         default: admin@admin.com
       admin_api_key:
         type: string
         description: key to access the API with admin role
         default: not_very_secret_api_key
       user:
         type: string
         description: username to launch the galaxy daemon
         default: galaxy
       version:
         type: string
         description: galaxy version to install
         default: master
       instance_description:
         type: string
         description: galaxy instance description
         default: "INDIGO Galaxy test"
       instance_key_pub:
         type: string
         description: galaxy instance ssh public key
         default: your_ssh_public_key
       export_dir:
         type: string
         description: path to store galaxy data
         default: /export

   - Storage input parameters:

     ::

       galaxy_storage_type:
         type: string
         description: Storage type (Iaas Block Storage, Onedaata, Filesystem encryption)
         default: "IaaS"
       userdata_provider:
         type: string
         description: default OneProvider
         default: "not_a_privder_url"
       userdata_token:
         type: string
         description: Access token for onedata space
         default: "not_a_token"
       userdata_space:
         type: string
         description: Onedata space
         default: "galaxy"

   - Galaxy flavor input parameters:

     :: 

       flavor:
         type: string
         description: Galaxy flavor for tools installation
         default: "galaxy-no-tools"

   - Reference data input parameters, for all possible options (CernVM-FS, Onedata and download).

     ::

       reference_data:
         type: boolean
         description: Install Reference data
         default: true
       refdata_dir:
         type: string
         description: path to store galaxy reference data
         default: /refdata
       refdata_repository_name:
         type: string
         description: Onedata space name, CernVM-FS repository name or subdirectory downaload name
         default: 'elixir-italy.galaxy.refdata'
       refdata_provider_type:
         type: string
         description: Select Reference data provider type (Onedata, CernVM-FS or download)
         default: 'onedata'
       refdata_provider:
         type: string
         description: Oneprovider for reference data
         default: 'not_a_provider'
       refdata_token:
         type: string
         description: Access token for reference data
         default: 'not_a_token'
       refdata_cvmfs_server_url:
         type: string
         description: CernVM-FS server, replica or stratum-zero
         default: 'server_url'
       refdata_cvmfs_repository_name:
         type: string
         description: Reference data CernVM-FS repository name
         default: 'not_a_cvmfs_repository_name'
       refdata_cvmfs_key_file:
         type: string
         description: CernVM-FS public key
         default: 'not_a_key'
       refdata_cvmfs_proxy_url:
         type: string
         description: CernVM-FS proxy url
         default: 'DIRECT'
       refdata_cvmfs_proxy_port:
         type: integer
         description: CernVM-FS proxy port
         default: 80

Input parameters are passed to the corresponding ansible roles, through custom type call:

::

    galaxy:
      type: tosca.nodes.indigo.GalaxyPortalAndStorage
      properties:
        os_storage: { get_input: galaxy_storage_type }
        token: { get_input: userdata_token }
        provider: { get_input: userdata_provider }
        space: { get_input: userdata_space }
        admin_email: { get_input: admin_email }
        admin_api_key: { get_input: admin_api_key }
        version: { get_input: version }
        instance_description: { get_input: instance_description }
        instance_key_pub: { get_input: instance_key_pub }
        export_dir: { get_input: export_dir }
      requirements:
        - lrms: local_lrms

    galaxy_tools:
      type: tosca.nodes.indigo.GalaxyShedTool
      properties:
        flavor: { get_input: flavor }
        admin_api_key: { get_input: admin_api_key }
      requirements:
        - host: galaxy

    galaxy_refdata:
      type: tosca.nodes.indigo.GalaxyReferenceData
      properties:
        reference_data: { get_input: reference_data }
        refdata_dir: { get_input: refdata_dir }
        flavor: { get_input: flavor }
        refdata_repository_name: { get_input: refdata_repository_name }
        refdata_provider_type: { get_input: refdata_provider_type }
        refdata_provider: { get_input: refdata_provider }
        refdata_token: { get_input: refdata_token }
        refdata_cvmfs_server_url: { get_input: refdata_cvmfs_server_url }
        refdata_cvmfs_repository_name: { get_input: refdata_cvmfs_repository_name }
        refdata_cvmfs_key_file: { get_input: refdata_cvmfs_key_file }
        refdata_cvmfs_proxy_url: { get_input: refdata_cvmfs_proxy_url }
        refdata_cvmfs_proxy_port: { get_input: refdata_cvmfs_proxy_port }
      requirements:
        - host: galaxy
        - dependency: galaxy_tools

.. Note::

   Note that Reference data custom type needs Galaxy installed to the ost ``host: galaxy``, but depends on galaxy tools ``dependency: galaxy_tools`` since it has to check installed and missing tools.

Finally we have virtual hardware customization:

::

        host:
         properties:
           num_cpus: { get_input: number_cpus }
           mem_size: { get_input: memory_size }

Image selection:

::

        os:
          properties:
            type: linux 
            distribution: centos
            version: 7.2
            image: indigodatacloudapps/galaxy

And Storage configuration, which takes the ``export_dir`` input for the mount point and ``storage_size`` input allowing for storage size customization.

::

        - local_storage:
            # capability is provided by Compute Node Type
            node: my_block_storage
            capability: tosca.capabilities.Attachment
            relationship:
              type: tosca.relationships.AttachesTo
              properties:
                location: { get_input: export_dir }
                device: hdb

::

    my_block_storage:
      type: tosca.nodes.BlockStorage
      properties:
      size: { get_input: storage_size }
