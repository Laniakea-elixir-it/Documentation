Obtaining an IAM access token
=============================

Prerequisites
-------------

#. Create a IAM client. The Redirect URI is not important, so you can exploit the IAM address itself. 

   .. figure:: _static/get_iam_token/get_iam_token_client_main.png
      :scale: 40%
      :align: center

#. Give the client the rigth **Scopes** and **Grant Types** as in the figure:

   .. figure:: _static/get_iam_token/get_iam_token_client_access.png
      :scale: 40%
      :align: center

#. Save.

#. Save **Client ID**, **Client Secret** and **Registration Access Token** or the full output json in the **JSON** tab for future access.

#. Login as Administrator user and select from the left menu **Manage Clients**.

#. Select the client just created.

#. Navigate to the **Tokens** tab and set it as in the figure and save. In particular the **Device Code Timeout** should not be empty.

   .. figure:: _static/get_iam_token/get_iam_token_client_tokens.png
      :scale: 40%
      :align: center

#. On any linux distirbution, e.g. Ubuntu, Install ``jq``:

   ::

     # apt-get install jq


#. Download the following script:

   ::

     wget https://raw.githubusercontent.com/Laniakea-elixir-it/Scripts/master/IAM/dc-get-access-token.sh

#. Give ``dc-get-access-token.sh`` execution permissions:

   ::

     chmod +x dc-get-access-token.sh

#. Create the file ``ìam.rc`` with the following content:

   ::

     IAM_DEVICE_CODE_CLIENT_ID="<get_iam_token_client_id>"
     IAM_DEVICE_CODE_CLIENT_SECRET="<get_iam_token_client_secret>"
     IAM_TOKEN_ENDPOINT="<iam_url>/token"
     IAM_DEVICE_CODE_ENDPOINT="<iam_url>/devicecode"


Get IAM access token
--------------------

#. Run ``dc-get-access-token.sh`` script

   ::

     # ./dc-get-access-token.sh

   .. figure:: _static/get_iam_token/get_iam_token_script_start.png
      :scale: 50%
      :align: center

#. Open in a browser the URL obtained from the script and paste code:

   .. figure:: _static/get_iam_token/get_iam_token_enter_code.png
      :scale: 50%
      :align: center

#. Authorize the client to create a token:

   .. figure:: _static/get_iam_token/get_iam_token_authorize.png
      :scale: 50%
      :align: center


#. Type ```Y`` on the shell script and get your access token:

   .. figure:: _static/get_iam_token/get_iam_token_script_end.png
      :scale: 50%
      :align: center

