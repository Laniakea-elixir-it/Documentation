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

Two possibilities are here explained.

OpenVPN Connect
~~~~~~~~~~~~~~~

Now you need to install the official client application that enables you to securely access network resources. Here we strongly suggest **OpenVPN Connect**, available on Windows, MacOS and Linux. The tutorial will show openVPN Connect steps.

.. note::
   **OpenVPN Connect** is not the only valid client option, **Tunnelblick** can also be used on **macOS** (but not on Windows).  
   However, it is essential to use a client that does **not** prompt for a password before authentication (Later in this guide, we will cover this topic in more detail),  
   so that the login verification code can be received by e-mail.

Please visit the `official OpenVPN CLient page <https://openvpn.net/client/>`_ to download the client.
Once you have installed and opened the client you should see the following window:

.. figure:: img/openvpn_connect_cloud_connection.png
   :scale: 30%
   :align: center

Here, at the bottom of the window, you need to upload a **.ovpn** file, e.g the one that you create in `the previous script <#enable-pam-plugin>`_: ``client.ovpn``.
Once you have uploaded the file, you will be redirected to the following page:

.. figure:: img/openvpn_ovpn_uploaded.png
   :scale: 30%
   :align: center

Now is sufficent to click connect and compile the following fields:

.. warning::
   **Important:** We use this client (**Tunnelblick** also supports this feature) because it allows the user to start the authentication process without requiring a real password.  
   When filling in the login fields, you can enter **any string** as the password , **it is not used for verification**.  
   The only mandatory field is your **e-mail address**, where the authentication code will be sent.  
   The password field cannot be left empty, but it can contain any value (e.g. ``aaaaaaa`` or ``password``) and can be changed freely at any login.

.. figure:: img/openvpn_login.png
   :scale: 30% 
   :align: center


.. tip::
   When you first configure the connection in **OpenVPN Connect**, you may see the following prompt:

   .. figure:: img/openvpn_missing_certificate.png
      :scale: 60%
      :align: center

      This message appears when the client expects an external certificate for authentication.  
      In our **example**, this step is not required, so we had simply disabled the *“Require External Certificate”* option in the connection profile. (you may want to keep it)

   Once disabled, you can proceed with the configuration as shown below:

   .. figure:: img/openvpn_disable_certificate.png
      :scale: 50%
      :align: center

      Make sure to:  
      - Disable the field: Require External Certificate
      - Click **Save Changes** to receive the email

Tunnelblick
~~~~~~~~~~~

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
