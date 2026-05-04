Deployment behind VPN
=====================

The PaaS provides the possibility to instantiate and configure VMs with private network only and then configure them to be accessed through a VPN, therefore providing complete isolation to the environment.

Isolation is reached using OpenStack tenant and security groups properties, granting the access only through VPN authentication, while the user authentication to the VPN using the same Laniakea credentials.

.. figure:: _static/vpn_architecture.png
   :scale: 40%
   :align: center

We use a jump host VM that has two fundamental functions: 

1. Allow IM to access the private network and perform VM installation and configuration;
2. Allow users to access the private network and the deployment. Therefore, we need to configure it to act as access point to the IM and to the users.

.. note::

   The procedure has been tested using Ubuntu 22.04 as OS on the jump host VM.

The VPN relies on OpenVPN, with both client and server configured to use the TCP protocol.

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

Service configuration
---------------------

.. toctree::
   :maxdepth: 2

   ansible_install
   manual_install
   laniakea_integration
   auth_intro
   ls_aai
   egi_checkin
   keycloak

References
----------

Install OpenVPN: https://community.openvpn.net/openvpn/wiki/OpenvpnSoftwareRepos  

Enable IP forwarding: https://linuxconfig.org/how-to-turn-on-off-ip-forwarding-in-linux  

Enable IP forwarding with OpenVPN: https://openvpn.net/faq/what-is-and-how-do-i-enable-ip-forwarding-on-linux/  

Iptables configuration: https://askubuntu.com/questions/1181115/openvpn-client-cannot-access-any-network-except-for-the-server-itself-after-conn 

Example eduPersonEntitlement: https://help.switch.ch/aai/support/documents/attributes/edupersonentitlement/  

EduPerson entitlement values: https://servicedesk.surf.nl/wiki/spaces/IAM/pages/128910063/Standardized+values+for+eduPersonEntitlement

