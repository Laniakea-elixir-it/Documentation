Deployment under VPN
====================

The PaaS provides the possibility to instantiate and configure VMs with private network only and then configure them to be accessed through a VPN, therefore providing complete isolation to the environment.

Isolation is reached using OpenStack tenant and security groups properties, granting the access only through VPN authentication, while the user authentication to the VPN using the same Laniakea credentials.

.. figure:: _static/vpn/vpn_architecture.png
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

.. figure:: _static/vpn/vpn_auth_flow.png
   :scale: 40%
   :align: center

PAM module installation and configuration
-----------------------------------------

Original instructions provided here: https://github.com/maricaantonacci/pam_oauth2_device#readme

Device code timeout is default is 0. It should be a different value: we set 300.

OpenVPN installation
--------------------

Jump host connection tweaks
===========================

https://askubuntu.com/questions/1181115/openvpn-client-cannot-access-any-network-except-for-the-server-itself-after-conn

aggiungo il masquarade

iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -d 172.18.7.0/24 -o ens4 -j MASQUERADE


rimuovo SNAT

iptables -t nat -D POSTROUTING 1

mettere ip_forwarding 1

PaaS Configuration
------------------
