TOSCA templates
===============

.. figure:: img/tosca_logo.png
   :scale: 100 %
   :align: center
   :alt: TOSCA logo

The `INDIGO PaaS Orchestrator <https://www.indigo-datacloud.eu/paas-orchestrator>`_ is the key software component of the INDIGO PaaS layer: it receives deployment requests from the user interface software layer and coordinates the deployment process over the IaaS platforms. The Orchestrator accepts the deployment requests written using the `TOSCA standard <http://docs.oasis-open.org/tosca/TOSCA-Simple-Profile-YAML/v1.0/TOSCA-Simple-Profile-YAML-v1.0.html>`_, allowing to deploy complex application using small building blocks, named node types, which exploit Ansible to install and configure the end-user applications or services, like Galaxy, on bare OS images. Therefore, to correctly orchestrate Galaxy deployment the following component are needed:

- `Ansible <https://www.ansible.com/>`_ roles to automate software installation and configuration (see section :doc:`/admin_documentation/ansible/ansible_roles`) 
- Custom types: define user configurable parameters, node requirements, call ansible playbooks.
- Artifact: define what to install and how to do it, through ansible role configuration.
- TOSCA template: the orchestrator interprets the TOSCA template and orchestrates the deployment.

.. Note::

   This section is not inteded to be a complete guide to TOSCA types, but aims to describes the solutions adopted to deploy Galaxy in |project_name|.

.. toctree::
   :maxdepth: 2
   :caption: TOSCA documentation

   tosca_customtypes
   tosca_galaxy
   tosca_galaxy_cluster
