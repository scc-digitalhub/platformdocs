API Manager Installation
=========================

WSO2 API Manager is an open source approach that addresses full **API lifecycle management**, **monetization**, and **policy enforcement**. 


WSO2 API Manager is a unique open approach to full lifecycle API development, integration and management. 
As part of the larger **Digital Hub Platform**, it is a central component used to deploy and manage **API-driven ecosystems**. 
It’s hybrid integration capabilities further simplify projects that span traditional as well as microservice environments. 

| It allows extensibility and customization, and ensures freedom from lock-in.
| The customized version of APIM inside Digital Hub can be set up following these steps:


Installation Requirements
--------------------------

* :doc:`aac`
* MySQL 5.5+

1. Download the API Manager 2.6.0 
-----------------------------------

Get the binary version from `here <https://wso2.com/api-management/install/>`_

2. Database SetUp 
------------------

Replace the default H2 db with mysql
	* using mysql as database ::
	
		  - mysql -u root -p
		  - create database regdb character set latin1 (*); 
		  - GRANT ALL ON regdb.* TO regadmin@localhost IDENTIFIED BY "regadmin";
		  - FLUSH PRIVILEGES;
		  - quit;

	.. note::
		character set is for windows only
	
	* copy mysql connector (i.e. mysql-connector-java-__-bin.jar) into repository/components/lib, downloadable from https://dev.mysql.com/downloads/connector/j
	* edit /repository/conf/datasources/master-datasources.xml
	
		* for the datasources **WSO2_CARBON_DB** and **WSO2AM_DB**, make these changes: ::

		  <url>jdbc:mysql://localhost:3306/regdb</url>
		  <username>regadmin</username>
		  <password>regadmin</password>
		  <driverClassName>com.mysql.jdbc.Driver</driverClassName> 

		* launch the following scripts (for MySQL 5.7 use versions __.5.7.sql)::
		
			  mysql -u regadmin -p -Dregdb < dbscripts/mysql.sql
			  mysql -u regadmin -p -Dregdb < dbscripts/apimgt/mysql.sql
			  
3. AAC integration :doc:`aac`
-----------------------------

Clone the project that contains the AAC connector and WSO2 custom theme: https://github.com/smartcommunitylab/API-Manager

3.1. APIM-AAC connector
^^^^^^^^^^^^^^^^^^^^^^^^
* build wso2aac.client project with Maven.
* copy ``wso2aac.client-1.0.jar`` from the project API-Manager/wso2aac.client to the WSO2 directory ``repository/components/lib``

3.2. APIM configurations
^^^^^^^^^^^^^^^^^^^^^^^^^

* In ``repository/conf/api-manager.xml``, change **APIKeyManager** and set **ConsumerSecret** with the value found in AAC for the client with clientId ``API_MGT_CLIENT_ID`` ::
	
	<APIKeyManager>
	  	<KeyManagerClientImpl>it.smartcommunitylab.wso2aac.keymanager.AACOAuthClient</KeyManagerClientImpl>
	  	<Configuration>
	  		<RegistrationEndpoint>http://localhost:8080/aac</RegistrationEndpoint>
	  		<ConsumerKey>API_MGT_CLIENT_ID</ConsumerKey>
	  		<ConsumerSecret></ConsumerSecret>
	  		<Username>admin</Username>
	  		<Password>admin</Password>
	  		<VALIDITY_PERIOD>3600</VALIDITY_PERIOD>
	  		<ServerURL>https://localhost:9443/services/</ServerURL>
	  		<RevokeURL>https://localhost:8243/revoke</RevokeURL>
	  		<TokenURL>http://localhost:8080/aac/oauth/token</TokenURL>			
	  	</Configuration>
	  </APIKeyManager>

* In ``repository/conf/api-manager.xml``, uncomment **RemoveOAuthHeadersFromOutMessage** and set it to false ::

	<RemoveOAuthHeadersFromOutMessage>false</RemoveOAuthHeadersFromOutMessage>

* In ``repository/conf/identity/identity.xml``  
	* find ::
	
		<OAuthScopeValidator class="org.wso2.carbon.identity.oauth2.validators.JDBCScopeValidator"/>

	* and replace with ::

		<OAuthScopeValidator class="it.smartcommunitylab.wso2aac.keymanager.CustomJDBCScopeValidator"/>

	* and in **SupportedGrantTypes** section disable **saml2-bearer** and **ntlm** and add::
	
		<SupportedGrantType>
		        <GrantTypeName>native</GrantTypeName>
		        <GrantTypeHandlerImplClass>it.smartcommunitylab.wso2aac.grants.NativeGrantType</GrantTypeHandlerImplClass>
		        <GrantTypeValidatorImplClass>it.smartcommunitylab.wso2aac.grants.NativeGrantValidator</GrantTypeValidatorImplClass>
		    </SupportedGrantType>

* In ``repository/conf/carbon.xml``, enable email username ::

	<EnableEmailUserName>true</EnableEmailUserName>

* In ``repository/conf/user-mgt.xml``, add the following property to ``<UserStoreManager>`` ::

	<Property name="UsernameWithEmailJavaScriptRegEx">^[\S]{3,30}$</Property>
	
3.3. APIM theming
^^^^^^^^^^^^^^^^^^
* copy the contents of project API-Manager/wso2.custom into the WSO2 directory

4. Keystore configuration
-----------------------------

Import and add WSO2 certificate to the default keystore.

Linux
	* ``sudo rm`` -f cert.pem && sudo echo -n | ``openssl`` s_client -connect localhost:9443 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > ./cert.pem
	* ``sudo keytool`` -import -trustcacerts -file cert.pem -alias root -keystore JAVA_HOME/jre/lib/security/cacerts

Windows
	* ``keytool`` -importkeystore -srckeystore <<wso2_root>>/repository/resources/security/wso2carbon.jks -destkeystore wso2.p12 -srcstoretype jks -deststoretype pkcs12 -alias wso2carbon -destkeypass 123456 
		.. note::
			use a temporary password (such '123456') for destination keystore and PEM passphrase, empty password for origin and "wso2carbon" for wso2carbon password.
	
	* ``openssl`` pkcs12 -in wso2.p12 -out wso2.pem
	
	* Edit wso2.pem and keep only the part between -----BEGIN CERTIFICATE----- and -----END CERTIFICATE-----
		
		* ``keytool`` -import -trustcacerts -file wso2.pem -alias root -keystore "%JAVA_HOME%/jre/lib/security/cacerts"

.. note::
	java cacerts default password is "changeit"
.. warning::
	keytool is bugged in most java 8 versions, returning a java.util.IllegalFormatConversionException: d != java.lang.String

5. Proxy server configuration (Apache)
---------------------------------------
5.1. Configure proxy for apps
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* Configure proxy publisher and subscriber apps: repository/deployment/server/jaggeryapps/publisher/site/conf/site.json (same for store):

	* context: /publisher
	* host: < mydomain.com >, e.g., am-dev.smartcommunitylab.it
* Configure management console: repository/conf/carbon.xml

	* <HostName>am-dev.smartcommunitylab.it</HostName>
	* <MgtHostName>am-dev.smartcommunitylab.it</MgtHostName>
* Configure WSO2 Tomcat reverse proxy: repository/conf/tomcat/catalina-server.xml

	* Add parameters to 9443 connector: ::
	
		proxyPort="443"
		proxyName="am-dev.smartcommunitylab.it"
		
	* Configure Apache Virtual Host: 
	
		* port 80: redirect port 80 to 443
		* port 443: ProxtPath and ProxyPathReverse / to ip:9443/
		
5.2. API Gateway
^^^^^^^^^^^^^^^^^^

* Configure Gateway endpoint: repository/conf/api-manager.xml::

	<GatewayEndpoint>http://api-dev.smartcommunitylab.it,https://api-dev.smartcommunitylab.it</GatewayEndpoint>
	
* Configure axis2 transport Ins (http and https): add the following parameters::

	<parameter name="proxyPort" locked="false">80</parameter>
	<parameter name="hostname" locked="false">api-dev.smartcommunitylab.it</parameter>
	
* Configure Apache Virtual Host:

	* port 80: ProxtPath and ProxyPathReverse / to ip:8280/
	* port 443: ProxtPath and ProxyPathReverse / to ip:8243/


6. API-M Custom User Store Manager
------------------------------------

In order to provide the necessary infrastructure for allowing API-M to interact with `Oganization Manager <https://github.com/smartcommunitylab/AAC-Org/tree/master/connectors/orgmanager-wso2connector>`_ it is important to deploy the two new bundles that extend the existing `UserStoreManagerService admin <https://github.com/wso2-extensions/identity-user-ws/blob/master/components/org.wso2.carbon.um.ws.service/src/main/java/org/wso2/carbon/um/ws/service/UserStoreManagerService.java>`_.
This extension is done in order to permit the admin account to create,update,delete users and assign/revoke roles within specific tenants.

The configuration steps are the following:

* build `orgmanager-wso2connector <https://github.com/smartcommunitylab/AAC-Org/tree/master/connectors/orgmanager-wso2connector>`_ project with Maven.

* copy **apim.custom.user.store-0.0.1.jar** from the project ``orgmanager-wso2connector/apim.custom.user.store`` to the WSO2 directory repository/components/dropins

* copy **apim.custom.user.store.stub-0.0.1.jar** from the project ``orgmanager-wso2connector/apim.custom.user.store.stub`` to the WSO2 directory repository/components/dropins

As a result the new admin stub can be accessible from the following endpoint: https://$APIM_URL/services/CustomUserStoreManagerService


Configuration using docker
--------------------------

| Docker resources for each of the component of the platform, including APIM, help you build generic Docker images for deploying the corresponding servers in containerized environments. 
| Configurations, custom JDBC drivers other than the default MySQL JDBC driver provided, extensions and other deployable artifacts are designed to be provided via volume mounts to the containers spawned.
| Refer to the `APIM's repository docker files <https://github.com/scc-digitalhub/API-Manager/tree/master/dockerfiles/apim>`_.
| Set up the parameters in the apim.env file according to the explanation provided in the table below:

+---------------------+--------------------------+-------------------------------------------------------------------+
| Property            |  Default                 |  Description                                                      |
+=====================+==========================+===================================================================+
| APIM_USER           |  ``admin``               |  The name of the admin user                                       |
+---------------------+--------------------------+-------------------------------------------------------------------+
| APIM_PASS           |  ``admin``               |  The password of the admin user                                   |
+---------------------+--------------------------+-------------------------------------------------------------------+
| APIM_HOSTNAME       |  ``api-manager``         |  The name of the running container of apim                        |
+---------------------+--------------------------+-------------------------------------------------------------------+
| APIM_REVERSEPROXY   |  ``api.platform.local``  |  The name of reverse proxy in the nginx config                    |
+---------------------+--------------------------+-------------------------------------------------------------------+
| APIM_GATEWAYENDPOINT|  ``gw.platform.local``   |  The value of gateway                                             | 
+---------------------+--------------------------+-------------------------------------------------------------------+
| ANALYTICS_HOSTNAME  |  ``am-analytics``        |  The name of the running container of apim analytics              |
+---------------------+--------------------------+-------------------------------------------------------------------+
| AAC_HOSTNAME        |  ``aac``                 |  The name of the container running AAC                            |
+---------------------+--------------------------+-------------------------------------------------------------------+
| AAC_CONSUMERKEY     |  ``f04ca519XXXX``        |  The value of consumer key from AAC console for APIM Client App   |
+---------------------+--------------------------+-------------------------------------------------------------------+
| AAC_CONSUMERSECRET  |  ``e181bf39XXXX``        |  The value of consumer secret from AAC console for APIM Client App|
+---------------------+--------------------------+-------------------------------------------------------------------+
| AAC_REVERSEPROXY    |  ``aac.platform.local``  |  The reverse proxy value for AAC                                  | 
+---------------------+--------------------------+-------------------------------------------------------------------+
| APIM_MYSQL_HOSTNAME |  ``mysql``               |  The name of the running container of mysql instance              |
+---------------------+--------------------------+-------------------------------------------------------------------+
| APIM_MYSQL_USER     |  ``wso2carbon``          |  MySQL db username                                                |
+---------------------+--------------------------+-------------------------------------------------------------------+
| APIM_MYSQL_PASS     |  ``wso2carbon``          |  MySQL db password                                                |
+---------------------+--------------------------+-------------------------------------------------------------------+

Keep in mind that, if you need to restart the container, you should *remove* it via ``docker rm -f container_name_or_id`` and then run it again. Stopping and restarting the container without removing it will result in an error.
