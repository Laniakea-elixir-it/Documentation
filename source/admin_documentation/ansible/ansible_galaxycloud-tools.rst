Galaxycloud-tools
=================

This Ansible role is for automated installation of tools from a Tool Shed into Galaxy. The role use the path scheme from the role indigo-dc.galaxycloud

When run, this role will create a virtual environment, install ephemeris and invoke the install script to tools into Galaxy. The script stop Galaxy (if running), start a local Galaxy instance on http://localhost:8080 and install tools.

The list of tools to install is provided in files/tool_list.yaml file, hosted on an external repository: https://github.com/indigo-dc/Galaxy-flavors-recipes.

The role automatically clone this repository to install tools.

Requirements
------------
This ansible role supports CentOS 7, Ubuntu 14.04 Trusty and Ubuntu 16.04 Xenial

.. Note::
  Minimum ansible version: 2.1.2.0

Role Variables
--------------

Path
****
``galaxy_instance_description``: set Galaxy brand

``galaxy_user``: set linux user to launch the Galaxy portal (default: ``galaxy``).

``GALAXY_UID``: set user UID (default: ``4001``).

``galaxy_FS_path``: path to install Galaxy (default: ``/home/galaxy``).

``galaxy_directory``: Galaxy directory (usually galaxy or galaxy-dist, default ``galaxy``).

``galaxy_install_path``: Galaxy installation directory (default: ``/home/galaxy/galaxy``).

``galaxy_config_path``: Galaxy config pat location.

``galaxy_config_file``: Galaxy primary configuration file.

``galaxy_venv_path``:  Galaxy virtual environment directory (usually located to ``<galaxy_install_path>/.venv``).

``galaxy_custom_config_path``: Galaxy custom configuration files path (default: ``/etc/galaxy``).

``galaxy_custom_script_path``: Galaxy custom script path (defautl: ``/usr/local/bin``).

``galaxy_log_path``: log file directory (default: ``/var/log/galaxy``).

``galaxy_instance_url``: instance url (default:  ``http://<ipv4_address>/galaxy/``).

``galaxy_instance_key_pub``: instance ssh public key to configure <galaxy_user> access.

Main options
************
``GALAXY_ADMIN_API_KEY``: Galaxy administrator API_KEY. https://wiki.galaxyproject.org/Admin/API. Please note that this key acts as an alternate means to access your account, and should be treated with the same care as your login password. To be changed by the administrator.(default value: ``GALAXY_ADMIN_API_KEY``)

``galaxy_tools_tool_list_files``: a list of yml files that list the tools to be installed.

``galaxy_tools_base_dir``: base dir to install installation script (default: ``/tmp``).

``galaxy_flavor``: galaxy flavor to install. Each flavor corresponds to a directory hosted here: https://github.com/indigo-dc/Galaxy-flavors-recipes (defautl: ``galaxy-no-tools``).

``lock_file_path``: add lock file to avoid role re-run during recipe update. To re-run the role remove the lock file {{lock_file_path}}/indigo-dc.galaxycloud-tools.lock (default: ``/var/run``).

``install_workflows: install workflows (default: ``false``).

``install_data_libraries: install data libs (default: ``false``).

``install_interactive_tours: enable interactive tours installation (default: ``false``).

``export_dir``: Galaxy userdata are stored here (default: ``/export``).

This role exploits a lite version of galaxy to install data-libraries. It install dataset in /home/galaxy/galaxy/database/files/000 by defaults.
If the default galaxy database directory is different you have two options. By default the role read galaxy.ini and move datasets to file_path dir:
::

  move_datasets: true
  set_dataset_dest_dir: false

Otherwise you can set the destination directory. Dataset_dest_dir must exist, since the role will not create it.

::

  move_datasets: true
  set_dataset_dest_dir: true
  dataset_dest_dir: '/path/to/dir'

Defaults values:

``move_datasets``:``true``

``set_dataset_dest_dir``:``true``

``dataset_dest_dir``:``/path/to/dir``

``add_more_assets``: add custom resources (i.e. visualisations plugins, custom web pages, etc.). Since there is no a standard way to retrieve and install visualisation plugin, we keep this recipes external and implement a common interface to insall these resources (default: ``false``).

Create bootstrap user
*********************
if an apy key is not present on galaxy, a new user is created to istall tools and removed.
This is a very basic implementation. Advanced one is located here: https://raw.githubusercontent.com/indigo-dc/ansible-galaxy-tools/master/files/manage_bootstrap_user.py
Currently, to create it, few informations are needed:
- galaxy installation path
- galaxy_database_connection
- and pbkdf2 enabled

``create_bootstrap_user``: ``false``

``galaxy_database_connection``: ``postgresql://galaxy:galaxy@localhost:5432/galaxy``

``use_pbkdf2``: ``true``

``bootstrap_user_mail``: ``admin@server.com``

``bootstrap_user_name``: ``admin``

``bootstrap_user_password``: ``password``

By default, the api key is random-generated, overwriting the ``galaxy_admin_api_key`` variable assignment. You can set it to a defined value, by setting this ``create_random_api_key`` to false.
``create_random_api_key``: ``true``

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

::

    - hosts: servers
      roles:
        - role: indigo-dc.galaxycloud-tools
          galaxy_flavor: "galaxy-rna-workbench"
          galaxy_admin_api_key: "ADMIN_API_KEY"
          when: galaxy_flavor != "galaxy-no-tools"

Example Sources list
--------------------

The role takes a sources list as input. Sources list recipe is used to describe Galaxy resources, tools, reference data, workflow, data libraries and/or visualization plugin to install.

The role always install tools:

::

  - name: "Set {{galaxy_flavor}} resources"
    set_fact:
      galaxy_tools_tool_list_files:
        - '{{galaxy_tools_base_dir}}/Galaxy-flavors-recipes/{{galaxy_flavor}}/tool-list-example.yml'

You can enable workflows installation setting the variable ``install_workflows`` to ``true``, then insert the directory containing wokflows:

::

  # set path to workflow files
  # ephemeris takes the worlflow path to install workflows
  - name: "Set {{galaxy_flavor}} workflows resources"
    set_fact:
      install_workflows: true
      galaxy_tools_workflow_list_path:
        - '{{galaxy_tools_base_dir}}/Galaxy-flavors-recipes/{{galaxy_flavor}}/workflow'

The same goes for data libraries. You have to enable the installation, setting ``install_data_libraries``to ``true``, then the yaml recipe path

::

  # set yaml recipes
  # ephemeris takes single files as argument 
  - name: "Set {{galaxy_flavor}} data library resources"
    set_fact:
      install_data_libraries: true
      galaxy_tools_data_library_list_files:
        - '{{galaxy_tools_base_dir}}/Galaxy-flavors-recipes/{{galaxy_flavor}}/library_data.yaml'

To enable tours set ``install_interactive_tours`` to ``true`` and the tours path:

::

  # set galaxy tours path
  # the whole dir is copied to galaxy/config/plugins/tours/
  - name: "Set {{galaxy_flavor}} tours resources"
    set_fact:
      install_interactive_tours: true
      galaxy_tools_interactive_tour_list_path:
        - '{{galaxy_tools_base_dir}}/Galaxy-flavors-recipes/{{galaxy_flavor}}/tours'

Finally, it is possible to install external resources, like visualisation plugins, setting ``add_more_assets`` to ``true``:

::

  # set more resources to be installed
  # like visualisation plugins.
  # since there is no a standard way to retrieve and install
  #  visualisation plugin, we keep this recepie external.
  - name: "Install visualisation plugins"
    set_fact:
      add_more_assets: true
      galaxy_tools_assets_recipe_list_files:
        - '{{galaxy_tools_base_dir}}/Galaxy-flavors-recipes/{{galaxy_flavor}}/visualisations.yml'

Example Tool list
-----------------
For each tool you want to install, you must provide tool ``name`` and ``owner`` and one between ``tool_panel_section_id`` and ``tool_panel_section_label`` in the yaml tool list.

::

  ---
  api_key: <Admin user API key from galaxy_instance>
  galaxy_instance: <Galaxy instance IP>

  tools:
  - name: fastqc
    owner: devteam
    tool_panel_section_label: 'Tools'
    install_resolver_dependencies: True
    install_tool_dependencies: False

  - name: 'bowtie_wrappers'
    owner: 'devteam'
    tool_panel_section_label: 'Tools'
    install_resolver_dependencies: True
    install_tool_dependencies: False

License
-------

Apache Licence v2
