Fast-luks script
================

The `fast-luks <https://github.com/Laniakea-elixir-it/fast-luks>`_ bash script is responsible for Laniakea Storage encryption. It parse common cryptsetup parameters to encrypt the volume. For this reason it checks for cryptsetup and dm-setup packages and it install cryptsetup, if not installed.

The default encryption parameters are:

::

  cipher_algorithm: aes-xts-plain64
  keysize: 256
  hash_algorithm: sha256
  device: /dev/vdb
  cryptdev: crypt [this is randomly generated]
  mountpoint: /export
  filesystem: ext4

From version ``v3.0.1`` Hashicorp Vault support has been integrated. It exploits a Vault token with the right write policy only, which can be used only one time and for a limited time duration (currently configured to expire after 12 hours), to store user secret passphrases. A temporary python virtual environment is created allowing fast-luks to store secrets on vault and then it is deleted.

The ``fast-luks`` script is automatically downloaded in ``/home/galaxy/laniakea_utils/fast-luks``. 

Full documentation on fast-luks script is hosted `here <https://github.com/Laniakea-elixir-it/fast-luks>`_. 

.. note::

   The script requires superuser rights.

