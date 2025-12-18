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

The VPN is based on OpenVPN, with clients and server are configured to use TCP protocol.

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
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The PAM module enables OAuth2 device authentication for OpenVPN.  
We will install version ``0.0.3`` on **Ubuntu 22.04**.

.. note::

   Original installation instructions are available here: `click here <https://github.com/maricaantonacci/pam_oauth2_device#readme>`_

Please note:

1. The steps below are adapted and tested on **Ubuntu 22.04**.
2. Use the release `0.0.3` of the PAM module:  
   https://github.com/maricaantonacci/pam_oauth2_device/releases
3. When you create the IAM client, **do not leave the default “device code timeout” at 0 seconds**.  
   Set it explicitly to **300 seconds (5 minutes)**, otherwise it will expire immediately.

You can follow these steps or refer to the original istructions (remember to apply the suggested modification):

1. Download the repository:

   .. code-block:: bash

      wget https://github.com/maricaantonacci/pam_oauth2_device/archive/refs/tags/v0.0.3.tar.gz
      
      tar -xzf v0.0.3.tar.gz
      
      cd pam_oauth2_device-0.0.3

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

   You can install and configure OpenVPN using the following `script <https://raw.githubusercontent.com/Nyr/openvpn-install/master/openvpn-install.sh>`_.:  
  
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

PaaS Configuration
~~~~~~~~~~~~~~~~~~

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

  ssh -i /etc/im/.ssh/id_rsa im@<JUMP_HOST_PUBLIC_IP>

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
        "private_network_proxy_host": "<JUMP_HOST_PUBLIC_IP>",
        "iam_organisation": "ELIXIR-PAAS"
      },
      "type": "tenant"
    }
  }

Automatic deployment of a bastion on OpenStack
----------------------------------------------

In this sub-section, is shown how to automatically deploy a bastion host in an OpenStack environment using Terraform, followed by direct configuration through an Ansible role. The repository documenting all steps is available at this `GitHub repository link <https://github.com/Laniakea-elixir-it/ansible-role-vpn-bastion>`_.  

The Ansible role you'll find can be used standalone or as part of an automated deployment pipeline together with the Terraform module in the ``terraform directory``, which handles the infrastructure provisioning on OpenStack.

.. tip::
   Choose this installation method only if you already have some knowledge of Terraform and Ansible.
   Otherwise, follow the full guide.

Start by cloning the repository on any VM you want:

.. code-block:: bash
 
   git clone https://github.com/Laniakea-elixir-it/ansible-role-vpn-bastion

.. note::
   If you plan to use the full automated workflow (Terraform + Ansible), you can skip this
   Ansible-only section and jump directly to the **Terraform** part.  
   If instead you want to configure an existing VM manually or test the PAM/OIDC setup,
   run this Ansible role as described below.

Ansible configuration
~~~~~~~~~~~~~~~~~~~~~

This playbook turns an **Ubuntu 22.04** VM into a **bastion** that accepts SSH logins via **OpenID Connect (device code flow)** using the ``pam_oauth2_device`` module.

.. note::
   If you're configuring the VM where you're running as your bastion host make sure that the vm match the requirements, if you'll integrate the terraform part also, this is not necessary. 
   A modified version of the pam module is used. See:
   `pam_oauth2_device <https://github.com/Laniakea-elixir-it/pam_oauth2_device>`_.

The playbook performs the following tasks:

#. Builds and installs the ``pam_oauth2_device`` PAM module.
#. Writes ``/etc/pam_oauth2_device/config.json`` for your chosen IdP.
#. Sends the device-code URL via SMTP (disabled by default).
#. Creates ``~/.ssh/authorized_keys`` for the ``im`` user if a public key is provided.

By cloning the repository, the Ansible section contains:

.. code-block:: bash

   ├─ inventory
   ├─ site.yml
   ├─ group_vars/
   |  ├─ bastion.yml          # public settings (non-secret)
   |  └─ bastion.vault.yml    # secrets (managed through Ansible Vault)
   ├─ templates/
   |  └─ pam_config.json.j2
   └─ .gitignore

Inside the directory you will find:

- an ``inventory`` file  
- a ``site.yml`` for installation and PAM configuration  
- a ``group_vars`` directory containing settings and secrets  
- templates used to configure your IdP  

.. note::
   Make sure the following requirements are met:

   #. **Target host (can be your vm):** Ubuntu 22.04 VM reachable via SSH (with sudo-capable user, e.g. ``ubuntu``).
   #. **Controller:** Ansible ≥ 2.15.
   #. **OIDC client:** ``client_id`` and ``client_secret`` registered at your IdP.
   #. (Optional) SMTP credentials for delivering device-code URLs by email.

Steps to follow
~~~~~~~~~~~~~~~

First edit the ``inventory`` file and set your bastion’s public IP and SSH user:

.. code-block:: bash

   [bastion]
   bastion1 ansible_host=BASTION_PUBLIC_IP ansible_user=ubuntu

Then choose your IdP and fill the provider endpoints. Open ``group_vars/bastion.yml``. IAM endpoints are already filled in; for other IdPs, replace the placeholders:

.. code-block:: yaml

   idp_provider: "iam"   # or lifescience | egi

    ...

   oidc_providers:
     iam:
       device_endpoint:   "https://iam.recas.ba.infn.it/devicecode"
       token_endpoint:    "https://iam.recas.ba.infn.it/token"
       userinfo_endpoint: "https://iam.recas.ba.infn.it/userinfo"
     lifescience:
       device_endpoint:   "FILL_ME"
       token_endpoint:    "FILL_ME"
       userinfo_endpoint: "FILL_ME"
     egi:
       device_endpoint:   "FILL_ME"
       token_endpoint:    "FILL_ME"
       userinfo_endpoint: "FILL_ME"

Then is important to modify the ``bastion.vault.yml`` and insert your sensible data.

.. warning::
   Put **secrets** into the Vault file.  
   **Never** commit the Vault file.

.. code-block:: bash 

   # OIDC client (confidential)
   client_id: "YOUR_OIDC_CLIENT_ID"
   client_secret: "YOUR_OIDC_CLIENT_SECRET"

   # SMTP password (only if enable_email: true in bastion.yml)
   smtp:
     smtp_password: "YOUR_SMTP_PASSWORD"

   # Create local UNIX users before enabling PAM (must include the OIDC preferred_username)
   preferred_username: "your_oidc_username"
   extra_local_users:
     - "im"            # technical jump user (optional)
     # - "anotheruser" # add more if needed

   # SSH public key for the 'im' user (optional)
   jump_user_pubkey: "ssh-rsa AAAA... comment"

   Once configured, run the playbook:

**(Optional)** enable email for device code/URL:

.. code-block:: bash

   enable_email: true
   smtp:
     smtp_server_url: "smtps://smtp.gmail.com:465"
     smtp_username: "your-smtp-user"
     # smtp_password goes in bastion.vault.yml

Then encrypt your valut and run the playbook:

.. code-block:: bash

   ansible-playbook -i inventory site.yml

Now you should have a fully functional and configured bastion host on your ``BASTION_PUBLIC_IP``. 

Create the Bastion Host with Terraform
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This procedure defines and deploys a Virtual Machine (VM) on OpenStack, configured to act as a Bastion Host (jump host). It serves as a secure SSH entry point to access resources in private networks.

Running the configuration will create:

#. An OpenStack keypair for SSH access.  
#. A bastion VM in OpenStack with both **public and private** NICs.  
#. An Ansible inventory file pointing to the VM with the correct SSH key and IP.  
#. A fully configured bastion host (via Ansible).  

.. note::
   Ensure the following requirements:

   - **Terraform:** ≥ 1.14.0  
   - **Ansible:** ≥ 2.15  
   - **OIDC client:** ``client_id`` + ``client_secret`` from your IdP  
   - (Optional) SMTP credentials  

Structure of the repository
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cloning the repository gives the following Terraform structure:

.. code-block:: bash

   terraform_bastion/
       ├─ main.tf
       ├─ terraform.tfvars
       └─ variables.tf

- ``main.tf`` contains the configuration and all required fields.  
- ``variables.tf`` defines and documents all variables.  
- ``terraform.tfvars`` contains sensitive values and must be kept private.

.. warning::
   If you fork the repository, **never** commit ``terraform.tfvars``.  
   Add it to ``.gitignore`` or encrypt and use a vault.

Steps to follow
~~~~~~~~~~~~~~~

The ``main.tf`` and ``variables.tf`` files are already setted, you need to modify the ``terraform.tfvars`` with your sensible configurations:

.. code-block:: bash 
   auth_url      = "AUTHENTICATOR-URL"
   user_name     = "NAME-OF-THE-USER"
   password      = "SUPER-SECRET-PASSWORD"
   tenant_name   = "TENANT OR PROJECT NAME"
   region        = "RegionOne"

   public_network = "public"
   flavor         = "DESIRED FLAVOUR"
   image          = "Ubuntu 22.04"

When you set all the configuration run the commant for terraform, and it will create and configure the bastion host for you:

.. code-block:: bash
   terraform init
   terrafrom apply

Configuration and management of identity and access policy
----------------------------------------------------------

mmmm

EGI
~~~

mmmm

LS AAI
~~~~~~

mmmm

IAM RECAS
~~~~~~~~~

mmm

Authentication & Entitlements
-----------------------------

Identity Providers (IdPs) expose user authorization data in different ways, some IdPs such as ReCaS IAM or AWS Cognito-embed group membership directly inside the access token as simple JSON attributes, for example:

.. code-block:: json

   {
     "sub": "1234567890",
     "name": "Mario Rossi",
     "group": "tester"
   }

Other federated AAI providers, such as the Life Science Authentication and Authorization Infrastructure (LS AAI) and the European Grid Infrastructure (EGI), use a more structured mechanism based on the ``eduPersonEntitlement`` attribute, defined in the eduPerson schema.

EDUPERSON-ENTITLEMENT
~~~~~~~~~~~~~~~~~~~~~

In Research and Education federations, organizations exchange authorization information using standardized schemas such as **eduPerson**. The ``eduPersonEntitlement`` attribute expresses rights or memberships
assigned to a user. Extracting the group name is slightly more complex than reading a flat attribute because the information is encoded inside a URN.

Generic structure:

.. code-block:: text

   urn:<authority>:<domain>:group:<path>:<optional_role>#<qualifier>

Components:

- ``urn:`` Uniform Resource Name  
- ``authority:`` issuing authority  
- ``domain:`` authority domain  
- ``group:`` group identifier  
- ``path:`` subgroup or hierarchical path  
- ``role:`` role inside the group  
- ``qualifier:`` IdP name or scope  

This is not a location identifier; it is purely a structured authorization string.

We focus on EGI and LS AAI in this enviroment.

EGI
~~~

EGI group entitlements follow this schema:

.. code-block:: text

   urn:mace:egi.eu:group:<vo_name>:role=<role>#<aai-domain>

Main elements:

- ``urn:mace:egi.eu:`` official EGI namespace  
- ``<vo_name>:`` the VO or group name  
- ``role=vm_operator / role=member:`` the user's VO role  
- ``#aai.egi.eu:`` the authority qualifier  

In our script, we currently **ignore the role** and extract only the VO/group name. This is fine for our use case, since role-based access is not needed.

EGI Check-in exposes two types of entitlements:

**a) Resource entitlements (res):**

These refer to backend services:

.. code-block:: text

   urn:mace:egi.eu:res:ggus.eu
   urn:mace:egi.eu:res:gocdb#aai.egi.eu
   urn:mace:egi.eu:res:rcauth#aai.egi.eu

These do **not** represent groups and are ignored by our script, because they do not provide useful authorization information for our purposes.

**b) Group entitlements:**

These contain the actual group (VO) and role information. Only these are used.

LS AAI
~~~~~~

LS AAI expresses group-based authorization using this structure:

.. code-block:: text

   urn:geant:lifescience-ri.eu:group:lifescience:<subgroup/subdomain>:<service>#aai.lifescience-ri.eu

Unlike EGI, LS AAI does **not** encode roles inside the entitlement. All LS AAI group information is carried inside the ``<subgroup>`` or ``<subdomain>`` path and the ``<service>`` part.

IAM ReCaS
~~~~~~~~~

For IAM ReCaS, the situation is simpler: the access token itself contains a ``group`` field. You can inspect this directly by decoding the JWT with any JWT encoder no entitlement parsing is
required.

AWS
~~~

AWS Cognito typically embeds groups in ``cognito:groups`` inside the token. 

(TODO: complete this part once integration tests are finished.)

Pam OAUTH2 module modification
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Thanks to additional features added to the original ``pam_oauth2_device`` module implemented in `this repository <https://github.com/Laniakea-elixir-it/pam_oauth2_device>`_, it is possible to extract group membership from all IdPs described above, including those using ``eduPersonEntitlement`` (EGI and LS AAI) and those adopting groups directly in the token (IAM ReCaS, AWS Cognito).

The modified module can:

- detect whether the IdP exposes groups through entitlements or token claims,
- normalize the extracted group list,  
- return a unified group representation to PAM, making the authentication flow
  consistent regardless of the IdP.

References
----------

Install OpenVPN: https://community.openvpn.net/openvpn/wiki/OpenvpnSoftwareRepos  
Enable IP forwarding: https://linuxconfig.org/how-to-turn-on-off-ip-forwarding-in-linux  
Enable IP forwarding with OpenVPN: https://openvpn.net/faq/what-is-and-how-do-i-enable-ip-forwarding-on-linux/  
Iptables configuration: https://askubuntu.com/questions/1181115/openvpn-client-cannot-access-any-network-except-for-the-server-itself-after-conn  
Example eduPersonEntitlement: https://help.switch.ch/aai/support/documents/attributes/edupersonentitlement/  
EduPerson entitlement values: https://servicedesk.surf.nl/wiki/spaces/IAM/pages/128910063/Standardized+values+for+eduPersonEntitlement

