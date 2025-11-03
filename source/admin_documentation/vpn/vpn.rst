Deployment under VPN
====================

The PaaS provides the possibility to instantiate and configure VMs with private network only and then configure them to be accessed through a VPN, therefore providing complete isolation to the environment.

Isolation is reached using OpenStack tenant and security groups properties, granting the access only through VPN authentication, while the user authentication to the VPN using the same Laniakea credentials.

.. figure:: _static/vpn_architecture.png
   :scale: 40%
   :align: center

We use a jump host VM that has two fundamental functions: (1) allow IM to access the private network and perform VM installation and configuration; (2) allow users to access the private network and the deployment. Therefore, we need to configure it to act as access point to the IM and to the users.

.. note::

   The procedure has been tested only using Ubuntu 22.04 as OS on the jump host VM.

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

VM configuration
----------------

Create VM for the jump host, in the tenant where you want enable the VPN deployments. You need a jump host for each tenant.

The VM should meet the following minimum requirements:

======= ==============================
OS      Ubuntu 22.04
vCPUs   2
RAM     4 GB
Network Public and private IP address.
======= ==============================

PAM module installation and configuration
-----------------------------------------

Original instructions to install the PAM module are provided here: https://github.com/maricaantonacci/pam_oauth2_device#readme

Please note:

#. Use Ubuntu 22.04 instead.

#. use the release ``0.0.3`` of the PAM module: https://github.com/maricaantonacci/pam_oauth2_device/releases

#. When you create the IAM client, please **the device code timeout default is 0 secs. It should be a different value: set it 300 secs.**

OpenVPN installation
--------------------

.. note::

   OpenVPN version >= 2.5 is needed for the server in order to enable the deferred authentication mechanism.

The first step is install OpenVPN

::

  wget -O - https://swupdate.openvpn.net/repos/repo-public.gpg | sudo apt-key add -

  echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/openvpn-repo-public.gpg] https://build.openvpn.net/debian/openvpn/release/2.5 jammy main" > /etc/apt/sources.list.d/openvpn-aptrepo.list

  sudo apt update

You can install OpenVPN with the `script <https://raw.githubusercontent.com/Nyr/openvpn-install/master/openvpn-install.sh>`_

.. note::

   Please select the following options: TCP for protocol

::

  wget https://raw.githubusercontent.com/Nyr/openvpn-install/master/openvpn-install.sh

  chmod +x openvpn-install.sh

  ./openvpn-install.sh

Enable the PAM plugin
---------------------

Create the file ``/etc/pam.d/openvpn`` with your favourit editor:

::

  auth required pam_oauth2_device.so
  account sufficient pam_oauth2_device.so

Then edit /etc/openvpn/server/server.conf adding the Public ip of the jump host and the private network IP.

.. note::

   Moreover, lines 26-31 are needed to be properly configured for the pam oauth2 module.

::

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

In particular:

#. ``duplicate-cn``: Allow multiple clients with the same common name to concurrently connect. In the absence of this option, OpenVPN will disconnect a client instance upon connection of a new client having the same common name.

#. ``setenv deferred_auth_pam 1``: enable deferred auth method.

#. ``reneg-sec 0``: avoid the end user to be challenged to reauthorize once per hour (default value).

#. ``hand-window 300``: set the handshake window to a larger value (default is 60s) to copy with email delays.

#. ``username-as-common-name``: use the authenticated username as the common name, rather than the common name from the client cert (neededas we are using the auth-user-pass on the client side).

Edit the client.ovpn file, with public IP of the jump ost and adding the needed options (lines 15-17):

::

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
  reneg-sec 0
  hand-window 300
  <ca>
  ...
  </ca>

Finally, restart the server:

::

  systemctl restart openvpn-server@server.service


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
