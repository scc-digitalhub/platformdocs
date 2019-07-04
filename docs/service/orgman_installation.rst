Installation
=============

*Org-Manager* is available here: https://github.com/smartcommunitylab/AAC-Org

This section will guide you through the steps necessary for running it: installing the server, configuring the connectors and installing the client.

Org-Manager can also run inside a Docker container. Refer to `Running with Docker`_ for instructions.

Server
------

Before you configure Org-Manager, you need to install the **AAC identity provider** and create an app for it.

Configuring the AAC identity provider
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Org-Manager requires the `AAC <https://github.com/smartcommunitylab/AAC>`_ identity provider to work. The repository explains how to install and configure it.

Once AAC is running, create a new app for Org-Manager by accessing the **Client Apps** menu and clicking on **NEW APP**.

In the **Settings** tab, under **redirect Web server URLs**, add the redirect URLs for server and client. If you're running them on *localhost*, for example, add both of the following (assuming the ports are *7979* and *4200*)::

  http://localhost:7979/login
  http://localhost:4200/login

To run the server within a Docker container, you need to add a third URL with the port (for example *7878*) Docker will expose the service through::

  http://localhost:7878/login

For more information on running the server inside a Docker container, see the `Running with Docker`_ section.

For **Grant types**, check ``Implicit`` and ``Client credentials``. For **Enabled identity providers**, check ``internal``.

In the **API Access** tab, grant all permissions under ``Basic profile service`` and under ``Role Management Service`` and save the app.

Finally, all users that will be administrators of Org-Manager, as well as all organization owners, need the following role: ``apimanager/carbon.super:profilemanager``.

Additionally, administrators will need this one too: ``organizations:ROLE_PROVIDER``.

To create the ``apimanager/carbon.super`` space, access the **Space Owners** menu, choose ``apimanager`` as **Parent Space** and click on **NEW USER**. Insert the **Username**, insert ``carbon.super`` under **New spaces** and click **ADD**. Click **UPDATE** to create this space.

Now that the space has been created, all users who will be administrators of Org-Manager, or owners of an organization, need the ``profilemanager`` role within this space.

Access the **User Roles** menu, pick ``apimanager/carbon.super`` as **Role Context**, and then, for each user, click **NEW USER**, insert the **Username**, insert ``profilemanager`` as **New role**, click **ADD** and then **UPDATE**.

Assign the ``organizations:ROLE_PROVIDER`` role to other administrator users in the same way as you did with the ``apimanager/carbon.super:profilemanager`` role.

.. _setting-up-the-server:

Setting up the server
^^^^^^^^^^^^^^^^^^^^^

The ``application.yml`` file contains various properties used by the server. The following is a brief explanation of the main properties and what values should be given to them. While properties in YAML are defined by indentation, this document will translate that indentation with a dot-separated notation, to keep the explanation shorter.

Properties appear with the following format::

  <property_name>: ${<environment_variable_name>:<default_value>}

When the server is run, the value for the property is taken from the indicated environment variable (set by Docker), but, if the environment variable cannot be found (for example when not running with Docker), it uses the default value instead.

For example, the property for the port of the service appears as follows::

  server:
    port: ${OMC_SERVER_PORT:7979}

If you are not running the server inside a Docker container, and want to use a different port, just change the ``7979`` part. For more information on running the server inside a Docker container, see the `Running with Docker`_ section.

- ``server.port`` – The port the server is running at. Sample value: ``7979``

- ``server.servlet.session.cookie.name`` – Name of the session cookie, used to avoid conflicts with other applications that use the name *JSESSIONID* and may be running on the same host. Sample value: ``ORGMANAGERSESSIONID``

- ``spring.datasource.url`` – Database server for the Org-Manager server. The format may vary depending on the database type. A typical format can look like this: ``jdbc:<database type>://<host>:<port>/<database name>``. Sample value: ``jdbc:postgresql://localhost:5432/orgmanager``

- ``spring.datasource.username`` – Name of the user in the database

- ``spring.datasource.password`` – Password of the user in the database

- ``spring.datasource.driver-class-name``: Driver for the database. Sample value: ``org.postgresql.Driver``

- ``spring.jpa.database-platform`` – Dialect for the database. Sample value: ``org.hibernate.dialect.PostgreSQLDialect``

There may be more properties under ``spring`` related to setting up the database.

- ``security.oauth2.client.clientId`` – Client ID for Org-Manager in the identity provider.

- ``security.oauth2.client.clientSecret`` – Client secret for Org-Manager in the identity provider.

- ``security.oauth2.client.accessTokenUri`` – URI for obtaining the access token

- ``security.oauth2.client.userAuthorizationUri`` – URI to obtain authorization by the identity provider

- ``security.oauth2.client.tokenInfoUri`` – URI to inspect the contents of the token

- ``security.oauth2.client.tokenName`` – Name used by the identity provider to identify the access token

- ``security.oauth2.client.userIdField`` – Name used by the identity provider for the field of the token that contains the ID of the user

- ``security.oauth2.client.organizationManagementScope`` – Identifier for the organization management scope, which grants administrator privileges

- ``security.oauth2.client.organizationManagementContext`` – The AAC context within which component contexts are nested. Having the ``ROLE_PROVIDER`` role within this context also grants administrator privileges.

- ``security.oauth2.resource.userInfoUri`` - scope for basic profile information

- ``aac.uri``: AAC host

- ``aac.apis.manageRolesUri`` - AAC API end-point for managing user roles

- ``aac.apis.userProfilesUri`` - AAC API end-point for retrieving profile information, used to associate user name with ID

- ``aac.apis.currentUserRolesApi`` - AAC API end-point for retrieving the authenticated user's roles

- ``aac.apis.currentUserProfileApi`` - AAC API end-point for retrieving the authenticated user's profile

Connectors
----------

All connectors take their configuration parameters from the server, which retrieves them from a single file, `application-components.yml <https://github.com/smartcommunitylab/AAC-Org/blob/master/server/src/main/resources/application-components.yml>`_.

When running the server with Docker, this file is replaced with a different one; see the `Running with Docker`_ section for more information.

Different connectors have different properties, so configuration for each connector is explained in its correspondent section.

Two properties are however required for all connectors:

- ``componentId`` - Identifies the component.

- ``implementation`` - Full name of the connector class that reflects tenant/user management operations on the component. The following value corresponds to a dummy class that causes no changes, to be used if the component does not need an external class for this purpose, or to simply disable a connector::

	it.smartcommunitylab.orgmanager.componentsmodel.DefaultComponentImpl

WSO2 Connector
^^^^^^^^^^^^^^

The WSO2 connector may be used for different WSO2 products, such as *API Manager* or *Data Services Server*.

In order to provide the necessary infrastructure for allowing WSO2 products to interact with Org-Manager, it is necessary to include in **repository/components/dropins** the *jar*'s of the following submodules::

	<module>apim.custom.user.store</module>
	<module>apim.custom.user.store.stub</module>

This extension is done in order to permit the admin account to create, update, delete users and assign/revoke roles within specific tenants and extends the existing `UserStoreManagerService admin <https://github.com/wso2-extensions/identity-user-ws/blob/master/components/org.wso2.carbon.um.ws.service/src/main/java/org/wso2/carbon/um/ws/service/UserStoreManagerService.java>`_.

The configuration steps are the following:
- Build **orgmanager-wso2connector** project with Maven.

- Copy **apim.custom.user.store-0.0.1.jar** from the project *orgmanager-wso2connector/apim.custom.user.store* to the WSO2 directory **repository/components/dropins**

- Copy **apim.custom.user.store.stub-0.0.1.jar** from the project *orgmanager-wso2connector/apim.custom.user.store.stub* to the WSO2 directory **repository/components/dropins**

As a result, the new admin stub can be accessed from the following end-point: ``https://$APIM_URL/services/CustomUserStoreManagerService``

After putting the *jar*'s in the proper folder, you should update the connector's configuration in `application-components.yml <https://github.com/smartcommunitylab/AAC-Org/blob/master/server/src/main/resources/application-components.yml>`_ accordingly for APIM and DSS components.

For example, for API Manager:

-	``name``: Name of the component, only needed for display.

-	``componentId``: ID of the component, should be ``apimanager``

-	``scope``: Scope of the component, should be ``components/apimanager``

-	``format``: Regular expression for the tenants' domains. Should be ``^([a-z0-9]+(-[a-z0-9]+)*\.)+[a-z]{2,}$``

-	``implementation``: Full class name of the class implementing the component. The class designed for API Manager is ``it.smartcommunitylab.apimconnector.APIMConnector``; alternatively, the value ``it.smartcommunitylab.orgmanager.componentsmodel.DefaultComponentImpl`` may be used to disable the API Manager connector.

-	``roles``: Comma-separated list of roles that may be assigned to users via OMC. It should consist of ``ROLE_PUBLISHER`` and ``ROLE_SUBSCRIBER``, as they are the 2 currently supported roles.

-	``host``: URI where NiFi is hosted.

-	``usermgmtEndpoint``: User service stub, should be ``/services/CustomUserStoreManagerService``

-	``usermgmtPassword``: Password for user management, default is ``admin``

-	``multitenancyEndpoint``: Multi-tenancy service stub, should be ``/services/TenantMgtAdminService``

-	``multitenancyPassword``: Password for tenant management, should be ``admin``

NiFi Connector
^^^^^^^^^^^^^^

This section explains how multitenancy works in NiFi and how to configure the connector so that tenancy operations issued by the server are performed in NiFi.

If you're not interested in how multi-tenancy is represented in NiFi, skip to the `Certificates`_ section to create the necessary certificates.

If you don't need to create certificates and only need to configure the connector, skip to the `Configuration`_ section.

Multi-tenancy in NiFi
"""""""""""""""""""""

The idea of multi-tenancy in NiFi is that **process groups** represent tenants and have policies defined for them, listing which **users** or **user groups** are allowed to view or alter them. User groups are equivalent to teams, so if permission to view a process group is given to a user group, all users belonging to it can view it.

Users will still be able to see other teams' process groups on the flow, but they will only appear as rectangles that they can neither interact with or view details of. The only information they can see about them is the amount of data they are processing.

Certificates
""""""""""""

Executing tenant and user management operations in a secured NiFi instance requires specific authorizations, so Org-Manager needs to act with the permissions granted to the administrator user.

Since *OpenID Connect* is used to secure NiFi, we have to authenticate by providing the administrator's SSL certificate and configuring NiFi to recognize it.

This section describes how to do this and is heavily based on a very useful and detailed `article <https://community.hortonworks.com/content/supportkb/151106/nifi-how-to-create-your-own-certs-for-securing-nif.html>`_ by Matt Clarke from the Hortonworks Community.

Two pieces of software are needed for this process:

- `Keytool <https://docs.oracle.com/javase/6/docs/technotes/tools/windows/keytool.html>`_: comes bundled with Java's JRE, so you should find it in your Java installation folder, usually in `C:\\Program Files\\Java\\jre1.8.0_191\\bin`, depending on your version.

- `OpenSSL <https://www.openssl.org/source/>`_

The steps described in this section are written for Windows' *Command Prompt*, so the syntax for paths and the like may vary depending on your OS. Remember to quote paths if they contain spaces.

Step 1: Creating a Certificate Authority (CA)
#############################################

The first thing to do is creating a *Certificate Authority* (*CA*). This CA will sign the administrator's certificate, stating that it can be trusted.

Change directory to the ``bin`` subfolder of your OpenSSL installation (for example, ``cd C:\OpenSSL\openssl-1.0.2j\bin``). If you don't, you will need to replace ``openssl`` in each command with the **path to the openssl.exe file**.

1. .. _nifi-certificates-11:

	**Creating the CA's private key**
	
	This will create the private key for your CA and place it in the ``C:\certs`` folder. If you omit the path and just write ``myCA.key``, it will be in the **same folder as openssl.exe**. You will be asked to choose a password. ::
	
		openssl genrsa -aes128 -out C:\certs\myCA.key 4096
		
2. .. _nifi-certificates-12:
	
	**Creating a pem certificate**
	
	This command creates the CA's certificate. You will be asked to provide the password you chose in :ref:`1.1 <nifi-certificates-11>`. You will then have to fill the CA's profile (country, organization name, etc.): the data you insert in this step is not important for our purposes, but it might be preferable to pick something that will help you recognize this certificate.
	
	`1095` is the validity (in days) of the certificate, feel free to change it as you see fit. ::
	
		openssl req -x509 -new -key C:\certs\myCA.key -days 1095 -out C:\certs\myCA.pem
	
3. .. _nifi-certificates-13:

	**Converting from pem to der**
	
	Converting the certificate into **der** format is necessary for the next step, performed by Keytool. ::

		openssl x509 -outform der -in C:\certs\myCA.pem -out C:\certs\myCA.der
		
Step 2: Creating NiFi's Truststore
##################################

A truststore lists which certificates can be trusted. It is necessary to add the CA's certificate to this truststore, otherwise the CA's signature on the administrator's certificate will be meaningless.

Change directory to where **keytool.exe** is located (probably something like ``C:\Program Files\Java\jre1.8.0_191\bin``), or replace ``keytool`` with the **path to the keytool.exe file**.

1. .. _nifi-certificates-21:

	**Creating the truststore**
	
	This will create the truststore and include the CA's certificate in it, meaning it can be trusted. You will be asked to choose a password for the truststore. It will then show you the CA's certificate and ask you to confirm it can be trusted by typing the word *yes* in your system's language. ::
	
		keytool -import -keystore C:\certs\truststore.jks -file C:\certs\myCA.der -alias myCA
		
2. .. _nifi-certificates-22:

	**Configuring NiFi to use the new truststore**
	
	Open the `nifi.properties` file (it can be found inside the `conf` subfolder of your NiFi installation) and edit the following fields. In newer NiFi versions, the `needClientAuth` field may not be present, in which case you can omit it. The password for the `truststorePasswd` field is the one you chose in :ref:`2.1 <nifi-certificates-21>`. ::

		nifi.security.truststore=C:/certs/truststore.jks
		nifi.security.truststoreType=JKS
		nifi.security.truststorePasswd=MyTruststorePassword
		nifi.security.needClientAuth=true
		
Step 3: Generating a keystore for the NiFi server
#################################################

This is not strictly related to our tenant-providing process; however, when running a secured instance of NiFi, it is necessary for it to have a keystore, and for the browser that accesses NiFi to trust it. Since we just created a CA, we can use it to create NiFi's keystore.

1. .. _nifi-certificates-31:
	
	**Generate a keystore for the NiFi server**
	
	Change directory to the path to **keytool.exe** (for example ``cd C:\Program Files\Java\jre1.8.0_191\bin``), or replace ``keytool`` with the **path to keytool.exe**.

	The following command will generate the keystore. It will ask you to choose a password for the keystore, and then to fill the profile of the certificate, similarly to what happened when generating the CA. When asked for the full name (it should be the first thing asked after password confirmation), insert your domain's name. If you're doing this on localhost, simply type `localhost`.

	Finally, it will ask you to choose a password for the private key. If you just hit enter, it will use the same password as the keystore's. ::

		keytool -genkey -alias nifiserver -keyalg RSA -keystore C:\certs\nifiserver.jks -keysize 2048
		
2. .. _nifi-certificates-32:

	**Generating a certificate sign request**
	
	This command will generate a certificate with a request to sign it. It may ask for both the passwords you chose in :ref:`3.1 <nifi-certificates-31>`: first the keystore's password and then the private key's password. If they are the same, it will only ask once. ::
	
		keytool -certreq -alias nifiserver -keystore C:\certs\nifiserver.jks -file C:\certs\nifiserver.csr
		
3. .. _nifi-certificates-33:

	**Signing the NiFi server's certificate**
	
	Once again, change directory to OpenSSL's ``bin`` subfolder, or replace ``openssl`` accordingly.
	
	This command will have the CA sign your NiFi server's certificate to state that it can be trusted. It will ask for the password you chose in :ref:`1.1 <nifi-certificates-11>`. ::

		openssl x509 -sha256 -req -in C:\certs\nifiserver.csr -CA C:\certs\myCA.pem -CAkey C:\certs\myCA.key -CAcreateserial -out C:\certs\nifiserver.crt -days 730
		
4. .. _nifi-certificates-34:

	**Import the CA's public key into the keystore**
	
	Switch back to Keytool's folder, or replace ``keytool`` accordingly.

	This command will include the CA's public key into your keystore, so that it may be used to verify your certificate's validity. It will ask for the keystore's password, which you chose in :ref:`3.1 <nifi-certificates-31>`. You will have to confirm that the certificate can be trusted by typing *yes* in your system's language. ::

		keytool -import -keystore C:\certs\nifiserver.jks -file C:\certs\myCA.pem
		
5. .. _nifi-certificates-35:

	**Import the signed NiFi server's certificate into the keystore**
	
	This command will import the certificate you signed in :ref:`3.3 <nifi-certificates-33>` into the keystore. It will ask for the two passwords you chose in :ref:`3.1 <nifi-certificates-31>`: first the keystore's password and then the private key's password, or just one of them if they are the same. ::

		keytool -import -trustcacerts -alias nifiserver -file C:\certs\nifiserver.crt -keystore C:\certs\nifiserver.jks
		
6. .. _nifi-certificates-36:

	**Configuring NiFi to use the new keystore**
	
	Open the `nifi.properties` file (from the `conf` subfolder of your NiFi installation) and edit the following properties, using the two passwords chosen in :ref:`3.1 <nifi-certificates-31>`::

		nifi.security.keystore=C:/certs/nifiserver.jks
		nifi.security.keystoreType=JKS
		nifi.security.keystorePassword=MyKeystorePassword
		nifi.security.keyPassword=MyPrivateKeyPassword
		
7. .. _nifi-certificates-37:

	**Adding the CA to your browser**
	
	When accessing NiFi, your browser will likely state that the connection cannot be trusted. This is because, even though the NiFi server's certificate is signed by your CA, your browser does not know your CA.

	It may offer you to add an exception, but at this point you might as well add the CA you created to the list of trusted CAs.

	For example, to do it in Mozilla Firefox:
	
	**Settings** > **Options** > **Privacy and security** > **Show certificates** (on the right, near the bottom, in the *Certificates* section) > **Authorities** tab > **Import** > open your `myCA.pem` file and check both boxes.
	
	You might need to restart your browser. Afterwards, you should be able to access NiFi. If it still says the connection cannot be trusted, you might have inserted the wrong name in :ref:`3.1 <nifi-certificates-31>`, and have to repeat steps :ref:`3.1 <nifi-certificates-31>` through :ref:`3.5 <nifi-certificates-35>`.
	
Step 4: Make the CA sign the administrator's certificate
########################################################

By having the administrator's certificate signed by the CA, it will be recognized as valid by NiFi, since it trusts the CA.
Change directory back to the ``bin`` subfolder of your OpenSSL installation, or replace ``openssl`` with the **path to the openssl.exe file**.

1. .. _nifi-certificates-41:

	**Generating the administrator's certificate's private key**
	
	Same command as when you created the CA's private key. It will ask you to choose a password. ::

		openssl genrsa -aes128 -out C:\certs\admin.key 2048
		
2. .. _nifi-certificates-42:

	**Generating a certificate sign request**
	
	Like in step 3.2, this command will generate a certificate with a request to sign it. You will be asked to provide the password to the private key you just created. It will then ask you to fill the profile of the certificate, similarly to what you did with the CA.
	
	It is now important to provide the name of the administrator (for example in **Common Name**, or **Email Address**), as it will be used by NiFi to associate this certificate to the admin user account (see :ref:`4.5 <nifi-certificates-45>`) for more information). The other fields are not very meaningful, but again, try to pick something that will help you recognize the certificate.

	Also note that it will ask you for a **challenge password** and an **optional company name**. The challenge password is very rarely used by some CAs when requesting to revoke a certificate. Both fields can safely be left blank. ::

		openssl req -new -key C:\certs\admin.key -out C:\certs\admin.csr
		
3. .. _nifi-certificates-43:

	**Signing the administrator's certificate**
	
	You can now have the CA sign your administrator's certificate. It will ask for the password you created in :ref:`1.1 <nifi-certificates-11>`. ::

		openssl x509 -req -in C:\certs\admin.csr -CA C:\certs\myCA.pem -CAkey C:\certs\myCA.key  -CAcreateserial -out C:\certs\admin.crt -days 730
		
4. .. _nifi-certificates-44:

	**Converting from crt to p12**
	
	This command will convert the signed certificate into **p12** format. It will ask you to provide the password you chose in :ref:`4.1 <nifi-certificates-41>`, and then it will ask you to choose an export password, needed to extract the certificate from the p12 file. ::

		openssl pkcs12 -export -out C:\certs\admin.p12 -inkey C:\certs\admin.key -in C:\certs\admin.crt -certfile C:\certs\myCA.pem -certpbe PBE-SHA1-3DES -name "admin"
		
5. .. _nifi-certificates-45:

	**Configure NiFi to find the administrator's name**
	
	Finally, you have to uncomment two lines from the `nifi.properties` file (inside the `conf` subfolder of your NiFi installation) and give them proper values.
	
	They are regular expressions used by NiFi to find the administrator's name inside the certificate you created in :ref:`4.2 <nifi-certificates-42>`. Configuring these two lines incorrectly can lead to 403 errors.

	The field names are: `EMAILADDRESS` (Email address), `CN` (Common Name), `OU` (Organizational Unit Name), `O` (Organization Name), `L` (Locality Name), `ST` (State or Province Name), `C` (Country Code).

	This particular configuration will take the administrator's name from the `Common Name` field, but you can alter it depending on how you filled the profile during :ref:`4.1 <nifi-certificates-41>`. ::

		nifi.security.identity.mapping.pattern.dn=^(EMAILADDRESS=(.*?), )?CN=(.*?), OU=(.*?), O=(.*?), L=(.*?), ST=(.*?), C=(.*?)$
		nifi.security.identity.mapping.value.dn=$3

Configuration
"""""""""""""

Many of the fields represent NiFi API end-points and have fixed values: although unlikely, there is a chance that they may change in newer versions of NiFi. If you suspect this has happened, you should be able to find the new end-point in the `official documentation <https://nifi.apache.org/docs/nifi-docs/rest-api/index.html>`_.

-	``name``: Name of the component, only needed for display.

-	``componentId``: ID of the component, should be ``nifi``

-	``scope``: Scope of the component, should be ``components/nifi``

-	``implementation``: Full class name of the class implementing the component. The class designed for NiFi is ``it.smartcommunitylab.nificonnector.NiFiConnector``; alternatively, the value ``it.smartcommunitylab.orgmanager.componentsmodel.DefaultComponentImpl`` may be used to disable the NiFi connector.

-	``roles``: Comma-separated list of roles that may be assigned to users via OMC. It should consist of all roles listed in the ``readRoles`` field plus all roles listed in the ``writeRoles`` field.

-	``host``: URI where NiFi is hosted.

-	``listUsersApi``: NiFi API end-point for listing users. Should be ``/nifi-api/tenants/users``, unless a new version of NiFi changes it into something different.

-	``createUserApi``: NiFi API end-point for creating a user. Should be ``/nifi-api/tenants/users``

-	``deleteUserApi``: NiFi API end-point for deleting a user. Should be ``/nifi-api/tenants/users/``

-	``listUserGroupsApi``: NiFi API end-point for listing user groups. Should be ``/nifi-api/tenants/user-groups``

-	``createUserGroupApi``: NiFi API end-point for creating a user group. Should be ``/nifi-api/tenants/user-groups``

-	``updateUserGroupApi``: NiFi API end-point for updating a user group. Should be ``/nifi-api/tenants/user-groups/``

-	``deleteUserGroupApi``: NiFi API end-point for deleting a user group. Should be ``/nifi-api/tenants/user-groups/``

-	``getPolicyApi``: NiFi API end-point to retrieve a policy. Should be ``/nifi-api/policies/``

-	``createPolicyApi``: NiFi API end-point to create a policy. Should be ``/nifi-api/policies``

-	``updatePolicyApi``: NiFi API end-point to update a policy. Should be ``/nifi-api/policies/``

-	``listProcessGroupsApi``: NiFi API end-point to list process groups. Should be ``/nifi-api/process-groups/``

-	``getProcessGroupApi``: NiFi API end-point to retrieve a process group. Should be ``/nifi-api/process-groups/``

-	``createProcessGroupApi``: NiFi API end-point to create a process group. Should be ``/nifi-api/process-groups/``

-	``deleteProcessGroupApi``: NiFi API end-point to delete a process group. Should be ``/nifi-api/process-groups/``

-	``accessApi``: NiFi API end-point to retrieve the status of the current access. Should be ``/nifi-api/access``

-	``keystorePath``: Absolute path to the certificate. Following the example in the `Certificates`_ section, it would have the value ``C:/certs/admin.p12``, determined in :ref:`4.4 <nifi-certificates-44>`.

-	``keystoreType``: Type of the certificate. In the example, it would have the value ``PKCS12``.

-	``keystoreExportPassword``: The password for the certificate. In the example, it would have the value chosen in :ref:`4.4 <nifi-certificates-44>`.

-	``truststorePath``: Absolute path to the truststore. In the example, it would be ``C:/certs/truststore.jks``, determined in :ref:`2.1 <nifi-certificates-21>`.

-	``truststoreType``: Type of the truststore. In the example, it would have the value ``JKS``.

-	``truststorePassword``: Password of the truststore. In the example, it would have the value chosen in :ref:`2.1 <nifi-certificates-21>`.

-	``adminName``: Name of the administrator user.

-	``ownerRole``: Role used by AAC to indicate ownership. Should be ``ROLE_PROVIDER``. Will have both read and write permissions on process groups.

-	``readRoles``: Names of the roles which will have read-only permissions on process groups. While multiple roles may be listed, separated by a comma, they would all be equivalent, so listing 1 role only is advisable. The ``roles`` field should contain all roles listed in this field and all roles listed in the ``writeRoles`` field, or consistency issues may arise.

-	``writeRoles``: Names of roles which will have both read and write permissions (just like the owner role) on process groups. While multiple roles may be listed, separated by a comma, they would all be equivalent, so listing 1 role only is advisable. The ``roles`` field should contain all roles listed in this field and all roles listed in the ``readRoles`` field, or consistency issues may arise.

Running with Docker
-------------------

The server contains some default configuration that, when running with Docker, cannot be changed without recompiling the whole project.

To avoid this, two files are necessary for Docker to override this default configuration.

The **first** one is an *env* file: when Docker runs the container, it will use this file to create several environment variables that the server will read to configure itself.

The *env* file to alter is `/dockerfiles/orgmanager-config.env <https://github.com/smartcommunitylab/AAC-Org/blob/master/dockerfiles/orgmanager-config.env>`_, which contains a sample configuration.

Variables appear with the ``<NAME>:<value>`` format. The uppercase part matches its name as described in the :ref:`setting-up-the-server` section, while on the right is the value to assign.

Make sure the ``OMC_SECURITY_OAUTH2_CLIENTID`` and ``OMC_SECURITY_OAUTH2_CLIENTSECRET`` variables respectively contain the client ID and secret generated by AAC for the server.
In addition, replace *host:port* addresses for the Postgres database and AAC appropriately.

The **second** file will contain the configuration for the components, such as API Manager or Apache NiFi. Unlike the previous file, which creates environment variables for the server to retrieve values from, this one simply replaces a default configuration file.

The file must be in *yml* format and its structure is identical to the default `application-component.yml <https://github.com/smartcommunitylab/AAC-Org/blob/master/server/src/main/resources/application-components.yml>`_ file.

A sample configuration is present in `dockerfiles/application-components.yml <https://github.com/smartcommunitylab/AAC-Org/blob/master/dockerfiles/application-components.yml>`_.

Replace *host:port* values with the addresses of the services.

Once you have configured these two files and Docker is running, open a console and change directory to the root folder (``AAC-Org``) of the project and execute this command to build a Docker image::

  docker build -t orgmanager .

This command will take some time to compile the whole project and will create an image named *orgmanager*. If you wish to name it something else, simply replace ``orgmanager`` with the name you wish to use.

Note that the final dot of the command, separated by a space, is important: without it, an error will be returned.

All that remains is to run the container using this image. The following command will run the server inside a Docker container, mounting the two configuration files described earlier. ::

  docker run --env-file dockerfiles/orgmanager-config.env -v <absolute_path_to_project>/dockerfiles/application-components.yml:/tmp/server/target/config/application-components.yml -p 7878:7979 -t orgmanager

Note that you need to replace ``<absolute_path_to_project>`` with the full path to this project. If you're running it on Windows, the command would look similar to this::

  docker run --env-file dockerfiles/orgmanager-config.env -v //c/Eclipse/Workspace/AAC-Org/dockerfiles/application-components.yml:/tmp/server/target/config/application-components.yml -p 7878:7979 -t orgmanager

If you have configured the NiFi connector to run, you need to provide the `Certificates`_ you created for it::

  docker run --env-file dockerfiles/orgmanager-config.env -v <absolute_path_to_certificates_folder>:/certs -v <absolute_path_to_prject>/dockerfiles/application-components.yml:/tmp/server/target/config/application-components.yml -p 7878:7979 -t orgmanager

The command with the NiFi certificates might look like the following::

  docker run --env-file dockerfiles/orgmanager-config.env -v //c/Eclipse/Workspace/AAC-Org/dockerfiles/certs:/certs -v //c/Eclipse/Workspace/AAC-Org/dockerfiles/application-components.yml:/tmp/server/target/config/application-components.yml -p 7878:7979 -t orgmanager

Client
------

*Angular* is required to run the *Organization Management Console*, the front-end for Org-Manager.

Install `Node.js <https://nodejs.org/en/download/>`_, then open a console and run the commands ``npm install`` to install app dependencies and ``npm install -g @angular/cli`` to install Angular CLI.

Before running the client, some parameters must be configured in the `src/environments/environment.ts <https://github.com/smartcommunitylab/AAC-Org/blob/master/client/src/environments/environment.ts>`_ file:

- ``aacUrl``: AAC address. If you're hosting it on *localhost* through port *8080*, it should be ``http://localhost:8080/aac/``.
- ``aacClientId``: Client ID for Org-Manager on AAC.
- ``redirectUrl``: URL where the client is hosted. Default value is ``http://localhost:4200/``.
- ``scope``: Scopes required to enable authentication via AAC. The value should remain ``profile.basicprofile.me,user.roles.me``.
- ``locUrl``: Root for the server's APIs. If you're hosting Org-Manager on *localhost* through port *7979*, it should be ``http://localhost:7979/api/``.

You can now run the client by opening a console, changing directory to the ``client`` subfolder of the project and executing `ng serve`.
