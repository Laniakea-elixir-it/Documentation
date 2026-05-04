Manual configuration
--------------------

Create a virtual machine (VM) that will be used as **jump host** in the tenant where you are planning to enable VPN deployment. 

.. note::
   Each tenant in your cloud should have its own dedicated jump host.

The VM should meet the following minimum requirements:

======= ==============================
OS      Ubuntu 22.04
vCPUs   2
RAM     4 GB
Network Public and private IP address
======= ==============================

PAM module installation and configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The PAM module enables OAuth2 device authentication for OpenVPN.  
We will install version ``0.0.4`` on **Ubuntu 22.04**.

.. note::

   Original installation instructions are available here: `click here <https://github.com/maricaantonacci/pam_oauth2_device#readme>`_

Please note:

1. The steps below are adapted and tested on **Ubuntu 22.04**.
2. Use the release `0.0.4` of the PAM module:  
   https://github.com/maricaantonacci/pam_oauth2_device/releases
3. When you create the IAM client, **do not leave the default “device code timeout” at 0 seconds**.  
   Set it explicitly to **300 seconds (5 minutes)**, otherwise it will expire immediately.

You can follow these steps or refer to the original istructions (remember to apply the suggested modification):

1. Download the repository:

   .. code-block:: bash

      wget https://github.com/maricaantonacci/pam_oauth2_device/archive/refs/tags/v0.0.4.tar.gz
      
      tar -xzf v0.0.4.tar.gz
      
      cd pam_oauth2_device-0.0.4

2. Install required tools:

   .. code-block:: bash

      sudo apt update
      
      sudo apt install -y build-essential libcurl4-openssl-dev libpam0g-dev

3. Build and install the module:

   .. code-block:: bash

      make
      
      sudo make install

Remember that when creating the OIDC client on your identity provider,  **do not** leave the default ``device code timeout`` at 0 seconds but set it to **300 seconds** 
(or any number > 0) to prevent immediate expiration.

.. warning::
   It is **essential** to include the correct scopes in your configuration file (e.g., ``/etc/pam_oauth2_device/config.json``) to ensure the module can retrieve the necessary user information:

   1. Use ``"scope": "openid profile eduperson_entitlement",``: if your Identity Provider (IdP) uses **Eduperson entitlements** for authorization.
   2. Use ``"scope": "openid profile",`` :if your IdP relies on standard **OIDC profiles** or internal IAM groups.

   **Example snippet (chose only one ``"scope":``):**
   
   .. code-block:: json

      "oauth": {
        "client": {
            "id": "YOUR_CLIENT_ID",
            "secret": "YOUR_CLIENT_SECRET"
        },
        "scope": "openid profile",  OR  "scope": "openid profile eduperson_entitlement",
        "device_endpoint":"https://endpoint/.../",
        ...
        }

OpenVPN installation
~~~~~~~~~~~~~~~~~~~~

.. note::

   OpenVPN **version >= 2.5** is required to enable the *deferred authentication* mechanism,
   which is needed by the PAM OAuth2 module.

1. The first step is to add the OpenVPN repository:

   .. code-block:: bash

      wget -O - https://swupdate.openvpn.net/repos/repo-public.gpg | sudo apt-key add -
      
      echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/openvpn-repo-public.gpg] https://build.openvpn.net/debian/openvpn/release/2.5 jammy main" > /etc/apt/sources.list.d/openvpn-aptrepo.list
      
      sudo apt update

2. Then you need to install OpenVPN:

   You can install and configure OpenVPN using the following `script <https://raw.githubusercontent.com/Nyr/openvpn-install/master/openvpn-install.sh>`_:  
  
   .. code-block:: bash

      wget https://raw.githubusercontent.com/Nyr/openvpn-install/master/openvpn-install.sh
      
      chmod +x openvpn-install.sh
      
      sudo ./openvpn-install.sh

   During the script execution, **be sure that the following options were selected**:

   - **Protocol:** TCP  
   - **Port:** 1194 (default)  
   - **Server name:** choose a name or press Enter to use the default one  

Once the script completes, your OpenVPN server will be installed and ready for configuration.

Server Certificates and OIDC configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following steps focus on generating the required server and client certificates, if you haven't done so already.
To procede you have to have installed OpenVPN ed Easy-RSA:

.. code-block:: bash 
   sudo apt update

   sudo apt install -y openvpn easy-rsa

OpenVPN requires a valid Public Key Infrastructure (PKI) for secure communication between the server and the clients.  

.. note::

   The following example uses **Easy-RSA** to generate the necessary certificates and keys.

You can start by  generating the certificates:

.. code-block:: bash

   make-cadir ~/openvpn-ca

   cd ~/openvpn-ca

   ./easyrsa init-pki

   ./easyrsa build-ca

   ./easyrsa gen-req bastion nopass

   ./easyrsa sign-req server bastion

   ./easyrsa gen-req <YOUR_CLIENT_NAME> 

   ./easyrsa sign-req client <YOUR_CLIENT_NAME>

Copy the required certificates and keys to OpenVPN directory:

.. code-block:: bash

   sudo mkdir -p /etc/openvpn/server

   sudo cp pki/ca.crt pki/issued/bastion.crt pki/private/bastion.key /etc/openvpn/server/

Generate the TLS key and certificate revocation list (CRL):

.. code-block:: bash

   sudo openvpn --genkey secret /etc/openvpn/server/tc.key

   ./easyrsa gen-crl

   sudo cp pki/crl.pem /etc/openvpn/server/

   sudo chown nobody:nogroup /etc/openvpn/server/crl.pem

.. _enable-pam-plugin:

Enable the PAM plugin
~~~~~~~~~~~~~~~~~~~~~

Create the file ``/etc/pam.d/openvpn`` with your preferred editor:

.. code-block:: bash

   vim /etc/pam.d/openvpn

Fill the file with the following content:

.. code-block:: bash

   auth     required    pam_oauth2_device.so
   account  sufficient  pam_oauth2_device.so

Then edit ``/etc/openvpn/server/server.conf`` to include the jump host's **public IP** and the **private network IP** range that should be accessible.

.. note::

   Lines marked below are critical for correct PAM + OAuth2 integration,  
   especially when using username/password authentication only (no client certificates).

A look to the file:

.. code-block:: bash

   local <PUBLIC IP OF THE JUMP HOST>
   port 1194
   proto tcp
   dev tun
   ca ca.crt
   cert server.crt
   key server.key
   dh dh.pem
   auth SHA512
   tls-crypt tc.key
   topology subnet
   server 10.8.0.0 255.255.255.0
   #push "redirect-gateway def1 bypass-dhcp"
   push "route <PRIVATE NETWORK> 255.255.255.0"
   ifconfig-pool-persist ipp.txt
   push "dhcp-option DNS 8.8.8.8"
   push "dhcp-option DNS 8.8.4.4"
   keepalive 10 120
   cipher AES-256-CBC
   user nobody
   group nogroup
   persist-key
   persist-tun
   verb 7
   crl-verify crl.pem
   plugin /usr/lib/x86_64-linux-gnu/openvpn/plugins/openvpn-plugin-auth-pam.so openvpn
   duplicate-cn
   setenv deferred_auth_pam 1
   reneg-sec 0
   hand-window 300
   username-as-common-name

Particular attention over key parameters:

- ``duplicate-cn``: Allow multiple clients with the same common name to connect concurrently. Without this option, OpenVPN disconnects any existing client instance when a new one with the same common name connects.
- ``setenv deferred_auth_pam 1``: Enable the deferred authentication method for PAM.
- ``reneg-sec 0``: Prevents users from being prompted to reauthorize every hour (default behavior).
- ``hand-window 300``: Increases the handshake window from 60 seconds to 300 seconds to accommodate possible email or token delays.
- ``username-as-common-name``: Use the authenticated username as the common name, rather than the one from the client certificate. This is required when using ``auth-user-pass`` on the client side.

Then we switch to the client configuration.

Update the ``client.ovpn`` file to use the jump host's **public IP** and include the required authentication parameters.

.. code-block:: bash

   client
   dev tun
   proto tcp
   remote <PUBLIC IP OF THE JUMP HOST> 1194
   resolv-retry infinite
   nobind
   persist-key
   persist-tun
   remote-cert-tls server
   auth SHA512
   cipher AES-256-CBC
   ignore-unknown-option block-outside-dns
   block-outside-dns
   verb 3
   auth-user-pass
   auth-nocache
   reneg-sec 0
   hand-window 300
   <ca>
   **...<INSERT_YOUR_CLIENT_CERTIFICATE_HERE>...**
   </ca>

Then apply the configuration by restarting the server

.. code-block:: bash

   sudo systemctl restart openvpn-server@server.service


Recommendations:

- Always set ``auth-nocache`` in the client to avoid storing credentials in memory.  
- Remove ``duplicate-cn`` if you want to restrict users to a single active session.  
- Keep OpenVPN and PAM modules up to date.  

Jump host connection tweaks
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once OpenVPN is configured is important to fix the networking configuration.

It may be necessary to configure Linux IP forwarding (have a look to `the following link <https://linuxconfig.org/how-to-turn-on-off-ip-forwarding-in-linux>`_).

Indeed iptables is not well configured:

::

  # run this command to optain the ip table
  sudo iptables -t nat -L --line-numbers

  # expected output:

  Chain PREROUTING (policy ACCEPT)
  num  target     prot opt source               destination         
  
  Chain INPUT (policy ACCEPT)
  num  target     prot opt source               destination         
  
  Chain OUTPUT (policy ACCEPT)
  num  target     prot opt source               destination         
  
  Chain POSTROUTING (policy ACCEPT)
  num  target     prot opt source               destination         
  1    SNAT       all  --  10.8.0.0/24         !10.8.0.0/24          to:<JUMP_HOST_PUBLIC_IP>


You need to tell to iptables that the network source is the openvpn network (here 10.8.0.0/24), the destination need to be the private network (here 172.18.7.0/24) doing NAT, so we add the masquarade:

::

  iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -d 172.18.7.0/24 -o ens4 -j MASQUERADE

Then we have to remove the SNAT line:

::

  iptables -t nat -D POSTROUTING 1

To make the change permanent, disable and stop the default OpenVPN iptables service:

::

  systemctl stop openvpn-iptables.service

  systemctl disable openvpn-iptables.service
