Galaxyctl: Galaxy management
============================

Galaxyctl is a python script collection used for Galaxy management, to properly check uWSGI Stats and to correctly retrieve Galaxy and uWSGI workers status. It's source code is located on `Laniakea GitHub <https://github.com/Laniakea-elixir-it/galaxyctl>`_.

.. note::

   Since the script parse ``supervisorctl`` or ``systemd`` commands, it needs to be run as superuser.

===========  =========  ====================
Moudule      Action     Description
===========  =========  ====================
galaxy       status     Show galaxy status
|            stop       Stop Galaxy. ``--force`` check uwsgi master process. If it is still running, after galaxy stop, it is killed.
|            start      Start Galaxy. ``--force`` force galaxy to start by restarting it. ``--retry`` option allow to specify number of tentative retart (default 5). ``--timeout`` allow to customize uWSGI stats server wait time. These options are used during galaxy instantiation and you should not use them on production.
|            restart      Restart Galaxy. ``--force`` force galaxy to start by restarting it. ``--retry`` option allow to specify number of tentative retart (default 5). ``--timeout`` allow to customize uWSGI stats server wait time. These options are used during galaxy instantiation and you should not use them on production.
|            startup    This method is used only to run galaxy for the first time and you shoud not use it in production. ``--retry`` option allow to specify number of tentative retart (default 5). ``--timeout`` allow to customize uWSGI stats server wait time.
===========  =========  ====================

Galaxyctl basic usage
---------------------
The script requires superuser commands to be used. Its basic commands are:

=====================  ==============================
Action                 Command
=====================  ==============================
Start Galaxy           sudo galaxyctl start galaxy
Stop Galaxy            sudo galaxyctl stop galaxy
Restart Galaxy         sudo galaxyctl restart galaxy
Check Galaxy Status    sudo galaxyctl status galaxy
=====================  ==============================

Logging
-------
Logs are stored in ``/var/log/galaxy/galaxyctl.log`` file.

Advanced options
----------------

stop
~~~~
To stop galaxy:

::

  sudo galaxyctl stop galaxy

The script check the uWSGI Stats server to retrieve workers PID and their status. If, after uWSGI stop, workers are still up and running, they are killed, allowing Galaxy to correctly start next time.
The ``--force`` options allow to kill uwsgi master process if it is still alive after galaxy stop (in case of uwsgi FATAL error or ABNORMAL TERMINATION). Please check galaxy logs before run ``--force`` option.

start
~~~~~
To start Galaxy:

::

  sudo galaxyctl start galaxy

Once Galaxy started, galaxyctl waits and check the uWSGI Stats server. Since it is the last software loaded, this ensure that Galaxy has correctly started.
The script also check that at least 1 uWSGI worker has correctly started and it is accepting requests.

If no workers are available you have to restart Galaxy.
Galaxyctl is able to automatically restart galaxy if the option ``--force`` is specified, restarting it until the workers are correctly loaded
The number of retries is set, by default, to 5. It can be customized using ``--retry`` option, e.g. ``--retry 10``.
These options were not designed for production, but are used only during VMs instantiation phase to ensure Galaxy can correctly start.

restart
~~~~~~~
To restart Galaxy:

::

  sudo galaxyctl restart galaxy

The options ``--force``, ``--timeout`` and ``--retry`` are available for restart command too.

Galaxy first start
~~~~~~~~~~~~~~~~~~
Galaxy takes longer to start the first time. Since the uWSGI stats server is the last software component started, the script waits to ensure that Galaxy has correctly started. Then uWSGI workers are checked to ensure Galaxy is accepting requests. If not, uWSGI is restarted.
Currently, before rise an error, the script try to restart galaxy 5 times, while the waiting time is set to 600 seconds.
The command used in ``/usr/local/bin/galaxy-startup`` script, is

::

  galaxyctl startup galaxy -c /home/galaxy/galaxy/galaxy.ini -t 600

Configuration file
-------------------

Supervisord and systemd/upstart are supported to start/stop/restart/status Galaxy. The init system can be set using the variables ``init_system``: two values are, currently, allowed: ``supervisord`` and ``init``

=============  ===========================================
init_system    Explanation
=============  ===========================================
supervisord    Supervisord is current default, it is mandatory for docker container, since there's no systemd on docker images.
init           CentOS 7 and Ubuntu 16.04 use systemd, while  Ubuntu 14.04 is using upstart.
=============  ===========================================

Through ``galaxyctl_libs.DetectGalaxyCommands`` method the script automatically retrieves the right command to be used and it  is compatible with both CentOS 7 and Ubuntu 16.04.

If Supervisord is used to manage Galaxy (which is our default choice), configuration files have to be specified using the variable ``supervisord_config_file``
On CentOS:

::

  supervisord_conf_file = '/etc/supervisord.conf'

while on Ubuntu:

::

  supervisord_conf_file = '/etc/supervisor/supervisord.conf'

Galaxyctl needs galaxy.yml to retrieve uWSGI stats server information, through the variable:

::

  galaxy_config_file = '/home/galaxy/galaxy/config/galaxy.yml'

Features
--------

.. toctree::
   :maxdepth: 2

   galaxyctl_libs
   galaxyctl_api
