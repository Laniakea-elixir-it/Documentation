Storage encryption workflow
===========================

When the storage encrpyptions is required by the user the following workflow is triggered:

#. All required software are installed, e.g. cryptsetup.

#. A strong alphanumerical passphrase is generated (100 characters long).

#. The storage is encrypted. Laniakea adopts, by default,  ``xts-aes-plain64`` cipher with ``256`` bit keys ans ``sha256`` hashing algorithm.

   ::
   
     # Defaults values

     cipher_algorithm='aes-xts-plain64'
     keysize='256'
     hash_algorithm='sha256'
     device='/dev/vdb'
     cryptdev='crypt'
     mountpoint='/export'
     filesystem='ext4'

#. The passphrase is uploaded on Vault, allowing user to retrieve it through the Laniakea dashboard.

#. Once the LUKS partition is created, it is unlocked.

   The unlocking process will map the partition to a new device name using the device mapper. This alerts the kernel that device is actually an encrypted device and should be addressed through LUKS using the ``/dev/mapper/<cryptdev_name>`` so as not to overwrite the encrypted data. ``cryptdev_name`` is random generated to avoid accidental overwriting.

#. The volume is mounted, by default, on ``/export``, with standard ``ext4`` filesystem and Galaxy is configured to store here datasets.
