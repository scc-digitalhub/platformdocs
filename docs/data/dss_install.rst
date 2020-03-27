Data Services Server Installation
===================================

WSO2 Data Services Server is based on Java OSGi technology, which allows components to be dynamically installed, started, stopped, updated, and uninstalled while eliminating component version conflicts.
Given below are the steps you need to follow to build the product from source code:


1. Git clone the product-level repository https://github.com/coinnovationlab/product-dss.git.
2. Use one of the following commands to build your repository and create complete release artifacts of WSO2 DSS.

+---------------------------------------------+---------------------------------------------------------------------------+
| Command                                     | Description                                                               |
+=============================================+===========================================================================+
| mvn clean install                           | The binary and source distributions.                                      |
+---------------------------------------------+---------------------------------------------------------------------------+
| mvn clean install -Dmaven.test.skip=true    | The binary and source distributions, without running any of the unit tests|
+---------------------------------------------+---------------------------------------------------------------------------+
| mvn clean install -Dmaven.test.skip=true -o | The binary and source distributions,                                      |
|                                             | without running any of the unit tests, in offline mode.                   |
|                                             | This can be done only if you have already built the source at least once. |
+---------------------------------------------+---------------------------------------------------------------------------+

.. warning::

	**Certificate validation errors**
	
	Product DSS makes use of an old version of the Eclipse Equinox framework that is signed with a certificate that is not trusted by recent versions of Java.
	If the Maven build fails with the error **One or more certificates rejected. Cannot proceed with installation**, that means you have to import the certificate to the Java certificate store.::
	
		jar -xvf $MAVEN_HOME/repository/org/eclipse/tycho/tycho-p2-runtime/0.13.0/eclipse/plugins/org.eclipse.equinox.launcher_1.2.0.v20110725-1610.jar
		cd META-INF
		openssl pkcs7 -in ECLIPSEF.RSA -print_certs -inform DER -out EEF.cer`` This certificate can be found also in the root of Product DSS repo.
		keytool -import -file ECLIPSEF.cer -alias eclipse -keystore "$JAVA_HOME/jre/lib/security/cacerts"-storepass changeit -noprompt

3. You can find the new binary pack (ZIP file) in the ``<PRODUCT_HOME>/modules/distribution/target`` directory 
4. Unzip this file in order to start the configuration of the environment 
5. Download the JDBC driver for MySQL from https://dev.mysql.com/downloads/connector/j and copy it to ``<PRODUCT_HOME>/repository/components/lib`` directory.
6. Download ther PostgreSQL driver from  https://jdbc.postgresql.org/download.html  and copy it to ``<PRODUCT_HOME>/repository/components/lib`` directory.
7. Open ``./repository/conf/carbon.xml`` and set 2 the value of Offset ``<Offset>0</Offset>``  (optional if you have several WSO2 products installed)
8. Changing the default database (using mysql instead of h2 default db for WSO2 products)::

		mysql -u root -p
		create database dss
		GRANT ALL ON dss.* TO dssadmin@localhost IDENTIFIED BY "dssadmin";
		mysql -u dssadmin -p dss < '<PRODUCT_HOME>/dbscripts/mysql.sql';  OR mysql5.7.sql
	

9. Open the ``<PRODUCT_HOME>/repository/conf/datasources/master-datasources.xml`` file and locate the ``<datasource>`` configuration element.
You simply have to update the URL pointing to your MySQL database, the username, and password required to access the database and the MySQL driver details 
See the example below:::

	<datasource>
            <name>WSO2_CARBON_DB</name>
            <description>The datasource used for registry and user manager</description>
            <jndiConfig>
                <name>jdbc/WSO2CarbonDB</name>
            </jndiConfig>
            <definition type="RDBMS">
                <configuration>
                    <url>jdbc:mysql://localhost:3306/$dbname?autoReconnect=true</url>
                    <username>$user</username>
                    <password>$password</password>
                    <driverClassName>com.mysql.jdbc.Driver</driverClassName>
                    <maxActive>50</maxActive>
                    <maxWait>60000</maxWait>
                    <testOnBorrow>true</testOnBorrow>
                    <validationQuery>SELECT 1</validationQuery>
                    <validationInterval>30000</validationInterval>
                    <defaultAutoCommit>true</defaultAutoCommit>
                </configuration>
            </definition>
        </datasource>
 
10. Run the product in the following modes:

* **Default**  

	To start the server, you run the script wso2server.bat (on Windows) or wso2server.sh (on Linux/Solaris) from the bin folder.::

		On Windows:          <PRODUCT_HOME>\bin\wso2server.bat --run
		On Linux/Solaris:    sh <PRODUCT_HOME>/bin/wso2server.sh

* **Custom** (Including Security Integrations with AAC Component)  
	* In the file repository/conf/security/authenticators.xml put the following xml configuration ::
	
		<!-- Example AAC OAUTH2 provider →
		<Authenticator name="OAUTH2SSOAuthenticator" disabled="false">
		  <Priority>3</Priority>
		  <Config>
		     <Parameter name="OauthProviderName">AAC</Parameter>
		     <Parameter name="LoginPage">/carbon/admin/login.jsp</Parameter>
		         <Parameter name="ServiceProviderID">carbonServer</Parameter>
		     <Parameter name="IdentityProviderSSOServiceURL">http://localhost:8080/aac</Parameter>
		     <Parameter name="LandingPage">https://mydomain.com/dss_proxy_context_path/carbon/oauth2-sso-acs/custom_login.jsp</Parameter>
		     <Parameter name="RedirectURL">https://mydomain.com/dss_proxy_context_path/oauth2_acs</Parameter>
		     <Parameter name="UserProvisioningEnabled">true</Parameter>
		     <Parameter name="TenantProvisioningEnabled">true</Parameter>
		     <Parameter name="TenantDefault">testdomain.com</Parameter>
		     <Parameter name="ClientID">YOUR_AAC_CLIENT_ID</Parameter>
		     <Parameter name="ClientSecret">YOUR_AAC_CLIENT_SECRET</Parameter>
		     <Parameter name="AuthorizationURL">http://localhost:8080/aac/oauth/authorize</Parameter>
		     <Parameter name="TokenURL">http://localhost:8080/aac/oauth/token</Parameter>
		     <Parameter name="CheckTokenEndpointUrl">http://localhost:8080/aac/oauth/introspect</Parameter>
		     <Parameter name="APIUserInfoURL">http://localhost:8080/aac/basicprofile/me</Parameter>
		     <Parameter name="APIRoleInfoURL">http://localhost:8080/aac/userroles/me</Parameter>
		     <Parameter name="GetRolesOfTokenURL">http://localhost:8080/aac/userroles/token</Parameter>
		     <Parameter name="ApiKeyCheckURL">http://localhost:8080/aac/apikeycheck</Parameter>
		     <Parameter name="MaxExpireSecToken">86400</Parameter>	     
		     <Parameter name="ScopesListUserInfo">profile.basicprofile.me profile.accountprofile.me user.roles.me user.roles.read</Parameter>
		     <Parameter name="UserNameField">username</Parameter>
		     <Parameter name="RoleContext">YOUR_ROLE_CONTEXT</Parameter>
		     <Parameter name="SelectTenantURL">https://mydomain.com/dss_proxy_context_path/carbon/oauth2-sso-acs/select_tenant.jsp</Parameter>
		         <Parameter name="TenantSelectedURL">https://mydomain.com/dss_proxy_context_path/forwardtenant</Parameter>
		     <Parameter name="OauthProviderName">AAC</Parameter>
		     <Parameter name="SecurityFilterClass">org.wso2.carbon.dataservices.core.security.filter.ServicesSecurityFilter</Parameter>
		  </Config>
		</Authenticator>
		
	You can find a description of each of these parameters :ref:`here <desc-custom-param>`:

	* Edit the file repository/conf/tomcat/web.xml by adding the cookie-config tag:::
	
		<session-config>
			<session-timeout>30</session-timeout>
			<cookie-config>
			         <name>JSESSIONID_DSS</name>
			</cookie-config>
		</session-config>
		
10. You can populate the environment with the default samples inside the product by running:  ::

	cd <PRODUCT_HOME/> samples; ant clean; ant

11. Regarding the OAUTH2 provider configuration you need to put the redirect link to: ``https://mydomain.com/dss_proxy_context_path/oauth2_acs`` 

.. _desc-custom-param:

Description of Custom Authentication Config Parameters
------------------------------------------------------

In order to define the necessary parameters of the custom authentication of DSS using OAUTH2 protocol we need to add the proper values inside the file ``<PRODUCT_HOME>/repository/conf/security/authenticators.xml`` ::

	<Parameter name="PROPERTY-NAME">PROPERTY-VALUE</Parameter>
	
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| PROPERTY-NAME                 | DESCRIPTION                                                                                                                           |
+===============================+=======================================================================================================================================+
| OauthProviderName             | The name of the provider that will appear on the login button                                                                         |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| LoginPage                     | The default login page                                                                                                                |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| IdentityProviderSSOServiceURL | The URL of the OAUTH2 provider                                                                                                        |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| LandingPage                   | The custom login page that contains the ‘Login With’ button.If empty it will directly  redirect to the IdentityProviderSSOServiceURL  |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| RedirectURL                   | The callback page where the service redirects the user-agent after an authorization code is granted                                   |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| UserProvisioningEnabled       | Boolean value to create or not the user on the fly if it doesn’t exist                                                                |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| TenantDefault                 | The default tenant to whom attach the users if the tenant doesn’t exist                                                               |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| TenantProvisioningEnabled     | Boolean value to create or not the tenant on the fly if it doesn’t exist.                                                             |
|                               | If the value is false then the user will belong to the TenantDefault                                                                  |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| ProvisioningDefaultRole       | The default roles to which will belong the users created on the fly.                                                                  |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| ClientID                      | The Clientid value of the Client Application                                                                                          |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| ClientSecret                  | The ClientSecret value of the Client Application                                                                                      |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| AuthorizationURL              | The API authorization endpoint                                                                                                        |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| TokenURL                      | The API token endpoint: the application requests an access token from the API,                                                        |
|                               | by passing the authorization code along with the authentication details, including the client secret, to the API token endpoint.      |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| CheckTokenEndpointUrl         | The endpoint to get the information regarding the generated token.                                                                    |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| GetRolesOfTokenURL            | The endpoint to check the validity of the token provided in the odata/rest requests.                                                  |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| ApiKeyCheckURL                | The endpoint to check the validity of the apiKey provided in the odata/rest requests.                                                 |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| MaxExpireSecToken             | The period of expiration of the  client_credentials token if this is an unlimited generated token.                                    |
|                               | The token is being saved in Cookies and can be regenerated according to the expire_in parameter if this is >0 or if it is unlimited   |
|                               | it will be regenerated after $MaxExpireSecToken seconds.                                                                              |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| APIUserInfoURL                | The URL to call the API that gives information about the user profile                                                                 |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| APIRoleInfoURL                | The URL to call the API that gives information about the roles of the user                                                            |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| ScopesListUserInfo            | Specifies the level of access that the application is requesting                                                                      |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| UserNameField                 | The name of the field that contains the username from the APIUserInfoURL request                                                      |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| RoleContext                   | The context that the roles must have in order to be considered inside the DSS installation (ex: components/dss.super)                 |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| SelectTenantURL               | The redirection page to select the desired tenant to whom belongs the user                                                            |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| TenantSelectedURL             | The servlet name to select the desired tenant to whom belongs the user                                                                |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+
| SecurityFilterClass           | The name of the class that implements the security filter of the odata/rest requests                                                  |
+-------------------------------+---------------------------------------------------------------------------------------------------------------------------------------+

