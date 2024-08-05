Galaxyctl: libraries
====================

Galaxyctl is a python script collection for Galaxy management (first start, stop/start/restart/status).

.. note::

   Galaxyctl requires superuser privileges.

.. note::

   Current version: ``v2.0.0``

================  ================
Script            Description
================  ================
galaxyctl_libs    Python libraries for uWSGI socket and stats server management, LUKS volume and Onedata space management.
galaxyctl         Galaxy management script. It integrates Luksctl and Onedatactl commands.
================  ================

Galaxyctl_libs is composed by several modules.

Dependencies
------------

Galaxyctl_libs depends on ``uWSGI`` for Galaxy management (i.e. currently no run.sh support). Moreover ``lsof`` is needed to check listening ports.

::

  uwsgi
  
  lsof


DetectGalaxyCommands
--------------------
Parse galaxy Stop/Start/Restart/Status commands. Currently it supports supervisord or systemd/upstart

UwsgiSocket
-----------
Get uWSGI socket from galaxy.ini config file (e.g. 127.0.0.1:4001) and using ``lsof`` return uWSGI master PID.

::

  master_pid, stderr, status = UwsgiSocket(fname='/home/galaxy/galaxy/config/galaxy.ini').get_uwsgi_master_pid()

UwsgiStatsServer
----------------
Read uWSGI stats server json.
The stats server is the last software which uWSGI run during galaxy start procedure. When the stats server is ready, galaxy is ready to accept requests.
Stats server address and port can be specified, but the class is able to read galaxy.ini file to recover stats informations.
Reading Stats json the class is able to detect if uWSGI workers accept requests or not.

=========  ====================
Inputs     Description
=========  ====================
server     uWSGI stats server address, e.g. 127.0.0.1
port       uUWSG stats server port, e.g. 9191
timeout    Wait time, in seconds, for the Stats server start. If galaxy is starting, ``300`` seconds as timeout is ok, while if galaxy is already running ``5`` seconds are enough.
fname      Galaxy config file, e.g. /home/galaxy/galaxy/config/galaxy.ini
=========  ====================

GetUwsgiStatsServer
*******************
To connect to running uWSGI stats server call:

::

   stats = UwsgiStatsServer(timeout=300, fname='/home/galaxy/galaxy/config/galaxy.ini)
   socket = stats.GetUwsgiStatsServer()

GetUwsgiStatsServer
*******************
To check if at least one uWSGI workers accept requests, call:

::

   stats = UwsgiStatsServer(timeout=300, fname='/home/galaxy/galaxy/config/galaxy.ini)
   status = stats.GetUwsgiStatsServer('/home/galaxy/galaxy/config/galaxy,ini')

GetBusyList
***********
To get the list of busy uWSGI workers:

::

  stats = UwsgiStatsServer(timeout=5, fname='/home/galaxy/galaxy/config/galaxy.ini)
  busy_list = stats.GetBusyList()
