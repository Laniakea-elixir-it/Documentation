Cryptsetup hints
================

The cryptsetup action to set up a new dm-crypt device in LUKS encryption mode is luksFormat:

::

  cryptsetup -v --cipher aes-xts-plain64 --key-size 256 --hash sha 256 --iter-time 2000 --use-urandom --verify-passphrase luksFormat crypt --batch-mode

where ``crypt`` is the new device located to ``/dev/mapper/crypt``.

To open and mount to ``/export``  an encrypted device:

::

  cryptsetup luksOpen /dev/vdb crypt

  mount /dev/mapper/crypt /export

To show LUKS device info:

::

  dmsetup info /dev/mapper/crypt

To umount and close an encrypted device:

::

  umount /export

  cryptsetup close crypt

To force LUKS volume removal:

::

  dmsetup remove /dev/mapper/crypt

.. note::

   Run as root.

Change LUKS password
--------------------

LUKS provides 8 slots for passwords or key files. First, check, which of them are used:

::

  cryptsetup luksDump /dev/<device> | grep Slot

where the output, for example, looks like:

::

  Key Slot 0: ENABLED
  Key Slot 1: DISABLED
  Key Slot 2: DISABLED
  Key Slot 3: DISABLED
  Key Slot 4: DISABLED
  Key Slot 5: DISABLED
  Key Slot 6: DISABLED
  Key Slot 7: DISABLED

Then you can add, change or delete chosen keys:

::

  cryptsetup luksAddKey /dev/<device> (/path/to/<additionalkeyfile>) 

  cryptsetup luksChangeKey /dev/<device> -S 6

As for deleting keys, you have 2 options:

#. delete any key that matches your entered password:

   ::

     cryptsetup luksRemoveKey /dev/<device>

#. delete a key in specified slot:

   ::

     cryptsetup luksKillSlot /dev/<device> 6
