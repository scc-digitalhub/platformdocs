Configuration and Installation
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

Docker 
^^^^^^^^^^^^

TODO