Galaxyctl: APIs
===============

A set of RESTFul APIs is distributed with Galaxyctl. It is written using python Flask micro framework and Gunicorn.

A systemd unit file is used for start/stop/restart the API.

=============  =========  ====================
Moudule        Action     Description
=============  =========  ====================
galaxyctl-api  status     Show status
|              stop       Stop the API
|              start      Start the API.
|              restart    Restart the API.
=============  =========  ====================

.. note::

   Galaxyct-api is configured to listen on ``5001`` port.

::

  $ sudo systemctl status galaxyctl-api
  ● galaxyctl-api.service - Gunicorn instance to serve luksctl api server
     Loaded: loaded (/etc/systemd/system/galaxyctl-api.service; enabled; vendor preset: disabled)
     Active: active (running) since Wed 2019-10-09 16:49:57 UTC; 2 weeks 2 days ago
   Main PID: 15648 (gunicorn)
     CGroup: /system.slice/galaxyctl-api.service
             ├─15648 /home/galaxy/.galaxyctl/api/venv/bin/python /home/galaxy/.galaxyctl/api/venv/bin/gunicorn --workers 2 --b...
             ├─15662 /home/galaxy/.galaxyctl/api/venv/bin/python /home/galaxy/.galaxyctl/api/venv/bin/gunicorn --workers 2 --b...
             └─15663 /home/galaxy/.galaxyctl/api/venv/bin/python /home/galaxy/.galaxyctl/api/venv/bin/gunicorn --workers 2 --b...
  
  Oct 09 16:49:57 vnode-0.localdomain systemd[1]: Started Gunicorn instance to serve luksctl api server.
  Oct 09 16:49:58 vnode-0.localdomain gunicorn[15648]: [2019-10-09 16:49:58 +0000] [15648] [INFO] Starting gunicorn 19.9.0
  Oct 09 16:49:58 vnode-0.localdomain gunicorn[15648]: [2019-10-09 16:49:58 +0000] [15648] [INFO] Listening at: http://0....5648)
  Oct 09 16:49:58 vnode-0.localdomain gunicorn[15648]: [2019-10-09 16:49:58 +0000] [15648] [INFO] Using worker: sync
  Oct 09 16:49:58 vnode-0.localdomain gunicorn[15648]: [2019-10-09 16:49:58 +0000] [15662] [INFO] Booting worker with pid: 15662
  Oct 09 16:49:58 vnode-0.localdomain gunicorn[15648]: [2019-10-09 16:49:58 +0000] [15663] [INFO] Booting worker with pid: 15663
  Hint: Some lines were ellipsized, use -l to show in full.

It used to connect the Laniakea Dashboard to the Galaxy instances, allowing end-user to perform some actions, e.g. to restart Galaxy, without accessing the Virtual Machine with SSH.

Currently, supported APIs are:

******************
``Restart Galaxy``
******************

A POST request is used to restart Galaxy if offline. To prevent unwanted restart, the API check if Galaxy is on line. If yes it return ``on-line`` else it run the galaxy-startup script. Also NGINX is restarted.

Example request:

::

  $ curl 'http://<galaxy_ip_address>:5001/galaxyctl_api/v1.0/galaxy-startup' -i -X POST -H 'Content-Type: application/json' -d '{"endpoint": "http://<galaxy_ip_address>/galaxy"}' 
