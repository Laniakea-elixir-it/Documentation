Hashicorp Vault
==================

Vault is exploited as secrets management store, to store and manage encryption passphrases

.. note::

   Current version: 1.1.2

VM configuration
----------------

Create a VM for Vault. The VM should meet the following minimum requirements:

======= ==============================
OS      Ubuntu 16.04
vCPUs   2
RAM     4 GB
Network Public IP address.
======= ==============================

.. warning::

   All the command will be run from the control machine VM.

Installation
------------

Create the file ``indigopaas-deploy/ansible/inventory/group_vars/vault.yaml`` with the following configured values:

::

  vault_fqdn: <dashboard_vm_dns_name>
  vault_image_name: vault:1.1.2
  vault_letsencrypt_email: "<valid_email_address>"

.. warning::

   Depending on your Cloud Provider network configuration, the ``vault_host`` variable needs to be added and configured with the private ip address associated to the VM, for example when a floating IP is used.

   In this case it is possible to set the IP address adding:

   ::

     vault_host: '<vm_private_ip_address>'

Run the role using the ``ansible-playbook`` command:

::

  # cd indigopaas-deploy/ansible 

  # ansible-playbook -i inventory/inventory playbooks/deploy-vault.yml

Installation video tutorial
***************************

.. raw:: html

   <a href="https://asciinema.org/a/W7zKE6Mc9uBjKUOqC30vkJDtb" target="_blank"><img src="https://asciinema.org/a/W7zKE6Mc9uBjKUOqC30vkJDtb.svg" /></a>


Vault initialization
--------------------

The Vault initialization can not be automated. To initialize it and get your root token for the initial configuration 


#. Login on the VM hosting Vault:

   ::

     ssh root@<vault_vm_ip_address>


#. Initialize Vault:

   ::

     # docker exec -it vault vault operator init
     Unseal Key 1: p7YF7vyLRrfeilwlD/QusQ+UESJiGrhn1TwCsBAa7fKV
     Unseal Key 2: OHoyPApMFuQTz9B20bmpJjzLgkCi2ELr+zKFdvKq8lmL
     Unseal Key 3: xDRcbkOsYL9uswFzCdFqpxudgvZFVfAwFCkigYMMMCHt
     Unseal Key 4: LJ0hHW5dsmbuFAnL+W/4NMtZUbuNkILFWXxL3zTYblzQ
     Unseal Key 5: Z1OvJ7RvT+pUVtqB93RAQ8q1s8l04clGVFn+oi22x4rZ
     
     Initial Root Token: s.YxsTl9H3f1qgAqH3cj4JAXR8
     
     Vault initialized with 5 key shares and a key threshold of 3. Please securely
     distribute the key shares printed above. When the Vault is re-sealed,
     restarted, or stopped, you must supply at least 3 of these keys to unseal it
     before it can start servicing requests.
     
     Vault does not store the generated master key. Without at least 3 key to
     reconstruct the master key, Vault will remain permanently sealed!
     
     It is possible to generate new unseal keys, provided you have a quorum of
     existing unseal keys shares. See "vault operator rekey" for more information.


#. Every initialized Vault server starts in the `sealed state <https://learn.hashicorp.com/vault/getting-started/deploy#sealunseal>`_. Unsealing has to happen every time Vault starts. It can be done via the API and via the command line. To unseal the Vault, you must have the threshold number of unseal keys. In the output above, notice that the "key threshold" is 3. This means that to unseal the Vault, you need 3 of the 5 keys that were generated.

   ::

     # docker exec -it vault vault operator unseal p7YF7vyLRrfeilwlD/QusQ+UESJiGrhn1TwCsBAa7fKV
     Key                Value
     ---                -----
     Seal Type          shamir
     Initialized        true
     Sealed             true
     Total Shares       5
     Threshold          3
     Unseal Progress    1/3
     Unseal Nonce       7a0891bb-7d0e-6efa-2081-9c60941f9a6d
     Version            1.1.2
     HA Enabled         false
     
     # docker exec -it vault vault operator unseal OHoyPApMFuQTz9B20bmpJjzLgkCi2ELr+zKFdvKq8lmL
     Key                Value
     ---                -----
     Seal Type          shamir
     Initialized        true
     Sealed             true
     Total Shares       5
     Threshold          3
     Unseal Progress    2/3
     Unseal Nonce       7a0891bb-7d0e-6efa-2081-9c60941f9a6d
     Version            1.1.2
     HA Enabled         false
     
     # docker exec -it vault vault operator unseal xDRcbkOsYL9uswFzCdFqpxudgvZFVfAwFCkigYMMMCHt
     Key             Value
     ---             -----
     Seal Type       shamir
     Initialized     true
     Sealed          false
     Total Shares    5
     Threshold       3
     Version         1.1.2
     Cluster Name    vault-cluster-e6688ec2
     Cluster ID      ccf2e852-69ca-bcd6-0079-6c820f9c0e67
     HA Enabled      false

#. Finally, authenticate as the initial root token (it was included in the output with the unseal keys):

   ::

     # docker exec -it vault vault login s.YxsTl9H3f1qgAqH3cj4JAXR8
     Success! You are now authenticated. The token information displayed below
     is already stored in the token helper. You do NOT need to run "vault login"
     again. Future Vault requests will automatically use this token.
     
     Key                  Value
     ---                  -----
     token                s.YxsTl9H3f1qgAqH3cj4JAXR8
     token_accessor       QEUBU4tepPWDatRu6jrnTbFW
     token_duration       ∞
     token_renewable      false
     token_policies       ["root"]
     identity_policies    []
     policies             ["root"]

.. warning::

   Save the unseal keys and the root token. Please read `Vault documentation <https://www.vaultproject.io/docs/concepts/seal.html>`_.

Initialization video tutorial
*****************************

.. raw:: html

   <a href="https://asciinema.org/a/7FDywKYg4BjkclD55imWsgtG9" target="_blank"><img src="https://asciinema.org/a/7FDywKYg4BjkclD55imWsgtG9.svg" /></a>

References
----------

`Vault documentation <https://learn.hashicorp.com/vault/getting-started/deploy#initializing-the-vault>`_
