Laniakea Dashboard
==================

The Laniakea Dashboard is the new, redesigned and reimplemented, user interface of Laniakea, developed using:

- `Flask <flask.pocoo.org/>`_ web micro-framework;

- `Jinja2 <jinja.pocoo.org/>`_ template engine;

- `Bootstrap <getbootstrap.com/>`_ 4 toolkit.

Lighter and more flexible than the previous interface, it has been integrated with Hashicorp Vault for user secrets management.

The Laniakea dashboard has, currently, two configuration files, in json format, which can be found in the ``/etc/orchestrator-dashboard`` directory: the ``config.json`` for the dashboard configuration and the ``vault-config.json``  specific for the Vault integretion configuration.

Moreover, the TOSCA templates for each Laniakea application, with the corresponding parameters and metadata file can be found in ``/opt/laniakea-dashboard-config``:

- ``/opt/laniakea-dashboard-config/tosca-templates``: this directory collects the TOSCA templates of applications shown in the dashboard.

- ``/opt/laniakea-dashboard-config/tosca-parameters``: this directory collects the parameters files corresponding to the TOSCA templates.

- ``/opt/laniakea-dashboard-config/tosca-metadata``: this directory collects the metadata files corresponding to the TOSCA templates.

These paths can be configured in the ``config.json`` file.

.. warning::

   The Laniakea configuration files and templates are automatically configured by the installation procedure. Please modify them only if you know what you are doing!

Configuration
-------------

.. toctree::
   :maxdepth: 2

   overview
   config_json
   vault_config_json
   apps_config
   parameters_config
   metadata_config
