Authentication and Authorization Controller (AAC)
=================================================

This module exposes the OAuth2.0 protocol functionality for user authentication and
for the authorization of resource access (e.g., APIs). The module allows for managing 
user registration and for authenticating user using social accounts, in particular Google+ and Facebook. 
Besides, this component manages the role assignment to users used in the resource
access authorization.

.. toctree::

Usage Scenarios
-----------------

Using AAC for Authentication
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is a scenario where AAC is used as an Idenity Provider in a federated authentication environment. 

To enable this scenario, AAC exposes OAuth2.0 protocol. Specifically, it is possible to use 
*OAuth2.0 Implicit Flow* as follows.

**Register Client App** 

Create Client App with AAC developer console (*/aac/*).
To do this

- Login with authorized account (see access configuration above);
- Click *New App* and specify the corresponding client name
- In the *Settings* tab check *Server-side* and *Browser access* and select the identity providers to be used 
  for user authentication (e.g., google, internal, or facebook). Specify also a list of allowed redirect addresses (comma separated).
- In the *Permissions* tab select *Basic profile service* and check *profile.basicprofile.me* and 
  *profile.accountprofile.me* scopes. These scopes are necessary to obtain the information of the currently signed 
  user using AAC API.
 
If API Manager is engaged, create a new API Manager application and subscribe the AAC API. 
         
**Activate Implicit Flow Authorization**

This flow is suitable for the scenarios, where the client application (e.g., client part of a Web app) makes the 
authentication and then direct access to the API without passing through its own Web server backend. 
This allows for generating only a token for a short time period, so the next time the API access is required, 
the authentication should be performed again. In a nutshell, the flow is realized as follows:

* The client app, when there is a need for the token, emits an authorization request to AAC in a browser window. 
  ``https://myaac.instance.com/aac/eauth/authorize``. The request accepts the following set of parameters

  * ``client_id``: the client_id obtained in developer console Indicates the client that is making the request. The value passed in this parameter must exactly match the value in the console.
  *  ``response_type`` with value ``token``,  which determines if the OAuth 2.0 endpoint returns a token.
  *  ``redirect_uri``: URL to which the AAC will redirect upon user authentication and authorization. The value of this parameter must exactly match one of the values registered in the APIs Console  (including the http or https schemes, case , and trailing ‘/’).
  * ``scope``: space-delimited set of permissions the application requests Indicates the access your application is requesting. 

* AAC redirects the user to the authentication page, where the user selects one of the identity providers and performs
  the sign in.
* Once authenticated, AAC asks the user whether the permissions for the requested operations may be granted.
* If the user accepts, the browser is redirected to the specified redirect URL, attaching the token data in the 
  url hash part: ::
  
  http://www.example.com#access\_token=025a90d4-d4dd-4d90-8354-779415c0c6d8&token\_type=Bearer&expires\_in=3600

* Use the obtained token to obtain user data using the AAC API: ::

      GET /aac/basicprofile/me HTTPS/1.1 
      Host: aacserver.com 
      Accept: application/json 
      Authorization: Bearer 025a90d4-d4dd-4d90-8354-779415c0c6d8


  The result of the invocation describes basic user properties (e.g., userId) that can be used to uniquely identify the 
  user.
  
  
**Using AAC for Securing Resources and Services**

In this scenario the goal is to restrict access to the protected resources (e.g., an API endpoint). Also in this case
the scenario relies on the use of OAuth2 protocol. The two cases are considered here:

* The protected resources deal with user-related data or operation. In this case (according to OAuth2.0), the access to the API should be accompanied with the ``Authorizatio`` header that contains the access token obtained via Implicit Flow (or via Authorization Code Flow).
* The protected resource does not deal with user-related data or operation and is not performed client-side. In this case, the access to the API should also be accompanied with the ``Authorizatio`` header that contains the access token obtained via OAuth2.0 client credentials flow.
  
The protected resource will use the OAuth2.0 token and dedicated AAC endpoints to ensure that the token is valid and (in case of user-related data) to verify user identity.  If the token is not provided or it is not valid, the protected resource should return 401 (Unauthorized) error to the caller. 

**Generating Client Credentials Flow Token**

In case the access to the non user-resource is performed, it is possible to use access token obtained through
OAuth2 client credentials flow. In this flow, the resulting token is associated to an client application only. 

The simplest way to obtain such token is through the AAC development console: on the *Overview* page of the client app use the *Get client credentials flow token* link to generate the access token. Note that the token is not expiring and therefore may be reused.

Alternatively, the token may be obtained through the AAC OAuth2.0 token endpoint call: ::

    POST /aac/oauth/token HTTPS/1.1
    Host: aacserver.com 
    Accept: application/json 
    client_id=23123121sdsdfasdf3242&
    client_secret=3rwrwsdgs4sergfdsgfsaf&
    grant_type=client_credentials
    
The following parameters should be passed:

* ``grant_type``: value ``client_credentials``
* ``client_id``: client app ID
* ``client_secret``: client secret

A successful response is returned as a JSON object, similar to the following: ::

    {
    "access_token": "025a90d4-d4dd-4d90-8354-779415c0c6d8",
    "token_type": "bearer",
    "expires_in": 38937,
    "scope": "profile.basicprofile.all"      
    }    
    
Finally, if the API Manager is used, the token may be obtained directly from the API Manager console.   
    

Data Model
-----------------

Roles
^^^^^^^^^^

In AAC the roles are contextualized in order to support varios multitenancy scenarios. The general idea is 
that the roles are associated to the role *spaces* uniquely identified by their namespace and managed
by the space *owners*. A user may have different roles in different spaces and the authorization
control for individual organizations, components, and deployment may be performed within the corresponding space. 
More specifically, each role is represented as a tuple with

* ``context``, defining the "parent" space of roles (if any)
* ``space``, defining the specific role space value. Together with the context form the unique role namespace 
* ``role``, defining the specific role value for the space.

In this way, the spaces may be hierarchically structured (e.g., departments of an organization may have their
own role space within the specific organization space).

Syntactically, the role is therefore represented in the following form: ``<context>/<space>:<role>``. To represent
the owner of the space the role therefore should have the following signature: ``<context>/<space>:ROLE_PROVIDER``.

The owner of the space may perform the following functionalities:

* associate/remove users to/from the arbitrary roles in the owned spaces (including other owners).
* create new child spaces within the owned ones. This is achieved through creation of the corresponding owner roles for the child space being created: ``<parentcontext>/<parentspace>/<childspace>:ROLE_PROVIDER``.

The operation of user role management and space management may be performed either via API or through
the AAC console. 

Some of the role spaces may be pre-configured for the AAC. These include:

* ``apimanager``: Role space for the API Manager tenants.
* ``authorization``: Role space for managing the Authorization API grant tenants.
* ``components``: Role space for managing tenants of various platform components. 
    

AAC API
----------
  
The Swagger UI for the AAC API is available at ``http://localhost:8080/aac/swagger-ui.html``.   
  
Profile API  
^^^^^^^^^^^^

To obtain the basic user data the following call should be performed (scope ``profile.basicprofile.me``):  :: 

    GET /aac/basicprofile/me HTTPS/1.1 
    Host: aacserver.com 
    Accept: application/json 
    Authorization: Bearer <token-value>  
  
If the token is valid, this returns the user data: ::

    {
    "name": "Mario",
    "surname": "Rossi",
    "userId": "6789",
    "username": "mario@gmail.com"
    }  

To obtain the account-related data (e.g., the Identity Provider-specific attributes),  the following call should be performed  (scope ``profile.accountprofile.me``): ::  

    GET /aac/accountprofile/me HTTPS/1.1 
    Host: aacserver.com 
    Accept: application/json 
    Authorization: Bearer <token-value>  
  
If the token is valid, this returns the user data, e.g., ::

    {
      "name": "Mario",
      "surname": "Rossi",
      "username": "rossi@gmail.com",
      "userId": "1",
      "accounts": {
        "google": {
          "it.smartcommunitylab.aac.surname": "Rossi",
          "it.smartcommunitylab.aac.username": "rossi@gmail.com",
          "it.smartcommunitylab.aac.givenname": "Mario",
          "email": "rossi@gmail.com"
        }
      }
    }

Token API
^^^^^^^^^^^^^^^^^^^^

To get the information associated to the token (ITEF RFC7662), the following API may be used ::

    POST /aac/token_introspection?token=<token-value> HTTPS/1.1 
    Host: aacserver.com 
    Accept: application/json 
    Authorization: Basic <client-credentials>  

The data provided represents the information about the app, the user, validity, and scopes. ::

        {
          "active": true,
          "client_id": "7b4f9b2a-71f6-412d-93e6-030c14910083",
          "scope": "profile.basicprofile.me profile.accountprofile.me openid"
          "username": "admin@carbon.super",
          "token_type": "Bearer",
          "sub": "8",
          "iss": "https://aac.example.com",
          "aud": "7b4f9b2a-71f6-412d-93e6-030c14910083", 
          "exp": 123456789,
          "iat": 123450000,
          "nbf": 123456788,
          "aac_user_d": "8",
          "aac_grantType": "implicit",
          "aac_applicationToken": false,
          "aac_am_tenant": "tenant1.com"
        }

Role API
^^^^^^^^^^

The role API allows for the checking the role of the specific users. More details can be found on the
Swagger documentation.

OpenID API
^^^^^^^^^^

The OpenID userinfo endpoint allows for getting the standard user info claims (scopes ``profile``, ``email``). The response is provided
in the form of JSON object or JWT token. ::

    GET /aac/userinfo HTTPS/1.1 
    Host: aacserver.com 
    Accept: application/json 
    Authorization: Bearer <token-value>  

The data provided represents the subset of standard OpenID claims. ::

      {
        "sub": "123456789",
        "name": "Mario Rossi",
        "preferred_username": "rossi@mario.com",
        "given_name": "Mario",
        "family_name": "Rossi",
        "email": "rossi@mario.com",
      }


Configuration
----------------

Installation Requirements
^^^^^^^^^^^^^^^^^^^^^^^^^^^

- RDBMS (tested on MySQL 5.5+)
- Java 1.8+
- MongodDB 3.4+ (optional, needed for Authorization Module)
- Apache Maven (3.0+)


AAC configuration properties are configured through the corresponding Maven profile settings. By default, ``local`` 
profile is activated and the execution environment is configured in the ``src/main/resources/application-local.yml`` file. 
The example configuration for the local execution is defined in ``application-local.yml.example``.
  

DB Configuration
^^^^^^^^^^^^^^^^^^^^

DB Configuration is performed via the JDBC driver configuration, e.g. for MySQL::

    jdbc:
      dialect: org.hibernate.dialect.MySQLDialect
      driver: com.mysql.jdbc.Driver
      url: jdbc:mysql://localhost:3306/aac
      user: ac
      password: ac

The schema and the user with the privileges should have been already configured.
**IMPORTANT!** The default configuration already contains MySQL driver version 5.1.40. In case a newer version 
of the DB is used, update the driver version in ``pom.xml`` accordingly.

AAC Endpoint Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The way the AAC is exposed is configured via the following properties:

- ``server.host`` (or similar Spring Boot settings). Defaults to ``localhost:8080``
- ``server.contextPath``. Defaults to ``/aac``
- ``application.url``. The reverse proxy endpoint for the AAC needed, e.g., for the authentication callback. 

Security Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following security properties MUST be configured for the production environment:

- ``admin.password``: the password for the AAC administrator user.
- ``admin.contexts``: default role contexts managed by the platform. Defaults to ``apimanager, authorization, components``
- ``admin.contextSpaces``: default role spaces associated to the admin. Defaults to ``apimanager/carbon.super`` (used by WSO2 API Manager as a default tenant)
- ``security.rememberme.key``: the secret key used to sign the remember me cookie.

For the user registration and the corresponding communications (email verification, password reset, etc), the 
SMTP email server should be configured, e.g.: ::

    mail:
      username: info@example.com
      password: somepassword
      host: smtp.example.com
      port: 465
      protocol: smtps 
 
Configuring Identity Providers
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default, AAC relies on the internally registered users. However, the integration with the external
identity provider is supported. Out of the box, the support for the OAuth2.0 providers can be achieved 
through the configuration files. Specifically, to configure an OAuth2.0 identity provider it is necessary
to specify the OAuth2.0 client configuration in the properties, e.g. for Facebook, ::

    oauth-providers:
      providers:
        - provider: facebook
          client:
            clientId: YOUR_FACEBOOK_CLIENT_ID
            clientSecret: YOUR_FACEBOOK_CLIENT_SECRET
            accessTokenUri: https://graph.facebook.com/oauth/access_token
            userAuthorizationUri: https://www.facebook.com/dialog/oauth
            preEstablishedRedirectUri: ${application.url}/auth/facebook-oauth/callback
            useCurrentUri: false
            tokenName: oauth_token
            authenticationScheme: query
            clientAuthenticationScheme: form
            scope:
                - openid
                - email
                - profile    

Besides, it is necessary to describe the provider and the possible identity mappings in
``src/main/resources/it/smartcommunitylab/aac/manager/authorities.xml`` file. The configuration amounts to
defining the properties to extract, mapping name and surname properties, defining the unique identity attribute
for the provider, and relations to the other providers' attributes (e.g., via email). For example, ::

    <authorityMapping name="google" url="google" public="true" useParams="true">
        <attributes alias="it.smartcommunitylab.aac.givenname">given_name</attributes>
        <attributes alias="it.smartcommunitylab.aac.surname">family_name</attributes>
        <attributes alias="it.smartcommunitylab.aac.username">email</attributes>
        <attributes>name</attributes>
        <attributes>link</attributes>
        <identifyingAttributes>email</identifyingAttributes>
    </authorityMapping>

defines the Google authentication attributes. Specifically, the ``email`` attribute is used to uniquely identify
Google users.  


Authorization Module
^^^^^^^^^^^^^^^^^^^^^^^^^^^

AAC allows for integration of the authorization module, where it is possible to configure the access rights for
the user at the level of data. To enable the authorization module, the corresponding ``authorization`` Maven profile
should be activated. 
 
Logging Configuration 
^^^^^^^^^^^^^^^^^^^^^^^^^^^

The logging settings may be configured via standard Spring Boot properties, e.g., for the log level

    logging:
      level:
        ROOT: INFO

The project relies on the Logback configuration (see ``src/main/resources/logback.xml``). The default 
configuration requires the log folder path defined with ``aac.log.folder`` property. (if the property is not set, application will use default value: `WORKING_DIRECTORY/logs`). 
 
OpenID Configuration 
^^^^^^^^^^^^^^^^^^^^^^^^^^^

AAC provide a basic implementation of the OpenID protocol. The implementation is based on the  `MitreID <https://mitreid.org/>`_ project.
 
To configure the issuer, it is necessary to specify the OpenID issuer URL: ::
    
    openid:
      issuer: http://localhost:8080/aac
      
OpenID extension requires RSA keys for JWT signature. The project ships with the pre-packaged generated key. 
The key MUST be replaced with your specific value in production environment. To generate new key please follow 
the instructions available `here <https://github.com/mitreid-connect/json-web-key-generator>`_. 

The resulting key should be placed in the resources (i.e., src/main/resources).

The OpenId metadata is available at ``/.well-known/openid-configuration``.

Execution
^^^^^^^^^^^^^^^^^^^^^^^^^^^

To execute from command line, use maven Spring Boot task: ::

    mvn -Plocal spring-boot:run
    
In case you run the tool from the IDE, add the profile configuration to the VM parameters: ::

    -Dspring.profiles.active=local

To enable the authorization module, add the corresponding profile to the profile list (comma-separated)

Once started, the AAC tool UI is available at ``http://localhost:8080/aac``.
