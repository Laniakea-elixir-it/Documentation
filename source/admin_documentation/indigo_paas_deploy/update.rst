Updating Laniakea
=================

The same ansible roles used to deploy Laniakea can be used to keep it up to date.

#. Update the indigpaas-deploy ansible roles:

::

  cd indigopaas-deploy

  git pull

#. All the services run inside docker container. Therefore, in most of cases, service aupdate requires to re-create the Docker container with the updated image. The corrisponding data are mounted inside the Docker container, thus avoiding any data loss during the update procedure.

   The services docker images can be changed in the corresponding configuration file in ``indigopaas-deploy/ansible/inventory/group_vars/<service>.yaml``.

#. Finally to update a service, just re-run the ansible role:

   ::
   
     # cd indigopaas-deploy/ansible 
   
     # ansible-playbook -i inventory/inventory playbooks/deploy-<service>.yml

.. warning::

   INDIGO Software catalogue is acively developed. So the update procedure of Laniakea depends on the INDIGO services evolution. We will keep this page updated accordingly.

.. note::

   All the (Galaxy) instances deployed with Laniakea are not influenced by the update procedure.

Current recommended configuration
---------------------------------

Currently, the following verions of the INDIGO services are recommended:

======================= =================== ======================================================
Service                 Version             Docker image
======================= =================== ======================================================
indigopaas-deploy       v1.0                ---
IAM                     1.5 rc2             indigoiam/iam-login-service:v1.5.0.rc2-SNAPSHOT-latest
IM                      1.8.8.1             indigodatacloud/im:1.8.6.1
CMDB                    indigo_2            indigodatacloud/cmdb:indigo_2
CPR                     indigo_2            indigodatacloud/cloudproviderranker:indigo_2
SLAM                    v2.0.0              indigodatacloud/slam:v2.0.0
Custom types            v3.0.1              ---
Orchestrator            2.1.2-final         indigodatacloud/orchestrator:2.1.2-final
Vault                   1.1.2               vault:1.1.2
Dashboard               laniakea-stable     laniakeacloud/laniakea-dashboard:stable
======================= =================== ======================================================
