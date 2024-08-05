|galaxy_docker| configuration
=============================

|project_name| leverages on an Ansible role (indigo-dc.galaxycloud_docker) to run a Galaxy Docker in a Virtual Machine.


Recommended images
------------------

Laniaeka exploits Ubuntu to run the Galaxy Docker. The recommended image is `Ubuntu 16.04 LTS cloud images <https://cloud-images.ubuntu.com/xenial/>`_

CMDB configuration
------------------

The image details must be uploaded on CMDB (see section :doc:`cmdb`). The json file needs the following details:

- ``image_id``: is the ID of the image on the cloud platform, e.g. Openstack.

- ``service``: is the service ID, configured on CMDB.

- ``image_name``: is the name of the image that has to be used in the tosca template in the ``image`` field.

SSH on CMDB virtual machine and create the file ``cmdb-data/ubuntu.json`` with the following content:

::

  {
    "type": "image",
    "data": {
        "image_id": "<ubuntu-image-id>",
        "image_name": "ubuntu-16.04-vmi",
        "architecture": "x86_64",
        "type": "linux",
        "distribution": "ubuntu",
        "version": "16.04",
        "service": "<service-id>"
    }
  }

where ``ubuntu-image-id`` is the image ID on the Cloud platform, while ``service-id`` is the service ID on CMDB.

To upload the image information on CMDB:

::

  curl -X POST http://cmdb:<cmdb_crud_password>@localhost:5984/indigo-cmdb-v2 -H "Content-Type: application/json" -d@cmdb-data/ubuntu.json

The image shuld now be available on CMDB with the name: ``ubuntu-16.04-vmi``.

.. note::

   All CMDB image are listed at the address: https://<proxy_vm_dns_name>/cmdb/image/list?include_docs=true

Tosca template configuration
----------------------------

The TOSCA template of the |galaxy_docker| is located at ``/opt/laniakea-dashboard-config/tosca-templates/galaxy-docker.yaml`` and is configured to exploit Ubuntu 16.04 as default image:

::

  galaxy_server:
  ...
      # Guest Operating System properties
      os:
        properties:
          image: ubuntu-16.04-vmi
  ...

If the images is uploaded on CMDB with a different name, the image field needs to be changed accordingly.
