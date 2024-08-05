Applications deploy under VPN
=============================

Laniakea provides the possibility to deploy its applications as VPN isolated environments usin just private networks.

Indeed the access is grant only through VPN authentication, using the same Laniakea credentials. 

Therefore only users authrised using the Laniakea authentication system can access the application server.

.. seealso::

   To login to the Laniakea dashboard visit the section: :doc:`/user_documentation//authentication/authentication`.

In the following we describe how to deploy Galaxy under VPN and exploit it. The step are identical for any other application on Laniakea.

.. note::

   The step are identical for any other application on Laniakea.

To deploy and application under VPN select among those available:

.. figure:: img/vpn_deployment_select.png
   :scale: 40 %
   :align: center

The deployment will follow as usual. Once the Deployment is complete click on ``details`` button, and navigate to ``Output values``.

   .. figure:: img/vpn_deployment_complete.png
   :scale: 20 %
   :align: center

.. figure:: img/vpn_deployment_output_values.png
   :scale: 30 %
   :align: center

Here if you click the Galaxy url, it is not possible to access it since it is not available.

   .. figure:: img/vpn_deployment_galaxy_fail.png
   :scale: 20 %
   :align: center

To access it, save the ovpn file on your computer. Please click on the ``Save Link As`` button and select ``Save Link As``.

.. figure:: img/vpn_deployment_save_ovpn_file.png
   :scale: 30 %
   :align: center

In this case we are using `tunnelblick <https://tunnelblick.net/>`_, which is available for OSX and Linux systems. Import the OVPN file on your client.

.. figure:: img/vpn_deployment_ovpn_file_import.png
   :scale: 30 %
   :align: center

Type your e-mail, the one used to register on Laniakea.

.. warning::

   Only the e-mail is needed, not the passoword! Leave any other filed blank.

.. figure:: img/vpn_deployment_tunnelblick_creds.png
   :scale: 30 %
   :align: center

You will receive an e-mail with authentication url. Click on it ...

.. figure:: img/vpn_deployment_mail.png
   :scale: 30 %
   :align: center

... authenticate and authorize the client.

.. figure:: img/vpn_deployment_client_authorization.png
   :scale: 20 %
   :align: center

Finally, you'll see on tunnelblick that your VPN tunnel is working fine.

.. figure:: img/vpn_deployment_tunnelblick_ok.png
   :scale: 50 %
   :align: center

And you can access Galaxy.

.. figure:: img/vpn_deployment_galaxy_ok.png
   :scale: 20 %
   :align: center
