Manage an encrypted instance
============================

Laniakea provides the possibility to encrypt the storage volume associated to the virtual machine on-demand.

A detailed description of Laniakea encryption strategy is reported here: :doc:`/admin_documentation/encryption/encryption`.

.. Warning::

   Only the external volume, where Galaxy data are stored, is encrypted, not the Virtual Machine root disk. The encryption layer should be secure enough to protect data uploaded from users to the Galaxy instance from any unwantend attention. However, users must be aware that the responsibility of correctly handling any sensitive data they upload to Laniakea falls on them and that the administrators of the Laniakea service can not be considered responsible for any data breach that may happen due to neglicence by Galaxy users or the action of external malicious attackers.



Retrieve the encrypted storage passphrase
-----------------------------------------

Cryptographic keys should never be transmitted in the clear. For this reason Laniakea encrypt your storage with a strong alphanumerical random passphrase.

This passphrase can be easily retrieved thorugh the dashboard.

.. Warning::

   If you require the storage encryption, please retrieve your passphrase as soon as possible and keep it secret.

#. Connect to the dashboard and click on the name of your encrypted instance.

#. In the overview tab, click on ``Retrieve LUKS passphrase`` button.

#. Copy your passphtase.

.. raw:: html

   <p align="center">
   <iframe width="640" height="360"
      src="https://www.youtube.com/embed/G_4aIqkXZTM">
   </iframe> 
   </p>

Restart Galaxy on an encrypted instance
---------------------------------------

In case of reboot of yout virtual instance, the encrypted storage cannot be automatically enabled again, since the encryption passphrase is needed. The user intervention is needed.

It is possible to do this through the dashboard.

#. Connect to the dashboard and click on the name of your encrypted instance.

#. In the overview tab, the button ``Unlock and mount volulme`` is available only if the encrypted storage is not mounted. Click it to unlock

#. It is now possible to restart Galaxy. The button ``Try to restart Galaxy`` will be enabled only if the encrypted storage is correctly mounted, avoiding to start Galaxy without user data.

.. raw:: html

   <p align="center">
   <iframe width="640" height="360"
      src="https://www.youtube.com/embed/g37X8jDSkG4">
   </iframe> 
   </p>

.. note::

   If the automatic procecure does not work, please have a look here: :doc:`/user_documentation/faq/faq`

Command line interface: luksctl
-------------------------------

To easily the encrypted storage management a python script, ``luksctl``, is installed. 

By default its configuration file is stored in ``/etc/luks/luks-cryptdev.ini``.

.. Warning::

   Please don't change it unless you know what you're doing.


.. Note::

   The script requires superuser rights.

Here the list of the currently available commands:

========  ======================  =========================
Action    Command                 Description
========  ======================  =========================
Open      sudo luksctl open       Open the encrypted device, requiring your passphrase.
Close     sudo luksctl close      Close and umount the encrypted device
Status    sudo luksctl status     Check device status
========  ======================  =========================

