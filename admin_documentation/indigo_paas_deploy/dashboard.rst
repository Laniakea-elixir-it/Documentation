Laniakea Dashboard
==================

The Laniakea Dashbaord is built on top of the INDIGO Orchestrator Dashboard.

.. note::

   Current Dahsboard version: stable version

VM configuration
----------------

Create VM for Dashboard. The VM should meet the following minimum requirements:

======= ==============================
OS      Ubuntu 16.04
vCPUs   2
RAM     4 GB
Network Public IP address.
======= ==============================

.. warning::

   In this tutorial we will use the same VM for vault and the dashbord, being the two services strictly connected.

   This is not requred.

.. warning::

   All the command will be run from the control machine VM.

IAM client configuration
-------------------------------

#. Login on IAM as Administrator User.

#. Navigate to **MitreID Dashboard** and select from the left panel **Self-service client registration**.

#. Create a **New client** and fill the form with the following paramethers

   ::

     Client name = dashboard_client

     redirect URI(s) = https://<dashboard_vm_dns_name>/login/iam/authorized

   .. figure:: _static/dashboard/dashboard_client_main.png
      :scale: 30%
      :align: center

#. In the Access tab select the follwing **Scopes**

   ::

     Scopes: openid, profile, email, address, phone, offline_access

   and for **Grant Types** select:

   ::

     Grant types: authorization code

   .. figure:: _static/dashboard/dashboard_client_access.png
      :scale: 30%
      :align: center

#. Save.

#. Save **Client ID**, **Client Secret** and **Registration Access Token** or the full output json in the **JSON** tab for future access.

Installation
------------

The Laniakea dashboard can be installed in three different ways: Stateless, with MySQL database and with MySQL and Vault integration.

The one with MySQL and Hashicorp Vault is the one used in Laniakea.

.. toctree::
   :maxdepth: 2

   dashboard_vault

Post installation steps to enable the callback
----------------------------------------------

If the callback is enabled, the Paas Orchestrator (:doc:`/admin_documentation/indigo_paas_deploy/orchestrator`) needs to be configured accordingly.

In particular, the dashboard CA certificate has to be copied on the PaaS Orchestrator Virtual Machine in ``/etc/orchestrator/trusted_certs``.

For Let's Encrypt certificats, those used in this wiki:

#. Connect through SSH to the Dashboard VM and copy the content of the file ``/etc/letsencrypt/live/<orchestrator_dashboard_dns_name>/chain.pem``.

#. Connect through SSH to the PaaS Orchestrator VM and paste the ``chain.pem`` to ``/etc/orchestrator/trusted_certs/dashboard-cert.pem``

#. Restart the PaaS Orchestrator with:

   ::

     # docker restart orchestrator

#. Once the Orchestrator is started the chain file can be removed:

   ::

     # rm /etc/orchestrator/trusted_certs/dashboard-cert.pem

Appendix A. Stateless version
-----------------------------

This is a simple graphical User interface of the INDIGO PaaS orchestrator. The automated storage encryption will not work.

.. toctree::
   :maxdepth: 2

   dashboard_stateless

.. warnings::

   The automated storage encryption will not work with this version.

Appendix B. Database version
----------------------------

This version comes with a MySQL database support.

.. toctree::
   :maxdepth: 2

   dashboard_db

.. warnings::

   The automated storage encryption will not work with this version.
