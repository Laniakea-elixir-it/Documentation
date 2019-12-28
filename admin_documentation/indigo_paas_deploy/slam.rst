SLA Manager (SLAM)
==================

The Service Level Agreement Manager (SLAM) is used to establish an agreement between customer and provider about capacity and quality targets. SLAM is using INDIGO IAM for authentication and INDIGO CMDB for configuration and authorization for providers.

.. note::
   Current SLAM version v2.0.0

VM configuration
----------------

Create VM for SLAM. The VM should meet the following minimum requirements:

======= ==============================
OS      Ubuntu 16.04
vCPUs   1
RAM     2 GB
Network Public IP address.
======= ==============================

.. warning::

   All the command will be run from the control machine VM.

SLAM IAM client creation
------------------------

Register a new IAM client for SLAM:

#. Login in IAM as admin.

#. Click on **MitreID Dashboard** and then **Self-service client registration**.

#. Click on **New client** and fill the form with the following parameters:

   ::

     Client name: slam_client

     redirect URI = https://<slam_vm_dns_name>:8443/auth

   .. figure:: _static/slam/slam_client_main.png
      :scale: 30%
      :align: center

#. In the Access tab select the following **Scopes** 

   ::

     Scopes: openid, profile, email, address, phone, offline_access

   and for **Grant Types** select:

   ::

     Grant types: authorization code

   .. figure:: _static/slam/slam_client_access.png
      :scale: 30%
      :align: center

#. Save.

#. Save **Client ID**, **Client Secret** and **Registration Access Token** or the full output json in the **JSON** tab for future access.

Installation
------------

Create the file ``indigopaas-deploy/ansible/inventory/group_vars/slam.yaml`` with the following configured values:

 ::
  
  letsencrypt_email: '<valid_email_address>'
  slam_dns_name: '<slam_dns_name>'
  slam_mysql_password: '*****'
  slam_mysql_root_password: '*****'
  slam_iam_url: 'https://<iam_dns_name>'
  slam_iam_client_id: '<slam-client-ID>'
  slam_iam_client_secret: '<slam-client-secret>'
  slam_cmdb_url: 'https://<proxy_dns_name>'
  slam_keystore_password: '*****'
  slam_create_keystore: true

.. warning::

   Set also your custom mysql password with ``slam_mysql_password``, ``lam_mysql_root_password`` and keystore password with ``slam_keystore_password``.

Run the role using the ``ansible-playbook`` command:

::

  # cd indigopaas-deploy/ansible 

  # ansible-playbook -i inventory/inventory playbooks/deploy-slam.yml

.. warning::

   SLAM will require few minutes to start and will be available at **https://<slam_dns_name>:8443/auth**.

Video tutorial
--------------

.. raw:: html

   <a href="https://asciinema.org/a/NmMSgc2QEB6I9Y32GASbXTBi7" target="_blank"><img src="https://asciinema.org/a/NmMSgc2QEB6I9Y32GASbXTBi7.svg" /></a>

SLAM configuration
------------------

Authorize SLAM
^^^^^^^^^^^^^^

#. SLAM is available at **https://<slam_dns_name>:8443/auth**. It will redirect you to IAM

#. Login as admin

#. Authorize SLAM

   .. figure:: _static/slam/slam_client_authorize.png
      :scale: 30%
      :align: center
   
   .. centered:: Fig.1: SLAM authorization

   .. figure:: _static/slam/slam_home.png
      :scale: 30%
      :align: center
   
   .. centered:: Fig.1: SLAM home page

Resources negotiation
^^^^^^^^^^^^^^^^^^^^^

To create new SLAs with SLAM follow this steps:

.. toctree::
   :maxdepth: 2

   slam_negotiation
