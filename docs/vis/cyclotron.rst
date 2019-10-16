Cyclotron
=============

   Cyclotron is a browser-based platform for creating dashboards. It provides standard boilerplate and plumbing, allowing non-programmers to easily create and edit dashboards using customizable components. It has a built-in dashboard editor, and hosts the dashboards directly.

   Dashboards are defined declaratively as a JSON document, which contains all the properties required to render the Dashboard.

Cyclotron is included in the platform as a fork of `Expedia's software <https://github.com/ExpediaInceCommercePlatform/cyclotron/tree/1.48.0>`_. Head over to https://www.cyclotron.io/ for an in-depth description of its functionalities. The fork introduces the following features:

- security based on Authentication and Authorization Control Module (AAC)
- new widgets: time slider, OpenLayers map, Google charts, Deck.gl, widget container
- new data sources: OData
- parameter-based interaction between dashboard components

.. toctree::
   cyclotron_aac
   cyclotron_slider
   cyclotron_openlayers
   cyclotron_googlecharts
   cyclotron_deckgl
   cyclotron_container
   cyclotron_odata
   cyclotron_parameters

Installation
------------

Note that the detailed installation procedure is only summarized here and is better described on `Expedia's page <https://github.com/ExpediaInceCommercePlatform/cyclotron>`_.

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

Using Cyclotron API
-------------------

AAC Roles and Cyclotron Permissions
***********************************

**NOTE**: read Data Model section on AAC page to understand the concepts of *role* and *space*.

In Cyclotron you can restrict access to a dashboard by specifiying a set of **viewers** and **editors**. These can be either users or *groups*. If you use AAC as authentication provider, then groups correspond to AAC *spaces*. By default, owners of a space in AAC have the role ``ROLE_PROVIDER``. In the AAC console, in the tab User Roles, owners (providers) of a space can add other users to it and assign them roles.

When a user logs in via AAC, Cyclotron reads their roles from AAC and assigns them certain groups depending on such roles. Precisely, Cyclotron checks whether the user has roles ``ROLE_PROVIDER``, ``reader`` or ``writer`` in some spaces. Here are some examples of roles:

- user A is provider of space T1 and reader of space T2:

::

   {"context":"components/cyclotron","space":"T1","role":"ROLE_PROVIDER","authority":"components/cyclotron/T1:ROLE_PROVIDER"}
   {"context":"components/cyclotron","space":"T2","role":"reader","authority":"components/cyclotron/T2:reader"}

- user B is reader of space T1:

::

   {"context":"components/cyclotron","space":"T1","role":"reader","authority":"components/cyclotron/T1:reader"}

- user C is writer of space T1:

::

   {"context":"components/cyclotron","space":"T1","role":"writer","authority":"components/cyclotron/T1:writer"}

When these users log in to Cyclotron via AAC they are assigned the following property:

- user A: ``memberOf: ['T1_viewers', 'T1_editors', 'T2_viewers']``
- user B: ``memberOf: ['T1_viewers']``
- user C: ``memberOf: ['T1_viewers', 'T1_editors']``

**Note 1**: providers and writers are equally considered editors by Cyclotron, i.e., user A as provider of T1 is member of T1_editors group.

**Note 2**: editors can also view, i.e., users A and C being members of T1_editors are also members of T1_viewers; but viewers cannot edit, i.e., groups *<group_name>_viewers* cannot be assigned as editors of a dashboard.

Restricting Access to Dashboards in JSON
****************************************

If you create a dashboard as a JSON document (either by POSTing it on the API or via JSON document editor on the website), this is its skeleton:

::

   {
       "tags": [],
       "name": "foo",
       "dashboard": {
           "name": "foo",
           "pages": [],
           "sidebar": {
               "showDashboardSidebar": true
           }
       },
       "editors": [],
       "viewers": []
   }

NOTE: if a dashboard has no editors or viewers specified, by default the permissions are restricted to the dashboard creator only, who is allowed to change this behaviour later on.

Resuming the example above, suppose user A wants to restrict access to its dashboard:

- dashboard editors list: can contain only group **T1_editors** or its members (e.g. user C)
- dashboard viewers list: can contain groups **T1_viewers** (*not* T1_editors as it is already a subset of T1_viewers, since editors are automatically also viewers) and **T2_viewers** or their members (e.g. users B and C)

Each editor or viewer specified in the lists must have three mandatory properties: ``category`` (either "User" or "Group"), ``displayName`` (used for readability purpose) and ``dn`` (unique name that identifies the user or group; corresponds to ``distinguishedName`` property in Cyclotron API User model).

Example 1. User A restricts edit permissions to themselves and gives view permissions to group T2:

::

   "editors": [{
       "category": "User",
       "displayName": "John Doe",
       "dn": "A"
   }],
   "viewers": [{
       "category": "Group",
       "displayName": "T2",
       "dn": "T2_viewers"
   }]

Example 2. User A restricts both edit and view permissions to group T2, i.e., every T2 member can view but only editor members can edit:

::

   "editors": [{
       "category": "Group",
       "displayName": "T2",
       "dn": "T2_editors"
   }],
   "viewers": [{
       "category": "Group",
       "displayName": "T2",
       "dn": "T2_viewers"
   }]

In short: use *<group_name>_editors* syntax for editors and *<group_name>_viewers* syntax for viewers.

If authentication is enabled, only dashboards that have no restriction on viewers can be viewed anonymously.

Testing
*******

If you want to test the authorization mechanism (e.g. on Postman), you can use the following URLs, setting the ``Authorization`` header with an appropriate token:

- to create a dashboard, POST on ``http://localhost:8077/dashboards``
- to update a dashboard, PUT on ``http://localhost:8077/dashboards/{dashboard_name}``
- to retrieve a dashboard, GET on ``http://localhost:8077/dashboards/{dashboard_name}``

Deployment
----------

TODO
