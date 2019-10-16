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
----------------------

On the **web app**, login can be performed:

- via LDAP by providing username and password
- via AAC using OAuth2 protocol, i.e., being redirected on AAC for authentication

On the **API**, requests can be authenticated:

- via session key
- via ``Authentication`` HTTP header
- via API key

Session key is the internal mechanism used by Cyclotron web app to authenticate its requests to the API.

Authentication header must provide a valid token issued by AAC. It is used by the web app if login is performed via AAC and can be used to request API services directly:

::

   GET /dashboards/mydashboard HTTP/1.1
   Host: localhost:8077
   Accept: application/json
   Authorization: Bearer 025a90d4-d4dd-4d90-8354-779415c0c6d8

API key is issued by AAC and can be passed as query parameter in the URL to request API services:

::

   http://localhost:8077/dashboards/mydashboard?apikey=dee7889d-ef09-474a-b69a-74b2ace47c50

Authentication Configuration with AAC
-------------------------------------
Open ``cyclotron-svc/config/config.js`` and update the properties according to your needs (remember to configure the same properties in the website config file, e.g. the API server URL. To use AAC as authentication provider, be sure to set the following properties with the correct AAC URLs:

.. code:: javascript

   enableAuth: true,
   authProvider: 'aac',
   oauth: {
       userProfileEndpoint: 'http://localhost:8080/aac/basicprofile/me',
       userRolesEndpoint: 'http://localhost:8080/aac/userroles/me',
       scopes: 'profile.basicprofile.me,user.roles.me',
       tokenValidityEndpoint: 'http://localhost:8080/aac/resources/access',
       tokenInfoEndpoint: 'http://localhost:8080/aac/resources/token',
       tokenRolesEndpoint: 'http://localhost:8080/aac/userroles/token',
       apikeyCheckEndpoint: 'http://localhost:8080/aac/apikeycheck',
       parentSpace: 'components/cyclotron'
   }

Open ``cyclotron-site/_public/js/conf/configService.js`` and update it too. Be sure to set the following properties under ``authentication`` (you will set the client ID later):

.. code:: javascript

   enable: true,
   authProvider: 'aac',
   authorizationURL: 'http://localhost:8080/aac/eauth/authorize',
   clientID: '',
   callbackDomain: 'http://localhost:8088',
   scopes: 'profile.basicprofile.me user.roles.me',
   userProfileEndpoint: 'http://localhost:8080/aac/basicprofile/me',
   tokenValidityEndpoint: 'http://localhost:8080/aac/resources/access'

Client Application Configuration on AAC
---------------------------------------

Log in to AAC as a provider user and click "New App" to create a client application. In the Settings tab:

- add Cyclotron website as redirect URL: ``http://localhost:8088/,http://localhost:8088`` (change the domain if it runs on a different host and port)
- check all the Grant Types and at least ``internal`` as identity provider (this must be approved on the Admin account under tab Admin -> IdP Approvals)

In the API Access tab:

- under Basic Profile Service, check ``profile.basicprofile.me`` to give access to user profiles to the client app
- under Role Management Service, check ``user.roles.me`` to give access to user roles

In the Overview tab, copy ``clientId`` property, then go back to ``cyclotron-site/_public/js/conf/configService.js`` and add it in the ``authentication`` section.

Now you can (re)start Cyclotron API and website with authentication enabled. Most services will now be protected and will require login and specific privileges.

**NOTE**: if you need to change the API port you can do it in the configuration file, but changing Cyclotron website port can only be done in ``cyclotron-site/gulpfile.coffee``, inside the Gulp task named ``webserver`` (line 281): update ``port`` and ``open`` properties as needed.




  
.. _Expedia's page: https://github.com/ExpediaInceCommercePlatform/cyclotron
