Deployment Guidelines in Production
====================================

1. Re-generate the keystore:
----------------------------------------------------------------------------------
These commands allow you to generate a new Java Keytool keystore file, create a CSR, and import certificates.
Execute the commands inside the folder <PRODUCT_HOME>/repository/resources/security.

* Generate a Java keystore and key pair
	``keytool -genkey``-alias mydomain -keyalg RSA -keystore keystore.jks -keysize 2048 -validity 3650 -dname "CN=mydomain,OU=test,O=test,L=test,S=test,C=TS"  -storepass mypassword -keypass mypassword
* Generate a certificate signing request (CSR) for an existing Java keystore
	``keytool -certreq``-alias mydomain -keystore keystore.jks -file mydomain.csr
* Import a root or intermediate CA certificate to an existing Java keystore
	``keytool -import``-trustcacerts -alias root -file Thawte.crt -keystore keystore.jks
* Import a signed primary certificate to an existing Java keystore
	``keytool -import``-trustcacerts -alias mydomain -file mydomain.crt -keystore keystore.jks
* Generate a keystore and self-signed certificate
	``keytool -genkey``-keyalg RSA -alias selfsigned -keystore keystore.jks -storepass password -validity 360 -keysize 2048
* Change a Java keystore password
	``keytool -storepasswd``-new new_storepass -keystore keystore.jks
* Check a particular keystore entry using an alias
	``keytool -list``-v -keystore keystore.jks -alias mydomain


1.1 Example for generating a new keystore:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* ``keytool -genkey`` -alias mydomain -keyalg RSA -keystore wso2carbonDev.jks -keysize 2048 -validity 3650 -dname "CN=mydomain.com,OU=test,O=test,L=test,S=test,C=TS"  -storepass mypassword -keypass mypassword
* ``keytool -export`` -alias mydomain.com -file pubkey.cer -keystore wso2carbonDev.jks -storepass mypassword -noprompt
* ``keytool -import`` -trustcacerts -alias mydomain.com -file pubkey.cer -keystore client-truststoreDev.jks -storepass mypassword -noprompt 

1.2 Example for importing a new certificate:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
* ``keytool -importcert`` -file $somepath/someserver.com -keystore client-truststore.jks -alias "SomeServer"

3. Update the following files with the new keyStore name and domain alias:
---------------------------------------------------------------------------

	* <PRODUCT_HOME>/repository/conf/carbon.xml
	* <PRODUCT_HOME>/repository/conf/tomcat/catalina-server.xml
	* <PRODUCT_HOME>/repository/conf/security/secret-conf.properties
	* <PRODUCT_HOME>/repository/conf/sec.policy


3. Setting Up Secure Vault Configuration (OPTIONAL)
----------------------------------------------------

Encrypt sensitive data in configuration files stored in file system 

	1. Locate ``<PRODUCT_HOME>/repository/conf/security/cipher-text.properties`` file.  This file contains the alias names and the corresponding plain text password in square brackets.
	2. Locate ``<PRODUCT_HOME>/bin/ciphertool.sh`` and run   ``ciphertool.sh -Dconfigure``
	3. Edit ``<PRODUCT_HOME>/repository/conf/security/secret-conf.properties``:
	       keystore.identity.alias=mydomain.com

This security configuration will require to restart DSS providing also the password of the keystore.
It is important to mention the fact that when changing the admin password you have to set the new value in the file ``<PRODUCT_HOME>/repository/conf/user-mgt.xml`` and using ciphertool it will be part of the sensitive data that are going to be encrypted.

4. Adding a Custom Proxy Path
-------------------------------
4.1. Install and configure a reverse proxy
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Example - Apache virtual host configuration ::

      <VirtualHost *:443>
      ServerAdmin webmaster@localhost
      ServerName mydomain.com
      ServerAlias mydomain.com
      ProxyRequests Off
      <Directory "/">
           RewriteEngine On
           RewriteRule csrfPrevention.js$  https://innovation.deda.com/dss/carbon/admin/js/csrfPrevention.js [P]
      </Directory>

      <Proxy>
         Order deny,allow
         Allow from all
      </Proxy>
      SSLEngine On
      SSLProxyEngine On
      SSLCertificateFile /etc/apache2/ssl/server.crt
      SSLCertificateKeyFile /etc/apache2/ssl/server.key
      SSLCACertificateFile /etc/apache2/ssl/ca.crt
      ProxyPass /dss https://mydomain.com:9444/
      ProxyPassReverse /dss https://mydomain.com:9444/
      </VirtualHost>

4.2. Configure DSS with proxy context path
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

4.2.1 Edit <PRODUCT_HOME>/repository/conf/carbon.xml
""""""""""""""""""""""""""""""""""""""""""""""""""""""

      * Change ``<HostName>mydomain.com</HostName>`` 
      * Change ``<MgtHostName>mydomain.com</MgtHostName>``
      * Change ``<KeyAlias>mydomain.com</KeyAlias>``
      * Comment the value of ServerURL and uncomment the following: ``<ServerURL>https://mydomain.com:${carbon.management.port}${carbon.context}/services/</ServerURL>``
      * Uncomment ``<ProxyContextPath>dss</ProxyContextPath> <MgtProxyContextPath>dss</MgtProxyContextPath>``
      
4.2.2 Edit <PRODUCT_HOME>/repository/conf/tomcat/catalina-server.xml
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
Locate HTTPS connector and set  ::

	proxyPort="443" proxyName="mydomain.com/dss"
	
4.2.3 Edit <PRODUCT_HOME>/repository/conf/security/Owasp.CsrfGuard.Carbon.properties 
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
.. code-block:: RST

	org.owasp.csrfguard.UnprotectedMethods=GET,POST   # this is to support oauth2 POST servlet (/forwardmultitenant)
	
4.2.4 Remember to update the file authenticators.xml according to the new proxy config:
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Example: ::

	<Parameter name="LandingPage">https://mydomain.com/dss/carbon/oauth2-sso-acs/custom_login.jsp</Parameter>
