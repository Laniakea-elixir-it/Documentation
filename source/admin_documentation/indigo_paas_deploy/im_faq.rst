Where are the deployments log?
******************************

The deployment logs are available in `/var/tmp/.im/<im-id>/<deployment_ip>/ctxt_agent.log`. For example:

::

  # tail -f /var/tmp/.im/1b0e064c-9a29-11e7-9c45-300000000002/90.147.102.27_0/ctxt_agent.log

.. Note::

   After each ansible role run, the log file is deleted!!
