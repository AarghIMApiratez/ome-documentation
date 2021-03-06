OMERO.web administration
========================

OMERO.web is the web application component of the OMERO platform which
allows for the management, visualization (in a fully multi-dimensional
image viewer) and annotation of images from a web browser. It also
includes the ability to manage users and groups.

Please read through :doc:`install-web/web-deployment` first.
We assume that OMERO.web has been in installed in a virtual environment
as described in :doc:`install-web/web-deployment`.

.. toctree::
    :maxdepth: 1
    :titlesonly:
    :hidden:

    install-web/web-deployment

Logging in to OMERO.web
-----------------------

Once you have deployed and started the server, you can use your browser
to access the OMERO.webclient:

-  **http://your\_host/omero** OR, for development server:
   **http://localhost:4080**

.. figure:: /images/installation-images/login.png
   :align: center
   :width: 55%
   :alt: OMERO.web login

   OMERO.web login

.. _omero_web_maintenance:

OMERO.web maintenance
---------------------

If an attempt is made to access OMERO.web whilst
it is not running, the generated NGINX configuration file will automatically
display a maintenance page.

-  Session cookies :property:`omero.web.session_expire_at_browser_close`:

   -  A boolean that determines whether to expire the session when the user
      closes their browser.
      See :djangodoc:`Django Browser-length sessions vs. persistent
      sessions documentation <topics/http/sessions/#browser-length-vs-persistent-sessions>`
      for more details. The default value is ``True``::

          $ omero config set omero.web.session_expire_at_browser_close "True"

   -  The age of session cookies, in seconds. The default value is ``86400``::

          $ omero config set omero.web.session_cookie_age 86400

- Clear session:

  Each session for a logged-in user in OMERO.web is kept in the session 
  store. Stale sessions can cause the store to grow with time. OMERO.web 
  uses by default the OS file system as the session store backend and 
  does not automatically purge stale sessions, see
  :djangodoc:`Django file-based session documentation <topics/http/sessions/#using-file-based-sessions>` for more details. It is therefore the responsibility of the OMERO 
  administrator to purge the session cache using the provided management command::
      
      $ omero web clearsessions

  It is recommended to call this command on a regular basis, for example 
  as a :download:`daily cron job <omero-web-cron>`, see
  :djangodoc:`Django clearing the session store documentation <topics/http/sessions/#clearing-the-session-store>` for more information.


.. _customizing_your_omero_web_installation:

Customizing your OMERO.web installation
---------------------------------------

OMERO.web offers a number of configuration options.
The configuration changes **will not be applied** until
Gunicorn is restarted using ``omero web restart``.

-  Session engine:

  -  OMERO.web offers alternative session backends to automatically delete stale data using the cache session store backend, see :djangodoc:`Django cached session documentation <topics/http/sessions/#using-cached-sessions>` for more details.

  - `Redis <https://redis.io/>`_ requires `django-redis <https://github.com/jazzband/django-redis/>`_ in order to be used with OMERO.web. We assume that Redis has already been installed. Follow the step-by-step deployment guides from :doc:`install-web/web-deployment`. To configure the cache, run::

      $ omero config set omero.web.caches '{"default": {"BACKEND": "django_redis.cache.
      RedisCache", "LOCATION": "redis://127.0.0.1:6379/0"}}'

  -  After installing all the cache prerequisites set the following::

        $ omero config set omero.web.session_engine django.contrib.sessions.backends.cache


- Use a prefix:

  By default OMERO.web expects to be run from the root URL of the webserver.
  This can be changed by setting :property:`omero.web.prefix` and
  :property:`omero.web.static_url`. For example, to make OMERO.web appear at
  `http://example.org/omero/`::

      $ omero config set omero.web.prefix '/omero'
      $ omero config set omero.web.static_url '/omero/static/'

  and regenerate your webserver configuration (see :ref:`omero_web_deployment`).

- Use a different host:

  The front-end webserver e.g. NGINX can be set up to run on a different
  host from OMERO.web. You will need to set
  :property:`omero.web.application_server.host` to ensure OMERO.web is
  accessible on an external IP.

All configuration options can be found on various sections of
:ref:`web_index` developers documentation. For the full list, refer to
:ref:`web_configuration` properties or::

	$ omero web -h

The most popular configuration options include:

-  Debug mode, see :property:`omero.web.debug`.

-  Customizing OMERO clients e.g. to add your own logo to the login page
   (:property:`omero.web.login_logo`) or use an index page as an alternative
   landing page for users (:property:`omero.web.index_template`). See
   :doc:`/sysadmins/customization` for further information.

-  Enabling a public user see :doc:`/sysadmins/public`.

Setting up CORS
---------------

Cross Origin Resourse Sharing allows web applications hosted at other origins
to access resources from your OMERO.web installation.
This can be achieved using the
`django-cors-headers <https://github.com/ottoyiu/django-cors-headers>`_ app
with additional configuration of OMERO.web. See the
`django-cors-headers <https://github.com/ottoyiu/django-cors-headers>`_ page
for more details on the settings.

In the virtual environment where OMERO.web is installed, install the app as **root**::

  $ pip install 'django-cors-headers<3.3'

And add it to the list of installed apps as the **omero-web** system user::

  $ omero config append omero.web.apps '"corsheaders"'

Add the cors-headers middleware. Configuration of OMERO.web middleware
was added in OMERO 5.3.2 and uses an 'index' to specify the ordering of middleware classes.
It is important to add the ``CorsMiddleware`` as the first class and
``CorsPostCsrfMiddleware`` as the last, for example::

  $ omero config append omero.web.middleware '{"index": 0.5, "class": "corsheaders.middleware.CorsMiddleware"}'
  $ omero config append omero.web.middleware '{"index": 10, "class": "corsheaders.middleware.CorsPostCsrfMiddleware"}'

Specify which origins are allowed access::

  $ omero config set omero.web.cors_origin_whitelist '["hostname.example.com"]'

Or allow access from all origins::

  $ omero config set omero.web.cors_origin_allow_all True
