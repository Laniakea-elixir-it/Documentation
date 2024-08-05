Vault configuration
===================

The Vault support can be enabled editing the ``/etc/orchestrator-dashboard/config.json`` file, inserting the Vault url:

::

  ...
  "VAULT_URL": "https://<vault_host>:<vault_port>"

Vault fine tuning can be done through the ``vault-config.json`` file at ``/etc/orchestrator-dashboard/vault-config.json``:

::

  {
    "VAULT_BOUND_AUDIENCE": "orchestrator-dashboard",
    "VAULT_SECRETS_PATH": "secrets",
    "WRAPPING_TOKEN_TIME_DURATION": "1h",
    "READ_POLICY": "read_only",
    "READ_TOKEN_TIME_DURATION": "12h",
    "READ_TOKEN_RENEWAL_TIME_DURATION": "12h",
    "WRITE_POLICY": "write_only",
    "WRITE_TOKEN_TIME_DURATION": "12h",
    "WRITE_TOKEN_RENEWAL_TIME_DURATION": "12h",
    "DELETE_POLICY": "delete_only",
    "DELETE_TOKEN_TIME_DURATION": "12h",
    "DELETE_TOKEN_RENEWAL_TIME_DURATION": "12h"
  }

Configuration options
---------------------

VAULT_BOUND_AUDIENCE
********************

``Description``: Vault is configured to exploits Json Web Token (JWT) for authentication. The role created on Vault (called ``laniakea``) authorizes **only** JWT with the given subject (i.e. user identifier) and this audience claim and gives it the policy. This parameter allows the dashboard to retrieve a token with the right bound audience to login on Vault.

``Default``: orchestrator-dashboard

VAULT_SECRETS_PATH
******************

``Description``: path on Vault where users secrets are stored.

``Default``: secrets/

WRAPPING_TOKEN_TIME_DURATION
***************************

``Description``: time duration of the wrapping token sent to the encryption script to upload secrets on Vault.

``Default``: 1h (1 hour)

READ_POLICY
***********

``Description``: Secrets reading policy name. This policy has to be configured on Vault with the right permissions to read secrets.

``Default``: read_only

READ_TOKEN_TIME_DURATION
************************

``Description``: time duration of the read token, to read secrets on vault

``Default``: 12h (12 hours)

READ_TOKEN_RENEWAL_TIME_DURATION
********************************

``Description``: renew time period of read token.

``Default``: 12h (12 hours)

WRITE_POLICY
************

``Description``: Secrets writing policy name: The correspondig policy has to be configured on Vault with the right permissions to write secrets.

``Default``: write_only

WRITE_TOKEN_TIME_DURATION
*************************

``Description``: time duration of the write token, to write secrets on vault

``Default``: 12h (12 hours)

WRITE_TOKEN_RENEWAL_TIME_DURATION
*********************************

``Description``: renew time period of write token.

``Default``: 12h (12 hours)

DELETE_POLICY
*************

``Description``: Secrets deletion policy name. This policy has to be configured on Vault with the right permissions to delete secrets.

``Default``: delete_only

DELETE_TOKEN_TIME_DURATION
**************************

``Description``: time duration of the delete token, to delete secrets on vault

``Default``: 12h (12 hours)

DELETE_TOKEN_RENEWAL_TIME_DURATION
**********************************

``Description``: renew time period of delete token.

``Default``: 12h (12 hours)
