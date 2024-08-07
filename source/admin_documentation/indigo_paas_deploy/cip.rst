Cloud Info Provider
===================

CMDB requires manual upload of tenant images and flavours. This operation can turn out to be a problem in the management of a large number of cloud images and flavors in the long run.

Therefore, it has been automated using the Cloud Info Provider (CIP), which periodically import all images and flavours available on Laniakea tenants.

CIP installation
----------------

#. Clone the CIP repository.

   ::

     git clone https://recas-paas:**********@baltig.infn.it/recas-bari/paas/cip.git /opt/cip


#. create a YAML file with the cloud details, we already used to configured CMDB, and name it as ``os.YOUR-CLOUD.yaml``

   ::
   
     site:
         name: <YOUR-CLOUD-NAME> <------
         id: <YOUR-CLOUD-NAME> <------
         is_public: false
         country: <your country> <------
         country_code: <your country code> <------
         latitude: <latitude> <------
         longitude: <longitude> <------
         roc: <roc>
         subgrid: <subgrid> <------
         giis_url: "<giis url>" <------
         owner_contacts:
             - <support@server.com> <------
         owner_contacts_iam:
     
     compute:
         total_cores: 0
         total_ram: 0
         max_dedicated_ram: 0
         min_dedicated_ram: 0
         accelerators_virt_type: UNKNOWN
         total_accelerators: 0
         max_accelerators: 0
         min_accelerators: 0
         hypervisor: UNKNOWN
         hypervisor_version: UNKNOWN
         service_production_level: production
         capabilities:
             - executionmanagement.dynamicvmdeploy
             - security.accounting
         failover: false
         live_migration: false
         vm_backup_restore: false
         endpoints:
             defaults:
                 iam_enabled: true
                 idp_protocol: openid
                 middleware_version: <Cloud version, e.g. yoga> <------
                 api_endpoint_technology: webservice
                 api_authn_method: openid
                 production_level: production
                 federation:
                   recas-bari:
                     issuer: <IAM ISSUER> <------
                     protocol: openid
         shares:
             <IAM ORGANISATION>: <------
                 auth:
                     project_id: <OPENSTACK TENANT NAME> <------
                 iam_organisation: <CORRESPONDING IAM ORGANISATION> <------
                 private_network_proxy_host: <PRIVATE IP BASTION> <------ (optional)
                 private_network_proxy_user: im <------ (optional)
             ... add more here ...  <------
         templates:
             defaults:
                 platform: amd64
                 network: public
                 network_in: undefined
                 network_out: true
         images:
             defaults:
                 os_type: Linux
                 architecture: x86_64
                 gpu_driver: 'NA'
                 gpu_cuda_driver: 'NA'
                 gpu_cudnn_driver: 'NA'


#. Copy the docker file from utils on cip directory

   cp utils/Dockerfile .

   and build it

   ::
   
     # docker build -t cip:latest .

#. We need operation account on IAM, belonging to the right group and tenant, all those configured in the ``shares`` section of the YAML file.
   
   Create it on iam and assign it to the right groups.

#. OIDC Agent is necessary to use the IAM user created in the previous section. It can be easily configured following `this <https://github.com/indigo-dc/oidc-agent>`_ procedure.

#. Then with this user create a new IAM client, **without** the password.

#. Edit ``utils/run_cip.sh`` adding CMDB username, password, URL and the IAM client name.

   ::
   
     #!/bin/bash
     set +x
     eval `oidc-keychain`
     
     docker run --rm --name cip -v /opt/cip:/opt/cip/sites cip:latest /opt/cip/sites/utils/collect-and-push.sh <CMDB USER> <CMDB PASSWORD> \
             http://<CMDB IP ADDRESS>:5984/indigo-cmdb-v2/_design/schema/_rewrite/ http://<CMDB IP ADDRESS>:5984/indigo-cmdb-v2 $(oidc-token <IAM CLIENT NAME>) 2>&1 | tee /opt/cip/log/cip.log

#. Edit ``utils/collect-and-push.sh`` adding OpenStack details.

#. CIP is run hourly, using cron. Add the file ``/etc/cron.d/run_cip``:

   :: 
   
     0 */6 * * * root /opt/cip/utils/run_cip.sh

#. To force CIP to update CMDB run:

   ::
   
     /opt/cip/utils/run_cip.sh
