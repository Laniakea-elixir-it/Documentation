Applications deploy under VPN
=============================

Laniakea provides the possibility to deploy its applications as VPN isolated environments usin just private networks.

Indeed the access is grant only through VPN authentication, using the same Laniakea credentials. 

Therefore only users authrised using the Laniakea authentication system can access the application server.

In the following we describe how to deploy and exploit these deployments.

.. seealso::

   To login to the Laniakea dashboard visit the section: :doc:`/user_documentation//authentication/authentication`.

Once the Deployment is complete, navigate to Output values

.. figure:: img/vpn_deployment_select.png
   :scale: 40 %
   :align: center

.. figure:: img/vpn_deployment_complete.png
   :scale: 20 %
   :align: center

.. figure:: img/vpn_deployment_output_values.png
   :scale: 30 %
   :align: center

.. figure:: img/vpn_deployment_galaxy_fail.png
   :scale: 20 %
   :align: center

.. figure:: img/vpn_deployment_save_ovpn_file.png
   :scale: 30 %
   :align: center

.. figure:: img/vpn_deployment_ovpn_file_import.png
   :scale: 30 %
   :align: center

.. figure:: img/vpn_deployment_tunnelblick_creds.png
   :scale: 30 %
   :align: center

.. figure:: img/vpn_deployment_mail.png
   :scale: 30 %
   :align: center

.. figure:: img/vpn_deployment_client_authorization.png
   :scale: 20 %
   :align: center

.. figure:: img/vpn_deployment_tunnelblick_ok.png
   :scale: 50 %
   :align: center

.. figure:: img/vpn_deployment_galaxy_ok.png
   :scale: 20 %
   :align: center

In this case we are using tunnelblick, which is available for McOS and Linux systems
