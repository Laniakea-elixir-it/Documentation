Proxy server
============
A proxy server is used to expose IM, CMDB, CPR and the PaaS Orchestrator.

VM configuration
----------------

The control machine can be used to run the proxy server. The VM should meet the following minimum requirements:

======= ==============================
OS      Ubuntu 16.04
vCPUs   1
RAM     2 GB
Network Public and private IP address.
======= ==============================

.. warning::

   All the command will be run from the control machine.

Installation
------------

Create the file ``indigopaas-deploy/ansible/inventory/group_vars/proxy.yaml`` with the following configured values:

::

 letsencrypt_email: "<valid_email_address>"
 domain_name: "<proxy_vm_dns_name>"

.. warning::

   Please provide a valid e-mail address, which is mandatory for Let's Encrypt certificate creation.              

Run the role using ``ansible-playbook``

::

  # cd indigopaas-deploy/ansible

  # ansible-playbook -i inventory/inventory playbooks/deploy-proxy.yml

Video tutorial
--------------

.. raw:: html

   <a href="https://asciinema.org/a/269283" target="_blank"><img src="https://asciinema.org/a/269283.svg" /></a>
