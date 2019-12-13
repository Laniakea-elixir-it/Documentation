PaaS Orchestrator
==================

PaaS Orchestrator is  the  core  component  of  the  PaaS  layer.  It  collects  high-level deployment requests from the software layer, and coordinates the resource or service deployment.

.. note::
   Current Orchestrator version: 2.1.2-final

VM configuration
----------------

Create VM for IM. The VM should meet the following minimum requirements:

======= ==============================
OS      Ubuntu 16.04
vCPUs   2
RAM     4 GB
Network Private IP address.
======= ==============================

IAM protected resource configuration for the Orchestrator
---------------------------------------------------------

#. Login on IAM then **MitreID Dashboard** and select **Self-service protected resource registration** as Administrator user.

#. Select **New Resource** with the follwoing parameters

   ::

     Name: orchestrator_client

     Scopes: openid, profile, offline_access

   .. figure:: _static/orchestrator/orchestrator_client_main.png
      :scale: 50 %
      :align: center

   .. figure:: _static/orchestrator/orchestrator_client_access.png
      :scale: 50 %
      :align: center

#. Save the protected resource.

#. Save **Client ID**, **Client Secret** and **Registration Access Token** or the full output json in the **JSON** tab for future access.

#. Edit the protected resource configuration page as Administrator user, through the **ADMINISTRATIVE**, **Manage Clients**

   .. figure:: _static/orchestrator/iam_manage_clients.png
      :scale: 50 %
      :align: center

#. Enable **Token exchange** and Check the flag at **Introspection**:

   ::

     Introspection Allow calls to the Introspection Endpoint?

   .. figure:: _static/orchestrator/orchestrator_admin_client_access.png
      :scale: 50 %
      :align: center

#. Navigate to the **Tokens** tab and set:

   - Access Token Timeout: 7200
   - ID Token Timeout: 7200

   and flag:

   -  Refresh tokens are issued for this client
   - Refresh tokens for this client are re-used
   - Active access tokens are automatically revoked when the refresh token is used
   - Refresh tokens do not time out 

   .. figure:: _static/orchestrator/orchestrator_admin_client_tokens.png
      :scale: 40%
      :align: center 

#. Save again the protected resource.

IAM  protected resource configuration for CLUES
------------------------------------------------

#. Login on IAM then **MitreID Dashboard** and select **Self-service protected resource registration** as Administrator user.

#. Select **New Resource** and set the follwoing parameters

   ::

     Name: clues_client

     Scopes: openid, profile, email, address,  phone, offline_access

   .. figure:: _static/orchestrator/clues_client_main.png
      :scale: 50 %
      :align: center

   .. figure:: _static/orchestrator/clues_client_access.png
      :scale: 50 %
      :align: center

#. Save the protected resource.

#. Save **Client ID**, **Client Secret** and **Registration Access Token** or the full output json in the **JSON** tab for future access.

#. Edit the protected resource configuration page as Administrator user, through the **ADMINISTRATIVE**, **Manage Clients**

#. Enable **Token exchange** and Check the flag at **Introspection**:

   .. figure:: _static/orchestrator/clues_admin_client_access.png
      :scale: 50 %
      :align: center

#. Navigate to the **Tokens** tab and set:

   - Access Token Timeout: 7200
   - ID Token Timeout: 7200

   and flag:
   
   -  Refresh tokens are issued for this client
   - Refresh tokens for this client are re-used
   - Active access tokens are automatically revoked when the refresh token is used
   - Refresh tokens do not time out

   .. figure:: _static/orchestrator/clues_admin_client_tokens.png
      :scale: 40%
      :align: center 

#. Save the protected resource again.

Orchestrator Installation
-------------------------

Create the file ``indigopaas-deploy/ansible/inventory/group_vars/orchestrator.yaml`` with the following configured values:

::

  orchestrator_url: https://<proxy_dns_name>/orchestrator
  orchestrator_image: indigodatacloud/orchestrator:2.1.2-final
  orchestrator_mysql_root_password: *****
  orchestrator_mysql_password: *****
  orchestrator_im_url: https://<proxy_dns_name>/im
  orchestrator_cmdb_url: https://<proxy_dns_name>/cmdb
  orchestrator_slam_url: https://<slam_dns_name>:8443/rest/slam
  orchestrator_cpr_url: https://<proxy_dns_name>/cpr
  orchestrator_iam_issuer: https://<iam_dns_name>/
  orchestrator_iam_client_id: <orchestrator_client_id>
  orchestrator_iam_client_secret: <orchestrator_client_secret>
  orchestrator_clues_iam_client_id: <clues_client_id> 
  orchestrator_clues_iam_client_secret: <clues_client_secrett> 
  orchestrator_custom_types: https://raw.githubusercontent.com/Laniakea-elixir-it/indigopaas-resources/master/orchestrator/custom_types.yaml
  disable_monitoring: True

.. Warning::

   SLAM and IAM are the only two services requiring a public IP, on the contrary all the others are behind the proxy.

.. Warning::

   In this guide we avoid monitoring installation, leaving this job to the Cloud provider.

Run the role using the ``ansible-playbook`` command:

::

  # cd indigopaas-deploy/ansible 

  # ansible-playbook -i inventory/inventory playbooks/deploy-orchestrator.yml

Video tutorial
--------------

.. raw:: html

   <a href="https://asciinema.org/a/eYWgO4OkemkpW5bpEUCx7K4UX" target="_blank"><img src="https://asciinema.org/a/eYWgO4OkemkpW5bpEUCx7K4UX.svg" /></a>

FAQ
---

.. toctree::
   :maxdepth: 2

   orchestrator_faq
   clues_faq
