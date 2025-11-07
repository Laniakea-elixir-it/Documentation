Deployment under VPN
====================

The PaaS provides the possibility to instantiate and configure VMs with private network only and then configure them to be accessed through a VPN, therefore providing complete isolation to the environment.

Isolation is reached using OpenStack tenant and security groups properties, granting the access only through VPN authentication, while the user authentication to the VPN using the same Laniakea credentials.

.. figure:: _static/vpn_architecture.png
   :scale: 40%
   :align: center

We use a jump host VM that has two fundamental functions: (1) allow IM to access the private network and perform VM installation and configuration; (2) allow users to access the private network and the deployment. Therefore, we need to configure it to act as access point to the IM and to the users.

.. note::

   The procedure has been tested using Ubuntu 22.04 as OS on the jump host VM.

The VPN is based on OpenVPN, with clients and server are configured to use TPC protocol.

We exploit a PAM plugin to enable authentication through OpenID Connect, exploiting Oauth2 device flow:

#. the user connects to the VPN server using an OpenVPN client;

#. PAM is configured to send verification code by e-mail to the user;

#. the user can authenticate with its own Laniakea credentials;

#. the OIDC provider (INDIGO-IAM) sends the access token to the VPN server, that is now able to verify users identity and authorizations;

#. if the user owns the right tenant permissions, he is granted access to the private network and can finally interact with the deployed application.

.. figure:: _static/vpn_auth_flow.png
   :scale: 40%
   :align: center

Let's get started with the installation.

VM configuration
----------------

Create a virtual machine (VM) that will be used as **jump host** in the tenant where you are planning to enable VPN deployment. 

.. note::
   Each tenant in your cloud requires its own dedicated jump host.

The VM should meet the following minimum requirements:

======= ==============================
OS      Ubuntu 22.04
vCPUs   2
RAM     4 GB
Network Public and private IP address
======= ==============================

PAM module installation and configuration
-----------------------------------------

The PAM module enables OAuth2 device authentication for OpenVPN.  
We will install version ``0.0.3`` on **Ubuntu 22.04**.

.. note::

   Original installation instructions are available here:
   https://github.com/maricaantonacci/pam_oauth2_device#readme

Please note:

1. The steps below are adapted and tested on **Ubuntu 22.04**.
2. Use the release `0.0.3` of the PAM module:  
   https://github.com/maricaantonacci/pam_oauth2_device/releases
3. When you create the IAM client, **do not leave the default “device code timeout” at 0 seconds**.  
   Set it explicitly to **300 seconds (5 minutes)**, otherwise it will expire immediately.

You can follow these steps or refer to the original one(remember to apply the suggested modification):

1. Download the repository

   .. code-block:: bash

      wget https://github.com/maricaantonacci/pam_oauth2_device/archive/refs/tags/v0.0.3.tar.gz
      tar -xzf v0.0.3.tar.gz
      cd pam_oauth2_device-0.0.3

2. Install required tools

   .. code-block:: bash

      sudo apt update
      sudo apt install -y build-essential libcurl4-openssl-dev libpam0g-dev

3. Build and install the module

   .. code-block:: bash

      make
      sudo make install

Remembrer that when creating the OIDC client on your identity provider,  **do not** leave the default ``device code timeout`` at 0 seconds but set it to **300 seconds** 
(or any number > 0) to prevent immediate expiration.

OpenVPN installation
--------------------

.. note::

   OpenVPN **version >= 2.5** is required to enable the *deferred authentication* mechanism,
   which is needed by the PAM OAuth2 module.

1. The first step is add the OpenVPN repository:

   .. code-block:: bash

      wget -O - https://swupdate.openvpn.net/repos/repo-public.gpg | sudo apt-key add -
      echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/openvpn-repo-public.gpg] https://build.openvpn.net/debian/openvpn/release/2.5 jammy main" > /etc/apt/sources.list.d/openvpn-aptrepo.list
      sudo apt update

2. Then you need to install OpenVPN

   You can install and configure OpenVPN automatically using the following script:  
   https://raw.githubusercontent.com/Nyr/openvpn-install/master/openvpn-install.sh

   .. code-block:: bash

      wget https://raw.githubusercontent.com/Nyr/openvpn-install/master/openvpn-install.sh
      chmod +x openvpn-install.sh
      sudo ./openvpn-install.sh

   During the script execution, **select the following options when prompted**:

   - **Protocol:** TCP  
   - **Port:** 1194 (default)  
   - **Server name:** choose a name or press Enter to use the default one  

   Once the script completes, your OpenVPN server will be installed and ready for configuration.


Enable the PAM plugin
---------------------

Create the file ``/etc/pam.d/openvpn`` with your preferred editor:

.. code-block:: bash

   auth     required    pam_oauth2_device.so
   account  sufficient  pam_oauth2_device.so

Then edit ``/etc/openvpn/server/server.conf`` to include the public IP of the jump host and the private network IP.

.. note::

   Lines marked below are critical for correct PAM + OAuth2 integration,  
   especially when using username/password authentication only (no client certificates).

.. code-block:: bash

   local <PUBLIC IP OF THE JUMP HOST>
   port 1194
   proto tcp
   dev tun

   # Base PKI configuration
   ca ca.crt
   cert server.crt
   key server.key
   dh dh.pem

   # TLS and encryption settings
   auth SHA512
   cipher AES-256-CBC
   tls-crypt tc.key
   topology subnet
   server 10.8.0.0 255.255.255.0

   # Network and routing
   #push "redirect-gateway def1 bypass-dhcp"
   push "route <PRIVATE NETWORK> 255.255.255.0"
   push "dhcp-option DNS 8.8.8.8"
   push "dhcp-option DNS 8.8.4.4"

   # Reliability and permissions
   keepalive 10 120
   user nobody
   group nogroup
   persist-key
   persist-tun
   verb 7
   crl-verify crl.pem

   # PAM integration
   plugin /usr/lib/x86_64-linux-gnu/openvpn/plugins/openvpn-plugin-auth-pam.so openvpn
   duplicate-cn
   setenv deferred_auth_pam 1
   reneg-sec 0
   hand-window 300
   username-as-common-name

Particular attention over key parameters:

1. ``duplicate-cn``: Allow multiple clients with the same common name to concurrently connect. In the absence of this option, OpenVPN will disconnect a client instance upon connection of a new client having the same common name.**Remove it if you want to enforce single-session per user.**

...

2. Client configuration


Update the ``client.ovpn`` file to use the public IP of the jump host and include the required authentication parameters.

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
   ...
   </ca>

3. Applying the configuration

.. code-block:: bash

   # Open port 1194 (if using UFW)
   sudo ufw allow 1194/tcp

   # Restart OpenVPN service
   sudo systemctl restart openvpn-server@server.service

   # Follow logs for debugging
   sudo journalctl -u openvpn-server@server.service -f

   # Check plugin path and OpenVPN version
   openvpn --version | head -n 5
   ls -l /usr/lib/x86_64-linux-gnu/openvpn/plugins/openvpn-plugin-auth-pam.so

4. Troubleshooting

- **Handshake fails or client certificate requested:**  
  Add ``verify-client-cert none`` (or ``client-cert-not-required`` for OpenVPN 2.4).

- **Authentication succeeds but connection drops during rekey:**  
  Ensure ``reneg-sec 0`` is set.

- **PAM module not triggered:**  
  The PAM service name in ``plugin ... openvpn`` must match the file in ``/etc/pam.d/``.

- **“plugin not found” error:**  
  Confirm plugin path with ``find /usr -name openvpn-plugin-auth-pam.so``.

- **Timeouts during authentication:**  
  Increase ``hand-window`` to 300 and ensure the IAM *device code timeout* is set to **300 seconds**.

5. Security recommendations

- Always set ``auth-nocache`` in the client to avoid storing credentials in memory.  
- Remove ``duplicate-cn`` if you want to restrict users to a single active session.  
- Keep OpenVPN and PAM modules up to date.  
- Limit SSH and management access to the jump host.

Jump host connection tweaks
---------------------------

Once the OpenVPN is configured you need to fix the networking configuration.

It may be necessary to configure Linux IP forwarding (see `here <https://linuxconfig.org/how-to-turn-on-off-ip-forwarding-in-linux>`_).

Indeed iptables is not well configured:

::

  # sudo iptables -t nat -L --line-numbers
  Chain PREROUTING (policy ACCEPT)
  num  target     prot opt source               destination         
  
  Chain INPUT (policy ACCEPT)
  num  target     prot opt source               destination         
  
  Chain OUTPUT (policy ACCEPT)
  num  target     prot opt source               destination         
  
  Chain POSTROUTING (policy ACCEPT)
  num  target     prot opt source               destination         
  1    SNAT       all  --  10.8.0.0/24         !10.8.0.0/24          to:212.189.202.200


We need to tell to iptables that the network source is the openvpn network (here 10.8.0.0/24), the destination the private network (here 172.18.7.0/24) doing NAT, so we add the masquarade:

::

  iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -d 172.18.7.0/24 -o ens4 -j MASQUERADE

Then we have to remove the SNAT line

::

  iptables -t nat -D POSTROUTING 1

To make this permanent disable and turn off  the openvpn iptables systemd file ``/etc/systemd/system/openvpn-iptables.service``

::

  systemctl stop openvpn-iptables.service

  systemctl disable openvpn-iptables.service

PaaS Configuration
------------------

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

  ssh -i /etc/im/.ssh/id_rsa im@212.189.202.200

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
        "private_network_proxy_host": "212.189.202.200",
        "iam_organisation": "ELIXIR-PAAS"
      },
      "type": "tenant"
    }
  }

References
----------

Install OpenVPN: https://community.openvpn.net/openvpn/wiki/OpenvpnSoftwareRepos

Enable IP forwarding: https://linuxconfig.org/how-to-turn-on-off-ip-forwarding-in-linux

Enable IP forwarding with OpenVPN: https://openvpn.net/faq/what-is-and-how-do-i-enable-ip-forwarding-on-linux/

Iptables configuration: https://askubuntu.com/questions/1181115/openvpn-client-cannot-access-any-network-except-for-the-server-itself-after-conn
