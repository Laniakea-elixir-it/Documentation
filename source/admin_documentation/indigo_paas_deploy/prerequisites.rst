Prerequisites
=============

The installation procedure exploits Ansible to deploy all the INDIGO services.

A Virtual Machine with ansible is mandatory for this purpose, which we will refer to as **control machine** in the following. This VM will be used to run the installation procedure of each INDIGO component on the remote VMs. 

Here, we will exploit the same VM we will use to deploy the Proxy Server.

VM configuration
----------------

======= ==============================
OS      Ubuntu 16.04
vCPUs   2
RAM     4 GB
Network Public and private IP address.
======= ==============================

This VM will be used as ``control machine`` VM to run Ansible and will also serve as host for the proxy server.

.. warning::

   All the command will be run on this ``control machine`` VM!

Ansible installation
--------------------

Ansible can be easily installed following the `documentation <https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html>`_.

We tested the whole procedure using ``Ansible 2.8.3`` with `Ubuntu 16.04 <https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#latest-releases-via-apt-ubuntu>`_.

Configuration
-------------

1. Clone the `indigopaas-deploy repository <https://github.com/indigo-dc/indigopaas-deploy/tree/devel>`_, the collection of recipes to install INDIGO-DataCloud PaaS micro-services 

  ::

    # git clone -b v1.0 https://github.com/indigo-dc/indigopaas-deploy.git
 
    # cd indigopaas-deploy

    # mkdir ansible/inventory

2. Create the file ``indigopaas-deploy/ansible/inventory/inventory`` and set the IP of the virtual machines for each service as shown in the following:
 

   .. code:: bash
   
      [iam]
      <iam_vm_public_ip>
   
      [im]
      <im_vm_private_ip>
   
      [cmdb]
      <cmdb_vm_private_ip>
   
      [cpr]
      <cpr_vm_private_ip>
   
      [slam]
      <slam_vm_public_ip>
   
      [proxy]
      <proxy_vm_private_ip>
   
      [orchestrator]
      <orchestrator_vm_private_ip>

      [vault]
      <vault_vm_public_ip>

      [orchestrator-dashboard]
      <dashboard_vm_public_ip> 

   .. warning::

      CMDB and CPR will be host on the same Virtual Machine in this guide.

   .. warning::

      Vault and the Orchestrator Dashboard will be host on the same Virtual Machine in this guide.

3. Create the ``group_vars`` directory in ``indigopaas-deploy/ansible/inventory/``

   ::

     # cd indigopaas-deploy/ansible/inventory 

     # mkdir group_vars

   This directory will be populated with the YAML files with the configuration variables for each indigo component, with the following structure:

   |        group_vars/
   |         ├── cmdb.yml
   |         ├── iam.yaml
   |         ├── im.yaml
   |         ├── orchestrator.yaml
   |         ├── proxy.yaml
   |         └── slam.yaml


SSH key pair configuration
--------------------------

To run Ansible on remote hosts we need to configure an SSH connection on each VM. 

You can create a new SSH key

::

  # ssh-keygen -t rsa -b 4096

The default vaules should be ok.

Then you can distribute your new key copying and pasting the public key, i.e. the content of the file ``.ssh/id_rsa.pub``, to ``/root/.ssh/authorized_keys`` on each virtual machine allowing ansible to  to execute indigopaas-deploy roles.

.. warning:: The Ansible roles will install all the services over HTTPS protocol using Let's Eencrypt certificates.
