SLA Manager (SLAM)
==================

.. warning::

   The Service Level Agreement Tools (SLAT) replaced SLAM starting from Laniakea v3.0.0.

The Service Level Agreement Tool (SLAT) is used to establish an agreement between customer and provider about capacity and quality targets. SLAT is using INDIGO IAM for authentication and INDIGO CMDB for configuration and authorization for providers.

.. note::
   Current SLAT version v0.1

VM configuration
----------------

Create VM for SLAM. The VM should meet the following minimum requirements:

======= ==============================
OS      Ubuntu 22.04
vCPUs   1
RAM     2 GB
Network Public IP address.
======= ==============================

.. warning::

   All the command will be run from the control machine VM.

SLAT IAM client creation
------------------------

Register a new IAM client for SLAT:

#. Login in IAM.

#. Click on **MitreID Dashboard** and then **Self-service client registration**.

#. Click on **New client** and fill the form with the following parameters:

   ::

     Client name: slat.client.name

     redirect URI = https://<slat_vm_dns_name>/login/iam/authorized

   .. figure:: _static/slam/slam_client_main.png
      :scale: 30%
      :align: center

#. In the Access tab select the following **Scopes** 

   ::

     Scopes: openid, profile, email, address, offline_access

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

Create the file ``indigopaas-deploy/ansible/inventory/group_vars/slat.yaml`` with the following configured values:

 ::
  
  slat_image_name: "marica/slat:latest"
  
  slat_iam_issuer: 'https://<iam_dns_name'
  slat_iam_client_id: '<slam-client-ID>'
  slat_iam_client_secret: '<slam-client-secret>'
  
  slat_trusted_oidc_idp_list: [{ 'iss': 'https://<iam_dns_name>', 'type': 'indigoiam' }]
  
  slat_admin_group: 'laniakea-paas-admins'
  
  slat_log_level: 'info'
  
  slat_gunicorn_workers: "2"
  
  slat_mysql_image: mysql:5.7
  slat_db_data_dir: /data/slat/mysql
  slat_cmdb_url: https://<proxy_dns_nanme>/cmdb
  
  slat_db_host: <slat_db_public_ip>
  slat_mysql_root_password: ******
  slat_db_name: slat
  slat_db_user: slat
  slat_db_password: *****
  slat_db_port: 3306
  
  slat_enable_https: True
  slat_ssl_cert_generation: letsencrypt
  slat_letsencrypt_email: '<valid_email_address>'
  slat_fqdn: "<slat_dns_name>"
  slat_ssl_cert_path: "/etc/letsencrypt/live/<slat_dns_name>/fullchain.pem"
  slat_ssl_key_path: "/etc/letsencrypt/live/<slat_dns_name>/privkey.pem"


.. warning::

   Set also your custom mysql password with ``slat_db_password`` and ``slat_mysql_root_password``.

Run the role using the ``ansible-playbook`` command:

::

  # cd indigopaas-deploy/ansible 

  # ansible-playbook -i inventory/inventory playbooks/deploy-slat.yml

.. warning::

   SLAT will require few minutes to start and will be available at **https://<slat_dns_name>:5001

Video tutorial
--------------

.. raw:: html

   <a href="https://asciinema.org/a/NmMSgc2QEB6I9Y32GASbXTBi7" target="_blank"><img src="https://asciinema.org/a/NmMSgc2QEB6I9Y32GASbXTBi7.svg" /></a>

SLAT configuration
------------------

Authorize SLAT
^^^^^^^^^^^^^^

#. SLAT is available at **https://<slam_dns_name>:5001. It will redirect you to IAM

#. Login as admin

#. Authorize SLAT

   .. figure:: _static/slam/slam_client_authorize.png
      :scale: 30%
      :align: center
   
   .. centered:: Fig.1: SLAM authorization

   .. figure:: _static/slat/slat_home.png
      :scale: 30%
      :align: center
   
   .. centered:: Fig.1: SLAT home page

Resources negotiation
^^^^^^^^^^^^^^^^^^^^^

SLAT will retrieve the available services from CMDB automatically.

.. toctree::
   :maxdepth: 2

   slam_negotiation
