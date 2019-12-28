Galaxy cluster template
=======================

The :doc:`ansible_galaxycloud` role provides the possibility to instantiate Galaxy with SLURM as Resource Manager, just setting the ``galaxy_lrms`` variable to ``slurm``.

This allows to instantiate Galaxy with SLURM cluster exploiting INDIGO custom types and ansible roles using INDIGO components:

- CLUES (INDIGO solution for automatic elasticity)
- Master node deployment with SLURM (ansible recipes + tosca types)
- Install Galaxy + SLURM support (already in our ansible role indigo-dc.galaxycloud)
- Worker node deployment
- Galaxy customization for worker nodes

The related tosca template is located `here <https://github.com/indigo-dc/tosca-types/blob/master/examples/galaxy_elastic_cluster.yaml>`_.

The input parameters allow to customize the number of virtual nodes, nodes and master virtual hardware:

::

    wn_num:
      type: integer
      description: Maximum number of WNs in the elastic cluster
      default: 5
      required: yes
    fe_cpus:
      type: integer
      description: Numer of CPUs for the front-end node
      default: 1
      required: yes
    fe_mem:
      type: scalar-unit.size
      description: Amount of Memory for the front-end node
      default: 1 GB
      required: yes
    wn_cpus:
      type: integer
      description: Numer of CPUs for the WNs
      default: 1
      required: yes
    wn_mem:
      type: scalar-unit.size
      description: Amount of Memory for the WNs
      default: 1 GB
      required: yes

.. Note::

   You can refere to :doc:`tosca_galaxy` section for galaxy input parameters.

The master node hosts Galaxy and Slurm controller:

::

    elastic_cluster_front_end:
      type: tosca.nodes.indigo.ElasticCluster
      properties:
        deployment_id: orchestrator_deployment_id
        iam_access_token: iam_access_token
        iam_clues_client_id: iam_clues_client_id
        iam_clues_client_secret: iam_clues_client_secret
      requirements:
        - lrms: lrms_front_end
        - wn: wn_node

    galaxy_portal:
      type: tosca.nodes.indigo.GalaxyPortal
      properties:
        admin_email: { get_input: admin_email }
        admin_api_key: { get_input: admin_api_key }
        version: { get_input: version }
        instance_description: { get_input: instance_description }
        instance_key_pub: { get_input: instance_key_pub }
      requirements:
        - lrms: lrms_front_end

    lrms_front_end:
      type: tosca.nodes.indigo.LRMS.FrontEnd.Slurm
      properties:
        wn_ips: { get_attribute: [ lrms_wn, private_address ] }
      requirements:
        - host: lrms_server

    lrms_server:
      type: tosca.nodes.indigo.Compute
      capabilities:
        endpoint:
          properties:
            dns_name: slurmserver
            network_name: PUBLIC
            ports:
              http_port:
                protocol: tcp
                source: 80
        host:
          properties:
            num_cpus: { get_input: fe_cpus }
            mem_size: { get_input: fe_mem }
        os:
          properties:
          image: linux-ubuntu-14.04-vmi

Then the worker nodes configuration (OS and virtual hardware):

::

    wn_node:
      type: tosca.nodes.indigo.LRMS.WorkerNode.Slurm
      properties:
        front_end_ip: { get_attribute: [ lrms_server, private_address, 0 ] }
      capabilities:
        wn:
          properties:
            max_instances: { get_input: wn_num }
            min_instances: 0
      requirements:
        - host: lrms_wn

    galaxy_wn:
      type: tosca.nodes.indigo.GalaxyWN
      requirements:
        - host: lrms_wn

    lrms_wn:
      type: tosca.nodes.indigo.Compute
      capabilities:
        scalable:
          properties:
            count: 0
        host:
          properties:
            num_cpus: { get_input: wn_cpus }
            mem_size: { get_input: wn_mem }
        os:
          properties:
          image: linux-ubuntu-14.04-vmi

.. Note::

   Note that to orchestrate Galaxy with SLURM we do not need new TOSCA custom types or ansible roles. Everythings is already built in INDIGO.
