Deployment beyond VPN
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
   If you are manually configuring the VM to act as a bastion host, ensure that the instance meets all the specified requirements. However, if you are using the Terraform integration, these manual steps are not necessary as the configuration is handled automatically.

   The specific version of the PAM module used is:
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
       device_endpoint:   "https://.../devicecode"
       token_endpoint:    "https://.../token"
       userinfo_endpoint: "https://.../userinfo"
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
   terraform apply

Configuration and management of identity and access policy
----------------------------------------------------------

Admins are responsible for ensuring that users are correctly mapped to their respective groups and roles before granting access to computing resources. The management of user identities and access rights is critical to maintaining the security and integrity of the infrastructure. This section outlines the primary Authentication and Authorization Infrastructures (AAI) integrated into any **tenant** or **project** a user is part of. Here are described:

#. **EGI Check-in**: The main proxy service for EGI resources.
#. **Life Science AAI (LS AAI):** The authentication infrastructure dedicated to the Life Science community
#. **INDIGO IAM:** The Identity and Access Management service (e.g., RECAS instance) for fine-grained authorization.

.. warning::
   Always verify the user's identity and their specific resource requirements via official communication channels before proceeding.
   Before granting, modifying or revoking access for any user within any system you must coordinate directly with the specific user requesting access.

EGI Check-in
~~~~~~~~~~~~

The EGI Check-in service is used for federated authentication. Admins must ensure the appropriate Virtual Organization (VO) memberships are verified.

.. warning::
   Human Ineraction needed: User groups and access permissions cannot be managed through the dashboard, you must coordinate with the EGI Check-in Team via email to finalize configuration. Refer to their `official protocol <https://www.egi.eu/service/check-in/>`_.

EGI Check-in client
^^^^^^^^^^^^^^^^^^^

To use EGI Check-in on your bastion host, you must register a client to obtain a **Client ID** and **Client Secret**.

* Visit the `EGI Federation Registry <https://aai.egi.eu/federation/egi/home>`_ to start the process.
* These credentials must be inserted into the PAM module configuration file on your bastion host.

LS AAI
~~~~~~

Specifically used for biological and medical research projects. Access policies here are often driven by project-specific attributes.

.. warning::
   Human Interaction needed: Managing user groups and accessibility requires direct communication with the LS AAI Team, please follow the protocol outlined in the `LS AAI Site <https://lifescience-ri.eu/ls-login/ls-aai-aup.html>`_.
   For more informations contact their team at: ``support@aai.lifescience-ri.eu`` (Always verify details through their official channels for the latest updates).

LS AAI assign new groups
^^^^^^^^^^^^^^^^^^^^^^^^

A brief look to the terminology used in this section:

#. **Facility**: Represents the physical or logical entity providing the service (e.g., a compute cluster or database).
#. **Resource**: An instance of a Facility assigned to a specific project or VO. It acts as the bridge between the service and the users.

.. note::
   This guide outlines the workflow for service providers who are not yet Virtual Organization (VO) Managers. We will cover the lifecycle from registering a new Facility to the final assignment of user groups. If you are already an admin of one facility you can skip directly to writing to the support for group assignment.

To register a facility visit: `LS AAI Service registration <https://services.aai.lifescience-ri.eu/spreg/>`_, an homepage like this will appear:

.. figure:: _static/request_facility_homepage.png
   :scale: 40%
   :align: center

Select ``NEW SERVICE`` from the panel on the right:

.. figure:: _static/creating_facility_service.png
   :scale: 40%
   :align: center

After selecting your preferred protocol (OIDC or SAML 2.0), complete the registration form:

.. figure:: _static/compile_this_for_facility.png
   :scale: 40%
   :align: center

.. note::
   Ensure you have your technical metadata and administrative contact details ready to complete these fields.

Once registered, your service will appear in the `LS AAI site <https://perun.aai.lifescience-ri.eu/facilities>`_ under the ``name`` that you have chosen previously. Here a look to an example:

.. figure:: _static/facility_created.png
   :scale: 40%
   :align: center

Clicking on your facility will redirect you to the ``Facility Management Center``:

.. figure:: _static/facilities_management.png
   :scale: 40%
   :align: center

To allow a Virtual Organization to use your facility, you must create a Resource. This defines the specific ``Project`` environment.

.. figure:: _static/resource_creating.png
   :scale: 40%
   :align: center

Once submitted:

.. figure:: _static/resource_created.png
   :scale: 40%
   :align: center

To finalize the workflow and connect specific user groups to your resource, you must contact the LS AAI support team via email: ``support@aai.lifescience-ri.eu``.

.. warning::
   To manage existing groups or modify user access as a Group Manager, please refer to the official: `HERE <https://perunaai.atlassian.net/wiki/spaces/PERUN/pages/123600928/Group+manager>`_ 

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

.. _connection_keycloak_configuration:

Configuring eduperson entitlements on Keycloak
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To simulate an environment like LS AAI or EGI Check-in, your Keycloak instance must be able to issue specific entitlements. In the Life Science context, these are typically formatted as URNs (``urn:mace:usegalaxy.it:group:vo.usegalaxy.it:role=member#usegalaxy.it``). 
In this section we take a look on how to implement eduperson entitlement inside your Keycloak.

.. note::
   While there are several ways to manage entitlements, this guide focuses on the manual User attribute method.

First, navigate to the **Users** section in the Keycloak admin console and select the user you wish to configure.

1. Go to the **Attributes** tab.
2. Enter the following key-value pair:
   * **Key**: ``eduperson_entitlement`` (mandatory)
   * **Value**: ``urn:mace:usegalaxy.it:group:vo.usegalaxy.it:role=member#usegalaxy.it`` (**Use your specific project URN**).
3. Click **Save**.

.. figure:: _static/keycloak_user_attr.png
   :scale: 40%
   :align: center

To ensure the entitlement is delivered correctly in the identity token, you must configure a Mapper.

#. Navigate to Client Scopes and select the eduperson_entitlement scope.
#. Go to the Mappers tab and click on your mapper configuration.
#. Ensure that Multivalued is set to ON.

.. figure:: _static/keycloak_eduperson.png
   :scale: 40%
   :align: center

.. warning::
   Enabling Multivalued is critical, this ensures that the eduperson_entitlement arrives as a JSON array matching the standard format.

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

