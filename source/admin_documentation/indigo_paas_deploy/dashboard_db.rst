Install Laniakea dashboard (database version)
==============================================

Create the file ``indigopaas-deploy/ansible/inventory/group_vars/orchestrator-dashboard.yaml`` with the following configured values:

::

  dashboard_fqdn: <dashboard_vm_dns_name>
  dashboard_image_name: laniakeacloud/laniakea-dashboard:withDB
  
  dashboard_iam_issuer: "https://<iam_address>/"
  dashboard_iam_client_id: "<im_client_id>'"
  dashboard_iam_client_secret: "<iam_client_secret>"
  dashboard_orchestrator_url: "https://<proxy_vm_dns_name>/orchestrator"
  dashboard_slam_url: "https://<slam_vm_dns_name>:8443"
  dashboard_cmdb_url: "https://<proxy_vm_dns_name>/cmdb"
  dashboard_im_url: "https://<proxy_vm_dns_name>/im"
  
  dashboard_tosca_template_repository_url: https://github.com/Laniakea-elixir-it/laniakea-dashboard-config.git
  dashboard_tosca_template_repository_dir: "/opt/laniakea-dashboard-config"
  dashboard_tosca_templates_dir: "/opt/laniakea-dashboard-config/tosca-templates"
  dashboard_tosca_parameters_dir: "/opt/laniakea-dashboard-config/tosca-parameters"
  dashboard_tosca_metadata_dir: "/opt/laniakea-dashboard-config/tosca-metadata"
  dashboard_administrators: "['<valid_email_address>']"
  
  dashboard_letsencrypt_email: "<valid_email_address>"

  dashboard_enable_db: True
  dashboard_db_sql_file_url: "https://raw.githubusercontent.com/Laniakea-elixir-it/orchestrator-dashboard/laniakea-stable/utils/orchestrator_dashboard.sql"
  dashboard_mysql_root_password: ******
  dashboard_db_password: ******

.. warning::

   Depending on your Cloud Provider network configuration, the database IP address needs to be further configured, for example using the private ip address associated to the VM, when a floating IP is used.

   In this case it is possible to set the database IP address adding:

   ::

     dashboard_db_host: '<vm_private_ip_address>'

.. warning::

   Set also your custom mysql password with: ``dashboard_mysql_root_password`` and ``dashboard_mysql_password``.

Run the role using the ``ansible-playbook`` command:

::

  # cd indigopaas-deploy/ansible 

  # ansible-playbook -i inventory/inventory playbooks/deploy-orchestrator-dashboard.yml
