Infrastructure Manager (IM)
===========================

The `Infrastructure Manager (IM) <https://www.grycap.upv.es>`_ is used to deploy virtual infrastructures, to deploy virtual infrastructures, e.g. Galaxy and the underlying virtual hardware.

.. note::
   Current IM version: 1.8.6.1

VM configuration
----------------

Create VM for IM. The VM should meet the following minimum requirements:

======= ==============================
OS      Ubuntu 16.04
vCPUs   2
RAM     4 GB
Network Private IP address.
======= ==============================

.. warning::

   All the command will be run from the control machine VM.

IAM protected resource configuration
------------------------------------

Register a new protected resource for IM on IAM:

#. Login on IAM as Administrator User.

#. Navigate to **MitreID Dashboard** and select from the left panel **Self-service protected resource registration**.

#. Create a **New Resource**.

#. Give it a name, e.g. ``im_test``.

#. Keep the default configuration and Save.

#. Save **Client ID**, **Client Secret** and **Registration Access Token** or the full output json in the **JSON** tab for future access.

#. As Administrator user select from the left menu **Manage Clients**.

#. Select the client just created.

#. Navigate to the **Tokens** tab and set it as in the figure and save. In particular set:

   - Access Token Timeout: 3600
   - ID Token Timeout: 1800

   .. figure:: _static/im/im_client_admin_tokens.png
      :scale: 40%
      :align: center

Installation
------------

Create the file ``indigopaas-deploy/ansible/inventory/group_vars/im.yaml`` with the following configured values:

::

 im_image_version: 1.8.6.1
 im_repo_tag: v1.8.6
 im_mysql_root_password: ********
 im_mysql_password: *********
 im_cfg_oidc_issuers: 'https://<iam_address>/'
 im_cfg_oidc_client_id: '<im_client_id>'
 im_cfg_oidc_client_secret: '<im_client_secret>'
 im_cfg_ssh_reverse_tunnels: 'True'
 im_ansible_version: '2.2.3.0'

.. warning::

   Set also your custom mysql password with: ``iam_mysql_root_password`` and ``im_mysql_password``.

.. warning::

   Current supported Ansible version: 2.2.3.0

Run the role using the ``ansible-playbook`` command:

::

  # cd indigopaas-deploy/ansible 

  # ansible-playbook -i inventory/inventory playbooks/deploy-im.yml

Video tutorial
--------------

.. raw:: html

   <a href="https://asciinema.org/a/pWo4o5QMhTawQ4aXIwUamOKOo" target="_blank"><img src="https://asciinema.org/a/pWo4o5QMhTawQ4aXIwUamOKOo.svg" /></a>

IM configuration
----------------

In order to allow IM to distinguish private from public networks, IM needs to be properly configured. Edit the IM configuration file ``/etc/im.cfg``, modifying the field `PRIVATE_NET_MASKS <https://imdocs.readthedocs.io/en/latest/manual.html#confval-PRIVATE_NET_MASKS>`_ with your favourite text editor, adding the network IP address. The IM will considers IPs not in these subnets as Public IPs.

::

  ...

  PRIVATE_NET_MASKS = 10.0.0.0/8,172.16.0.0/12,192.168.0.0/16,169.254.0.0/16,100.64.0.0/10,192.0.0.0/24,198.18.0.0/15,192.169.0.0/16 
  
  ...

IM testing
----------

#. Get IAM access token (see section :doc:`iam_token`)

#. Download an IM tosca template

   ::

    # mkdir im_test

    # cd im_test

    # wget https://raw.githubusercontent.com/Laniakea-elixir-it/IM-templates/devel/node_with_image.yaml

#. Configure the image url as ``ost://<keystone_url>/<image_id>``, as for example:

   ::

     image: ost://cloud.recas.ba.infn.it/f38d4e87-cc7e-4035-921b-6b200a9ebaee

   save and exit.

POST
****

The POST request, istantiate a new deployment

::

  curl -k -H 'Content-type: text/yaml' -H 'AUTHORIZATION: type = InfrastructureManager; username = mtangaro; token = eyJraWQiOiJyc2ExIiwiYWxnIjoiUlMyNTYifQ.eyJzdWIiOiJhOGJjZmU0OS1hOWY3LTQzMDctYWIzYS0wMmMyYmMzZWUxMTgiLCJpc3MiOiJodHRwczpcL1wvY2xvdWQtOTAtMTQ3LTc1LTIwNy5jbG91ZC5iYS5pbmZuLml0XC8iLCJleHAiOjE1NzAxMzIwNDYsImlhdCI6MTU3MDEyODQ0NiwianRpIjoiYmI5NjM4MmUtOGU5ZS00NmZmLWI2YzYtNWJkNGU1ZTFjZTRmIn0.OKqmt8NvUFWY22ui092yMPTIqCeGuyzjUfVAWllTeoZF-ea50RS91qSIHV8AW-O1AZSg4tM5O4W49jVSzvzVq4gLJEMKhBojaJSe9tVf0HE2REcfCb1pYi70jLBhC2TF-tiAmcb0ZywFcF3VEP8DhcPFrbd_JoiG0_q-vVtzcF4\nid = ost; type = OpenStack; host = <keystone_url>; username = <username>; password = ***** ; tenant = <tenant_name>; service_region = <region>' -i -X POST https://cloud-90-147-75-119.cloud.ba.infn.it/im/infrastructures --data-binary "@node_with_image.yaml"

  HTTP/1.1 100 Continue

  HTTP/1.1 200 OK
  Server: nginx/1.10.3 (Ubuntu)
  Date: Thu, 03 Oct 2019 15:54:37 GMT
  Content-Type: text/uri-list
  Content-Length: 100
  Connection: keep-alive
  Infid: c90796fe-e5f5-11e9-930c-fa163eefe815

  https://cloud-90-147-75-119.cloud.ba.infn.it/im/infrastructures/c90796fe-e5f5-11e9-930c-fa163eefe815

Where Infid, in this case ``a9feb488-e5f3-11e9-aafa-fa163eefe815``, is the IM UUID of your deployment

.. figure:: _static/im/im_openstack_post.png
   :scale: 50%
   :align: center

GET
***

The GET request can be used to list the VMs associated to a deployment:

::

  # curl -k -H 'Content-type: text/yaml' -H 'AUTHORIZATION: type = InfrastructureManager; username = mtangaro; token = eyJraWQiOiJyc2ExIiwiYWxnIjoiUlMyNTYifQ.eyJzdWIiOiJhOGJjZmU0OS1hOWY3LTQzMDctYWIzYS0wMmMyYmMzZWUxMTgiLCJpc3MiOiJodHRwczpcL1wvY2xvdWQtOTAtMTQ3LTc1LTIwNy5jbG91ZC5iYS5pbmZuLml0XC8iLCJleHAiOjE1NzAxMzIwNDYsImlhdCI6MTU3MDEyODQ0NiwianRpIjoiYmI5NjM4MmUtOGU5ZS00NmZmLWI2YzYtNWJkNGU1ZTFjZTRmIn0.OKqmt8NvUFWY22ui092yMPTIqCeGuyzjUfVAWllTeoZF-ea50RS91qSIHV8AW-O1AZSg4tM5O4W49jVSzvzVq4gLJEMKhBojaJSe9tVf0HE2REcfCb1pYi70jLBhC2TF-tiAmcb0ZywFcF3VEP8DhcPFrbd_JoiG0_q-vVtzcF4\nid = ost; type = OpenStack; host = <keystone_url>; username = <username>; password = ***** ; tenant = <tenant_name>; service_region = <region>' -i -X GET https://cloud-90-147-75-119.cloud.ba.infn.it/im/infrastructures/c90796fe-e5f5-11e9-930c-fa163eefe815
  HTTP/1.1 200 OK
  Server: nginx/1.10.3 (Ubuntu)
  Date: Thu, 03 Oct 2019 18:49:43 GMT
  Content-Type: text/uri-list
  Content-Length: 106
  Connection: keep-alive

  https://cloud-90-147-75-119.cloud.ba.infn.it/im/infrastructures/c90796fe-e5f5-11e9-930c-fa163eefe815/vms/0

The GET request can be used to list all VMs information:

::

  # curl -k -H 'Content-type: text/yaml' -H 'AUTHORIZATION: type = InfrastructureManager; username = mtangaro; token = eyJraWQiOiJyc2ExIiwiYWxnIjoiUlMyNTYifQ.eyJzdWIiOiJhOGJjZmU0OS1hOWY3LTQzMDctYWIzYS0wMmMyYmMzZWUxMTgiLCJpc3MiOiJodHRwczpcL1wvY2xvdWQtOTAtMTQ3LTc1LTIwNy5jbG91ZC5iYS5pbmZuLml0XC8iLCJleHAiOjE1NzAxMzIwNDYsImlhdCI6MTU3MDEyODQ0NiwianRpIjoiYmI5NjM4MmUtOGU5ZS00NmZmLWI2YzYtNWJkNGU1ZTFjZTRmIn0.OKqmt8NvUFWY22ui092yMPTIqCeGuyzjUfVAWllTeoZF-ea50RS91qSIHV8AW-O1AZSg4tM5O4W49jVSzvzVq4gLJEMKhBojaJSe9tVf0HE2REcfCb1pYi70jLBhC2TF-tiAmcb0ZywFcF3VEP8DhcPFrbd_JoiG0_q-vVtzcF4\nid = ost; type = OpenStack; host = <keystone_url>; username = <username>; password = ***** ; tenant = <tenant_name>; service_region = <region>' -i -X GET https://cloud-90-147-75-119.cloud.ba.infn.it/im/infrastructures/c90796fe-e5f5-11e9-930c-fa163eefe815/vms/0

  HTTP/1.1 200 OK
  Server: nginx/1.10.3 (Ubuntu)
  Date: Thu, 03 Oct 2019 18:52:38 GMT
  Content-Type: text/plain
  Content-Length: 2476
  Connection: keep-alive
  
  network public_net ( outports = '9001/tcp-9001/tcp,9000/tcp-9000/tcp' and
  provider_id = 'public_net' and
  outbound = 'yes' )
  system simple_node (
  instance_name = 'userimage-157011807495' and
  cpu.arch = 'x86_64' and
  disk.0.image.url = 'ost://cloud.recas.ba.infn.it/f38d4e87-cc7e-4035-921b-6b200a9ebaee' and
  net_interface.0.ip = '90.147.75.76' and
  memory.size = 2048M and
  cpu.count = 1 and
  disk.0.os.credentials.private_key = '-----BEGIN RSA PRIVATE KEY-----
  MIIEqAIBAAKCAQEAmNLLui9dXce/1XAj21inN5K4zrpgtst7cAJmZwnbIrVqEiNa
  q60MhINASHP5VR0HQpMqWuC1dlDE09XGp6qGzPa1+RFn894j5jd9X/H/HFbvMYN4
  DFq5AF+Lwj0AkCQT4+R/9iYYJbjuZug3UflAspCYzg7Ht94lVRNAzhlCM++96kkO
  j9jNxI5enX+MdKA0n1mOVhAyRi3wtfaQmhk2q47R1X9URqeE8UaZf6xL9KincVb/
  X94Wnc0dtbQfyHsNWM/Oo78pkrSfKxUNHC18Em/ZfJ+ADm7u27rY+V2eiKK+kahm
  8PCvOGO3qblBqwcnPUh/clVm5JGaiLal/keDlQIDAQABAoIBAAnjsj1VLVSRRY+5
  VwitvvxwqTbvhqytlEpWTWwjjiO726Za1VZAt4untrQ5lQv1+e9L+LSyz+tdJK+U
  qOtWtKx01qfMgY6ddHNEaf+YeGrMEWSB3nXmNQyaIkAqlGu/ee4IbmNuaaefRQYx
  xsquN4qWotzKxg/W91F/EnWD2u3jXyxOAOmRFBy5y1pU9YhcDR8w46+ZyV7h04f8
  hFbJILYA5kzmFtwHScUq5yGLlcddDGSK40EGJNpni4gNh61D4DOD/yzCrgqhL+th
  wfwSMOVhxWPBKOqlQDHqOyb21TVc+5UeFBwb+3LbfCdjfA7Sfi4Dpygvv1FPELCl
  ZGF1+0ECggCBAMUi+q15uresVXCyQrQ9HmZ2FcRwNc9BtB0ag5RuuuFNsh4suPcL
  hxJVG35vTfRgf9USO2WzCrgiAHzij6yT/USIoAFOUvLrtg+T5abd6Fec3lrvXgsL
  LLVX0NPK0RVqKhTAgNzEAqGEOkd8Ew3WWH0Klrwr3uxp1sEO8I3kt8/RAoIAgQDG
  dImkibakryLFd833OWdG33ClWT0kgFRBerq8taHZjdBejze9n67LzJludW77lqUQ
  VCpH424lxP7qIT+hNs/pFXi9Sq/VBsbfehwPoetDgv0yKSP1mRHiKOvTu47hHdst
  4q4iwxuYENLBjjESMKR2nge1pJMe2EUFURWHx87MhQKCAIAvp/QXqbzEmCmTc9SC
  Q+AsftFmSoYHk2eaPYWfhWEyBBlSCBeyyRufB+n8l6WttQJSHPU08aJevwGFLzPy
  UVhBkBG2HxwYU3kQrP0waKa5P1fVfdYrL0lgkVkPShFfbum7WIoOVGgaaZ+5Fjp4
  9t8vYzbrSGO8nR1oUFdAxhDVcQKCAIEAspZsxwSmt8xjHhCR6MhfiAfK9wE3ZIGX
  UNWA9hD9dSmJOY7oOlxYkE2uRRiopv8Jy4fyBH9Fv/dm7oq9F/abYsVPwghT8wAG
  N1VLq0Wq0TYvY9Rh58G3ti3dCszd5vdXJhO3YNDzJAT/o+6xeg0L8zKC/ZL8UeWN
  NxugpG/KSYECggCAbcJeVFjNQYEhroRg2dmVY/Y6cmndvCUudDs8hvtTmvWmFGri
  7dY1T7ACdWAbFYh+Q1x2SswHAOXC+FYJ2HJ8InbKeRAlQ7KDgDsofPGRCTRUL9HO
  mZQkIZqryAcSnC++OLNnbFGsTY4vhyotb3IgR/pC+6RSgqJFabFtA7Ttkgg=
  -----END RSA PRIVATE KEY-----
  ' and
  provider.host = 'cloud.recas.ba.infn.it' and
  disk.0.free_size = 20G and
  instance_id = 'c6e54a1e-f2ce-4cd5-a38f-f26858d57d7c' and
  instance_type = 'small' and
  state = 'unconfigured' and
  provider.port = 5000 and
  provider.type = 'OpenStack' and
  net_interface.0.connection = 'public_net' and
  disk.0.os.name = 'linux' and
  disk.0.os.credentials.username = 'cloudadm'
  )

DELETE
******

The DELETE request can be used to delete the deployment:

::

  # curl -k -H 'Content-type: text/yaml' -H 'AUTHORIZATION: type = InfrastructureManager; username = mtangaro; token = eyJraWQiOiJyc2ExIiwiYWxnIjoiUlMyNTYifQ.eyJzdWIiOiJhOGJjZmU0OS1hOWY3LTQzMDctYWIzYS0wMmMyYmMzZWUxMTgiLCJpc3MiOiJodHRwczpcL1wvY2xvdWQtOTAtMTQ3LTc1LTIwNy5jbG91ZC5iYS5pbmZuLml0XC8iLCJleHAiOjE1NzAxMzIwNDYsImlhdCI6MTU3MDEyODQ0NiwianRpIjoiYmI5NjM4MmUtOGU5ZS00NmZmLWI2YzYtNWJkNGU1ZTFjZTRmIn0.OKqmt8NvUFWY22ui092yMPTIqCeGuyzjUfVAWllTeoZF-ea50RS91qSIHV8AW-O1AZSg4tM5O4W49jVSzvzVq4gLJEMKhBojaJSe9tVf0HE2REcfCb1pYi70jLBhC2TF-tiAmcb0ZywFcF3VEP8DhcPFrbd_JoiG0_q-vVtzcF4\nid = ost; type = OpenStack; host = <keystone_url>; username = <username>; password = ***** ; tenant = <tenant_name>; service_region = <region>' -i -X DELETE https://cloud-90-147-75-119.cloud.ba.infn.it/im/infrastructures/c90796fe-e5f5-11e9-930c-fa163eefe815

  HTTP/1.1 200 OK
  Server: nginx/1.10.3 (Ubuntu)
  Date: Thu, 03 Oct 2019 15:43:52 GMT
  Content-Type: text/plain
  Content-Length: 0
  Connection: keep-alive

Test IM using OIDC
******************

It is possible to use an OIDC Token with IM for POST, GET and DELETE calls:

.. note::

   Please note in this case that the ``username`` parameter in the API call must be set to IAM organization name. For example, in the following, we used as IAM organization name ``laniakea`` and the username has been set accordingly.

--------
``POST``
--------

::

  export IAM_ACCESS_TOKEN="..."

  curl -k -H 'Content-type: text/yaml' -H "Authorization: id = ost; type = OpenStack; host = https://cloud.recas.ba.infn.it:5000/; username = laniakea; password = $IAM_ACCESS_TOKEN; tenant = oidc; auth_version = 3.x_oidc_access_token; service_region = recas-cloud;\nid = im; type = InfrastructureManager; token = $IAM_ACCESS_TOKEN" -i -X POST https://cloud-90-147-75-119.cloud.ba.infn.it/im/infrastructures --data-binary "@node_with_image.yaml"

-------
``GET``
-------

::

  export IAM_ACCESS_TOKEN="..."

  curl -k -H 'Content-type: text/yaml' -H "Authorization: id = ost; type = OpenStack; host = https://cloud.recas.ba.infn.it:5000/; username = laniakea; password = $IAM_ACCESS_TOKEN; tenant = oidc; auth_version = 3.x_oidc_access_token; service_region = recas-cloud;\nid = im; type = InfrastructureManager; token = $IAM_ACCESS_TOKEN" -i -X GET https://cloud-90-147-75-119.cloud.ba.infn.it/im/infrastructures

---------
``DELETE``
---------

::

  export IAM_ACCESS_TOKEN="..."

  curl -k -H 'Content-type: text/yaml' -H "Authorization: id = ost; type = OpenStack; host = https://cloud.recas.ba.infn.it:5000/; username = laniakea; password = $IAM_ACCESS_TOKEN; tenant = oidc; auth_version = 3.x_oidc_access_token; service_region = recas-cloud;\nid = im; type = InfrastructureManager; token = $IAM_ACCESS_TOKEN" -i -X DELETE https://cloud-90-147-75-119.cloud.ba.infn.it/im/infrastructures/<infrastructure_uuid>

FAQ
---

.. toctree::
   :maxdepth: 2

   im_faq

References
----------

`IM configuration <https://imdocs.readthedocs.io/en/latest/manual.html#configuration>`_

`IM APIs documentation <https://imdocs.readthedocs.io/en/latest/REST.html>`_
