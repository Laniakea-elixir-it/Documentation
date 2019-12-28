Basic configuration
===================

The dashboard configuration file is located at ``/etc/orchestrator-dashboard/config.json``, to make configuration changes.

::

  {
      "IAM_CLIENT_ID": "my_client_id",
      "IAM_CLIENT_SECRET": "my_client_secret",
      "IAM_BASE_URL": "https://iam-test.indigo-datacloud.eu",
      "ORCHESTRATOR_URL": "https://indigo-paas.cloud.ba.infn.it/orchestrator",
      "SLAM_URL": "https://indigo-slam.cloud.ba.infn.it:8443",
      "CMDB_URL": "https://indigo-paas.cloud.ba.infn.it/cmdb",
      "IM_URL": "https://indigo-paas.cloud.ba.infn.it/im",
      "TOSCA_TEMPLATES_DIR": "/opt/tosca-templates",
      "TOSCA_PARAMETERS_DIR": "/opt/tosca-parameters",
      "TOSCA_METADATA_DIR": "/opt/tosca-metadata",
      "CALLBACK_URL": "https://my-orchestrator-dashboard.com/callback",
      "DB_HOST": "localhost",
      "DB_USER": "my-user",
      "DB_PASSWORD": "my-password",
      "DB_NAME": "orchestrator_dashboard",
      "DB_PORT": "3306",
      "MAIL_SERVER": "your.smtp.server.com",
      "MAIL_PORT": "25",
      "MAIL_SENDER": "test@orchestrator.com",
      "ADMINS": "['admin@foo.it','other_admin@test.it']",
      "VAULT_URL": "https://my-vault-instance.com",
      "SUPPORT_EMAIL": "support@example.com",
      "EXTERNAL_LINKS": [ { "url": "https://indigo-paas.cloud.ba.infn.it/status-page", "menu_item_name": "Services status" } ],
      "ENABLE_ADVANCED_MENU": "no",
      "LOG_LEVEL": "info
  }

Configuration options
---------------------

IAM_CLIENT_ID
*************

``Description``: IAM client ID for the dashboard.

IAM_CLIENT_SECRET
*****************

``Description``: IAM client Secret for the dashaboard.

IAM_BASE_URL
************

``Description``: IAM url.


ORCHESTRATOR_URL
*****************

``Description``: Orchestrator url.


SLAM_URL
********

``Description``: SLAM url.

CMDB_URL
********

``Description``: CMDB url.

IM_URL
******

``Description``: IM url.

TOSCA_TEMPLATES_DIR
*******************

``Description``: Path of TOSCA tempaltes to be loaded.

``Defaults``: /opt/laniakea-dashboard-config/tosca-templates",

TOSCA_PARAMETERS_DIR
********************

``Description``: Path of TOSCA template parameters to create Dashboard configurable forms.

``Defaults``: /opt/laniakea-dashboard-config/tosca-parameters

TOSCA_METADATA_DIR
******************

``Description``: Path of TOSCA template metadata with additional info (e.g. icon path).

``Defaults``: /opt/laniakea-dashboard-config/tosca-metadata

CALLBACK_URL
************

``Description``: Dahsboard url for callback. Configure it as ``<dashboard url>/callback``

``Defaults``: https://my-orchestrator-dashboard.com/callback

DB_HOST
*******

``Description``: Dataase host. Configure it with the IP address of the Database host (do not leave ``localhost``).

``Defaults``: localhost

DB_USER
*******

``Description``: MySQL database user.

``Defaults``: orchestrator

DB_PASSWORD
***********

``Description``: MySQL database password.

DB_NAME
*******

``Description``: MySQL database name:

``Defaults``: orchestrator_dashboard

DB_PORT
*******

``Description``: MySQL database port.

``Defaults``: 3306

MAIL_SERVER
***********

``Description``: Mail server address allowing Dahsboard notifications.


MAIL_PORT
*********

``Description``: Mail server port.

``Defaults``: 25

MAIL_SENDER
***********

``Description``: Mail sender of the notification mail.

``Defaults``: Laniakea@elixir-italy.org

ADMINS
******

``Description``: Dahsobard administrator users. Set this to a comma-separated list of valid Galaxy users (email addresses). These users will have access to the ``Users`` section of the dashboard.

VAULT_URL
*********

``Description``: Vault url. This option enable vault support on Laniakea.

SUPPORT_EMAIL
*************

``Description``: Support email, displayed on 500 error page.

``Defaults``: laniakea.helpdesk@gmail.com

EXTERNAL_LINKS
**************

``Description``: create menu with external links, giving the url and the menu item name.

ENABLE_ADVANCED_MENU
********************

``Description``: if yes, show advanced options in the navbar and the configurator form.

LOG_LEVEL
*********

``Description``: Set log level.

``Defaults``: info
