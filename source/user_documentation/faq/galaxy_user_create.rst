User creation error
-------------------

Anonymous login is disabled by default on Laniakea Galaxy instances. On Galaxy 19.05 there is a bug in the Admin section to create the user.

.. note::

   Only Galaxy 19.05 is affected.

.. figure:: img/galaxy_admin_user_creation.png
   :scale: 40 %
   :align: center

Once you **create** the user, you will be redirect to the Admin panel. Currently, Galaxy is unable to set it correctly.

.. figure:: img/galaxy_admin_redirect_error.png
   :scale: 40 %
   :align: center

Actually, the user is correctly created and this is only redirect problem.

.. figure:: img/galaxy_admin_user_created.png
   :scale: 40 %
   :align: center

Galaxy is, in fact, wrongly redirecting you to http://<ip_address>/admin/users and not in the subpath http://<ip_address>/galaxy/admin/users. You need just to go on the Galaxy home page and then in the admin panel, to see the user created.
