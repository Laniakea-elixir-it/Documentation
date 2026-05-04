Enable Laniakea integration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once the OpenVPN part is configured, we need to teach IM and the PaaS how to exploit it.

When IM is installed and configured a SSH key pair is created and mounted in the IM Docker container, whose path is:

::

  # ll /etc/im/.ssh/

  ...
  -rw------- 1 root root 3357 Sep 20  2023 id_rsa
  -rw-r--r-- 1 root root  726 Sep 20  2023 id_rsa.pub

The public key has to be configured on the jump host. So login on the jump host VM. Then create a ``im`` user:

::

  useradd -m im

Log in as the new user

::

  su - im

Add the public key to the authorized_keys file:

::

  mkdir .ssh
  
  vim authorized_keys

Finally, you should be able to connect from the IM machine to the jump host with the command

::

  ssh -i /etc/im/.ssh/id_rsa im@<JUMP_HOST_PUBLIC_IP>

Now that we teached IM how to login in the Jump Host to access the tenant private network, we need to teach the PaaS that, if the deployment is only on the private network, IM has to use the jump host to access it.

This is done at tenant level via CMDB, adding two entries to the tenant:

::

  ...
  "private_network_proxy_user": "im",
  "private_network_proxy_host": "<JUMP HOST PUBLIC IP>"
  ...

with the command:

::

 curl -X PUT http://cmdb:********@localhost:5984/indigo-cmdb-v2/<TENANT CMDB ID> -H "Content-Type: application/json" -d@tenant_update.json

where tenat_update.json looks like:

::

  {
  "_id": "ce7fa82f858c3a182288eff7650040ca",
  "_rev": "1-6b1ac50c5532a5ee8cad48d482ff5316",
  "data": {
    "tenant_id": "3b38073bf9e04049bf0cab08b2c1c9a0",
    "service": "service-RECAS-BARI-openstack",
    "tenant_name": "ELIXIR-PAAS",
    "private_network_name": "private_net",
    "public_network_name": "public_net",
    "private_network_proxy_user": "im",
    "private_network_proxy_host": "<JUMP HOST PUBLIC IP>",
    "iam_organisation": "ELIXIR-PAAS"
  },
  "type": "tenant"

The resulting output is, for example:

::

  {
    "id": "ce7fa82f858c3a182288eff7650040ca",
    "key": [
      "tenant"
    ],
    "value": {
      "tenant_id": "3b38073bf9e04049bf0cab08b2c1c9a0",
      "tenant_name": "ELIXIR-PAAS",
      "iam_organisation": "ELIXIR-PAAS"
    },
    "doc": {
      "_id": "ce7fa82f858c3a182288eff7650040ca",
      "_rev": "2-d423458cf3f8a0747370dce0498b806c",
      "data": {
        "tenant_id": "3b38073bf9e04049bf0cab08b2c1c9a0",
        "service": "service-RECAS-BARI-openstack",
        "tenant_name": "ELIXIR-PAAS",
        "private_network_name": "private_net",
        "public_network_name": "public_net",
        "private_network_proxy_user": "im",
        "private_network_proxy_host": "<JUMP_HOST_PUBLIC_IP>",
        "iam_organisation": "ELIXIR-PAAS"
      },
      "type": "tenant"
    }
  }
