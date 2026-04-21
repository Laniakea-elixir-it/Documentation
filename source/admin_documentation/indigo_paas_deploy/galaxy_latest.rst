|galaxy_latest| configuration
=============================

Recommended images
------------------

The OS image has to be available in the Tenant. Laniakea has been tested using `CentOS 7 <https://cloud.centos.org/centos/7/images/CentOS-7-x86_64-GenericCloud-1907.qcow2>`_.

CMDB configuration
------------------

The image details must be uploaded on CMDB (see section :doc:`cmdb`). The json file needs the following details:

- ``image_id``: is the ID of the image on the cloud platform, e.g. Openstack.

- ``service``: is the service ID, configured on CMDB.

- ``image_name``: is the name of the image that has to be used in the tosca template in the ``image`` field.

SSH on CMDB virtual machine and create the file ``cmdb-data/centos.json`` with the following content:

::

  {
    "type": "image",
    "data": {
        "image_id": "<centos-image-id>",
        "image_name": "centos-7-vmi",
        "architecture": "x86_64",
        "type": "linux",
        "distribution": "centos",
        "version": "7",
        "service": "<service-id>"
    }
  }

where ``centos-image-id`` is the image ID on the Cloud platform, while ``service-id`` is the service ID on CMDB.

To upload the image information on CMDB:

::

  curl -X POST http://cmdb:<cmdb_crud_password>@localhost:5984/indigo-cmdb-v2 -H "Content-Type: application/json" -d@cmdb-data/centos.json

The image shuld now be available on CMDB with the name: ``centos-7-vmi``.

.. note::

   All CMDB image are listed at the address: https://<proxy_vm_dns_name>/cmdb/image/list?include_docs=true

Tosca template configuration
----------------------------

*********
``Galaxy``
*********

The TOSCA template of the |galaxy_latest| is located at ``/opt/laniakea-dashboard-config/tosca-templates/galaxy.yaml`` and is configured to exploit CentOS 7 as default image:

::

  galaxy_server:
  ...
      # Guest Operating System properties
      os:
        properties:
          image: centos-7-vmi
  ...

If the images is uploaded on CMDB with a different name, the image field needs to be changed accordingly.

**************************
``Galaxy (elastic) cluser``
**************************

The |galaxy_cluster| TOSCA template is ``/opt/laniakea-dashboard-config/tosca-templates/galaxy-cluster.yaml``. The |galaxy_elastic_cluster| TOSCA template is ``/opt/laniakea-dashboard-config/tosca-templates/galaxy-elastic-cluster.yaml``.

The image configuration is the same. There are two image fields, one for the fronte node:

::

    lrms_server:
    ...
        # Front-End Guest Operating System properties
        os:
          properties:
            image: centos-7-vmi
    ...

and one for the worker node:

::

    lrms_wn:
    ...
        # Worker Node Guest Operating System properties
        os:
          properties:
            image: galaxy-base
            #image: centos-7-vmi

If the images is uploaded on CMDB with a different name, the image field needs to be changed accordingly.
