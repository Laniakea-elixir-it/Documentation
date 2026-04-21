Create a IAM client or a protected resource
-------------------------------------------

#. Login as administrator or registered user.

   .. figure:: _static/iam/iam_dashboard.png
      :scale: 40%
      :align: center

#. Click on **MitreID Dashboard** and then **Self-service client registration** for client creation or **Self-service protected resource registration** to register a new protected resource.

   .. figure:: _static/iam/mitre.png
      :scale: 40%
      :align: center

#. Click on *New client* and provides at least the the following parameters:

   ::

     Client name = iam-client-name

     redirect URI(s) = http(s)://<service_url>

   .. warning::

      The redirect URI(s) is required only for client creation.

   .. figure:: _static/iam/client_main.png
      :scale: 40%
      :align: center

#. In the ``Access`` tab configure your client as requested by your service, for example:

   ::

     Scopes: openid, profile, email, address, phone, offline_access
       
     Grant Types: authorization code, refresh

   
   .. figure:: _static/iam/client_access.png
      :scale: 40%
      :align: center

#. Save the client.

   
   .. figure:: _static/iam/client_saved.png
      :scale: 40%
      :align: center

#. Save **Client ID**, **Client Secret** and **Registration Access Token** or the full output json in the **JSON** tab for future access.

   
   .. figure:: _static/iam/client_json.png
      :scale: 40%
      :align: center

#. If you need Token Introspection and/or Token exchange, login as Administrator user, and through the **ADMINISTRATIVE**, **Manage Clients**, in the ``Access`` tab flag the needed options.

   .. figure:: _static/iam/client_admin_access.png
      :scale: 40%
      :align: center
