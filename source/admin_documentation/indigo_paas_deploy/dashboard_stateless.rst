Install Laniakea dashboard (stateless version)
==============================================

Create the file ``indigopaas-deploy/ansible/inventory/group_vars/orchestrator-dashboard.yaml`` with the following configured values:

::

  dashboard_fqdn: <dashboard_vm_dns_name>
  dashboard_image_name: indigodatacloud/orchestrator-dashboard
  
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

Run the role using the ``ansible-playbook`` command:

::

  # cd indigopaas-deploy/ansible 

  # ansible-playbook -i inventory/inventory playbooks/deploy-orchestrator-dashboard.yml
