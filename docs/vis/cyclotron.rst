Cyclotron
=============

TODO Introduction

Installation
------------

Note that the detailed installation procedure is only summarized here and is better described on `Expedia's page`_.

Requirements:

- AAC (`installation instructions <https://github.com/smartcommunitylab/AAC>`_)
- Node.js
- MongoDB 2.6+ (`installation instructions <http://docs.mongodb.org/manual/installation/>`_)

Installation steps:

0. Ensure that MongoDB and AAC are running. Cyclotron will automatically create a database named "cyclotron".
1. Clone this repository.
2. Install the REST API and create the configuration file ``cyclotron-svc/config/config.js``. Paste in it the content of ``sample.config.js``, which contains the configurable properties of the API, such as MongoDB instance and AAC endpoints.
3. Install Cyclotron website.
4. Start both:

   - API: from ``cyclotron-svc`` run the command ``node app.js``
   - website: from ``cyclotron-site`` run the command ``gulp server``

Now Cyclotron is running with its default settings and authentication is disabled.

Authentication Methods
------------

On the **web app**, login can be performed:

- via LDAP by providing username and password
- via AAC using OAuth2 protocol, i.e., being redirected on AAC for authentication

On the **API**, requests can be authenticated:

- via session key
- via ``Authentication`` HTTP header
- via API key

Session key is the internal mechanism used by Cyclotron web app to authenticate its requests to the API.

Authentication header must provide a valid token issued by AAC. It is used by the web app if login is performed via AAC and can be used to request API services directly:

.. code-block:: html
  GET /dashboards/mydashboard HTTP/1.1
  Host: localhost:8077
  Accept: application/json
  Authorization: Bearer 025a90d4-d4dd-4d90-8354-779415c0c6d8

Normal text

.. toctree::
  dss_install
  dss_usage
  dss_rest_crud
  dss_deploy
  
.. _Expedia's page: https://github.com/ExpediaInceCommercePlatform/cyclotron
