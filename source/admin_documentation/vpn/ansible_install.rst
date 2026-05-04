Auomatic deployment of a jump host on OpenStack
-----------------------------------------------

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

Ansible configuration
~~~~~~~~~~~~~~~~~~~~~

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

Create the Jump Host with Terraform
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

Terraform configuration
~~~~~~~~~~~~~~~~~~~~~~~

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
