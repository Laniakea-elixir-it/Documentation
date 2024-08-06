Cloud Images Creation
=====================

Laniakea exploits Cloud images for Express builds, i.e. the deployment of application which are pre-installed on a cloid images, and just reconfigured and started at deployment time.

These images needs to be rebuilt every time a new software release or update is delivered, and, of course, in case of critical security issues. Therefore, in the long run, their manual creation was not a sustainable strategy.

We use `Hashicorp Packer <https://developer.hashicorp.com/packer>`_ to create images, whith OpenStack plugin, to instantiate the VM on openstack cloud and save it automatically on it, and Ansible for software installation and configuration. Packer will create a VM on OpenStack, will use ansible to configure the software on it (same role used by Laniakea Live builds), and, once finished, will save the image on Glance, ready to be used. Finally, the image can be shared with all Laniakea tenants.

The `Cloud image module <https://github.com/Laniakea-elixir-it/laniakea-ci-infrastructure/tree/master/cloud-images>`_ encompasses:

#. a `YAML file <https://raw.githubusercontent.com/Laniakea-elixir-it/laniakea-ci-infrastructure/master/cloud-images/images_list.yaml>`_ listing the image to create and Ansible inputs.

#. a `python <https://raw.githubusercontent.com/Laniakea-elixir-it/laniakea-ci-infrastructure/master/cloud-images/build_images.py>`_ script for creating the image with Packer, which build the Packer json file and run it;

#. the Ansible playbooks and corresponding variables for application installation, i.e. Galaxy, RStudio, Jupyther and more.

#. a `JenkinsFile <https://raw.githubusercontent.com/Laniakea-elixir-it/laniakea-ci-infrastructure/master/cloud-images/JenkinsFile>`_ for jenkins integration;

The image list
--------------

The list of the cloud images that need to be created is hosted on `GitHub <https://github.com/Laniakea-elixir-it/laniakea-ci-infrastructure/tree/master/cloud-images>`_ as a YAML file.

-----------------
``images_db_url``
-----------------

Every time a new image is created, it is stored on CMDB. To prevent the re-build of images alrady created we check, before image building if the image is already on CMDB. If yes it is skipped.

----------
``images``
----------

In the section ``images`` we describe the image to create. In the following we example of a galaxy cloud image is used.

::

      galaxy-express:
        name: "<IMAGE NAME>" <--------
        version: <vX.Y.Z> <--------
        build: no
        packer:
            ssh_username: rocky <--------
            source_image: "<IMAGE ID>" <--------
            flavor: large  <--------
            volume_size: "10"  <--------
            network_id: "<NETWORK ID>" <--------
            playbook_file: "galaxy.yml" <--------
            ansible_galaxy_file: "galaxy.yml" <--------

**name**: The name of the cloud image

**version**: The version of the cloud image in the form vX.Y.Z, e.g. v1.0.0

**build**: a boolean to enable image building. If yes the image is built, if no, it is not.

**ssh_username**: the user Ansible will use to access the created VM (default: rocky).

**source_image**: the soruce image ID on OpenStack to use.

**flavor**: the flavor of the VM to use on OpenStack (default:large).

**volume_size:** the size of the volume used for the VM (default: "10" GB).

**network_id**: the network ID to be attached to the VM

**playbook_file**: Ansible playbook which will be run by Ansible, e.g. galaxy.yml. Available playbooks are stored in the `playbook directory <https://github.com/Laniakea-elixir-it/laniakea-ci-infrastructure/tree/master/cloud-images/playbooks>`_.

**ansible_galaxy_file**: the requiremets file which will be used by **Ansible Galaxy** to install roles. All requirements file are in the `requiremets directory <https://github.com/Laniakea-elixir-it/laniakea-ci-infrastructure/tree/master/cloud-images/requirements>`_.

Image creation with Packer
--------------------------

Integration with Jankins
------------------------

Managing images
---------------


Finally, the created images are shared among all OpenStack tenant with stored on CMDB


Resources
---------

Cloud Veneto image management: https://userguide.cloudveneto.it/en/latest/ManagingImages.html

