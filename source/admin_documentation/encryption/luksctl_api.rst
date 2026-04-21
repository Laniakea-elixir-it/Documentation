LUKSctl: APIs
=============

A set of RESTFul APIs is distributed with LUKSctl. It is written using python Flask micro framework and Gunicorn. It's source code is located on `Laniakea GitHub <https://github.com/Laniakea-elixir-it/luksctl_api>`_.

A systemd unit file is used for start/stop/restart the API.

=============  =========  ====================
Moudule        Action     Description
=============  =========  ====================
luksctl-api    status     Show status
|              stop       Stop the API
|              start      Start the API.
|              restart    Restart the API.
=============  =========  ====================


.. note::

   LUKSctl-api is configured to listen on ``5000`` port.

::

  $ sudo systemctl status luksctl-api
  ● luksctl-api.service - Gunicorn instance to serve luksctl api server
     Loaded: loaded (/etc/systemd/system/luksctl-api.service; enabled; vendor preset: disabled)
     Active: active (running) since Fri 2019-10-25 14:23:06 UTC; 1 day 17h ago
   Main PID: 19972 (gunicorn)
     CGroup: /system.slice/luksctl-api.service
             ├─19972 /home/luksctl_api/luksctl_api/venv/bin/python /home/luksctl_api/luksctl_api/venv/bin/gunicorn --workers 2...
             ├─19995 /home/luksctl_api/luksctl_api/venv/bin/python /home/luksctl_api/luksctl_api/venv/bin/gunicorn --workers 2...
             └─19997 /home/luksctl_api/luksctl_api/venv/bin/python /home/luksctl_api/luksctl_api/venv/bin/gunicorn --workers 2...
  
  Oct 25 14:23:06 slurmserver systemd[1]: Started Gunicorn instance to serve luksctl api server.
  Oct 25 14:23:07 slurmserver gunicorn[19972]: [2019-10-25 14:23:07 +0000] [19972] [INFO] Starting gunicorn 19.9.0
  Oct 25 14:23:07 slurmserver gunicorn[19972]: [2019-10-25 14:23:07 +0000] [19972] [INFO] Listening at: https://0.0.0.0:...19972)
  Oct 25 14:23:07 slurmserver gunicorn[19972]: [2019-10-25 14:23:07 +0000] [19972] [INFO] Using worker: sync
  Oct 25 14:23:07 slurmserver gunicorn[19972]: [2019-10-25 14:23:07 +0000] [19995] [INFO] Booting worker with pid: 19995
  Oct 25 14:23:07 slurmserver gunicorn[19972]: [2019-10-25 14:23:07 +0000] [19997] [INFO] Booting worker with pid: 19997
  Oct 26 07:55:37 slurmserver sudo[24629]: luksctl_api : TTY=unknown ; PWD=/home/luksctl_api/luksctl_api ; USER=root ; C...status
  Oct 27 07:48:04 slurmserver sudo[21947]: luksctl_api : TTY=unknown ; PWD=/home/luksctl_api/luksctl_api ; USER=root ; C...status
  Hint: Some lines were ellipsized, use -l to show in full.

It used to connect the Laniakea Dashboard to the encrypted instances, allowing end-user to perform some actions, e.g. to mount and enable the LUKS storage volume, without accessing the Virtual Machine with SSH.

Currently, supported APIs are:

-----------------
``Volume Status``
-----------------

A GET request is used to check the status of the encrypted volume and show it in the Dhasboard. If the volume is open and mounted it return ``mounted``, othrewise it return ``umounted``. If the API is not available, an ``unavailable`` status is showed.

Example request:

::

  $ curl -k -i -X GET 'https://90.147.75.173:5000/luksctl_api/v1.0/status'
  HTTP/1.1 200 OK
  Server: gunicorn/19.9.0
  Date: Sun, 27 Oct 2019 08:02:54 GMT
  Connection: close
  Content-Type: application/json
  Content-Length: 27
  
  {"volume_state":"mounted"}

-----------------
``Volume Open``
-----------------

A POST request can be used to open and mount the encrypted volume in case of VM reboot. To prevent unwanted restart, the API check if the volume is already mounted. If yes it return ``mounted``, otherwise it run ``luksctl open`` command.

Example request:

::

  curl -k -X POST 'https://<vm_ip_address>:5000/luksctl_api/v1.0/open' -H 'Content-Type: application/json' -d '{ "vault_url": vault_url, "vault_token": wrapping_read_token, "secret_root": vault_secrets_path, "secret_path": secret_path, "secret_key": user_key }'

API configuration
-----------------

To perform the LUKSctl API, Laniakea creates a ``luksctl_api`` user on the Virtual Machine, and install the LUKSctl on its home directory. This user can only run the LUKS commands as super user, for security reasons. Finally, to sercure API communications, a self signed SSL certificate is created and installed.

The LUSKctl API currently support both single VMs and Cluster. Moreover, if the encrypted volume is used to host the Docker Engine files, it can be configured to correctly manage this scenario. This is managed using a json configuration file ``config.json``.

.. note::

   Laniakea provides automaric configuration for LUKSctl APIs.

*************
``Single VM``
*************

:Description:
	This is the default API configuration.

:config.json:
	::

	  {
	    "INFRASTRUCTURE_CONFIGURATION": "single_vm"
	  }

**********
``Docker``
**********

:Description: The Docker engine files are installed on the encrypted storage, so the Docker daemon needs to be restarted after LUKS volume mount. If ``VIRTUALIZATION_TYPE`` is set at ``docker`` after LUKS volume mount, the Docker daemon is restarted.

:config.json:
	::

	  {
	    "INFRASTRUCTURE_CONFIGURATION": "single_vm",
	    "VIRTUALIZATION_TYPE": "docker"
	  }

***********
``Cluster``
***********

Current cluster configuration foresee a NFS between front and worker nodes. If the Front End and/or the Worker Nodes are restarted, once the encrypted volume is opened and mounted, the NFS has to be restarted. If the cluster support is enabled in the API configuration file, after LUKS volum mount, the API contacts each worker nodes, via API, and restart the NFS module.

``Front End configuration``
	
:Description: 
	To enable API cluster support the variable ``INFRASTRUCTURE_CONFIGURATION`` has to be set at ``cluster`` on the front end and the worker nodes list has to be provided.

:config.json:
	::

	  {
	    "INFRASTRUCTURE_CONFIGURATION": "cluster",
	    "WN_IPS": ["127.0.0.1"]
	  }

``Worker Nodes(s) configuration``

:Description:
	On each worker node, the API needs the list of the NFS shared directores. This list is required to check if all directories have been properly mounted.

:config.json:
	::

	  {
	    "NFS_MOUNTPOINT_LIST": ["/home","/export"]
	  }	
