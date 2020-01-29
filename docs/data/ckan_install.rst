CKAN Integration
===================================

Different modes for the installation of CKAN platform is described in the official documentation (https://docs.ckan.org/en/latest/maintaining/installing/index.html).

The integration within the platform relies upon the organization and user model of the CKAN platform and the multi-tenancy model
supported by the DigitalHub platform. To guarantee the alignment between the two models it is necessary to

- align the tenants (organizations) and the corresponding users
- align the authentication mechanism to support the appropriate authorization models ansd Single Sign-On.

More specifically, multitenancy in CKAN is based on the concept of Organization. The datasets published are associated to a specific organization
and managed by that organization. The model of roles made available by AAC and managed by the Organization Manager allows for modelling these concepts.

To allow for authentication and authorization using AAC, CKAN provides extension plugins, in particular, OAuth2.0 authentication plugin. The existing
plugins, however, do not allow for a sophisticated user and organization management, and have been extended within the DigitalHub. In partticular,
the new CKAN extension allows for

- authentication of the users based on AAC OAuth2.0 integration
- understanding the roles and organizations of the authenticated user
- registering dynamically the organization and the users within CKAN

CKAN DigitalHub OAuth2.0 Plugin Installation
--------------------------------------------

The plugin is a Python module that is installed within the CKAN platform following the standard CKAN process.

To install the plugin, enter your virtualenv and install the package using pip as follows::

		pip install git+https://github.com/scc-digitalhub/ckanext-oauth2.git


Add the following to your CKAN .ini (generally /etc/ckan/default/production.ini) file:::

		ckan.plugins = oauth2 <other-plugins>

		## OAuth2 configuration

		ckan.oauth2.register_url = https://YOUR_OAUTH_SERVICE/users/sign_up
		ckan.oauth2.reset_url = https://YOUR_OAUTH_SERVICE/users/password/new
		ckan.oauth2.edit_url = https://YOUR_OAUTH_SERVICE/settings
		ckan.oauth2.authorization_endpoint = https://YOUR_OAUTH_SERVICE/authorize
		ckan.oauth2.token_endpoint = https://YOUR_OAUTH_SERVICE/token
		ckan.oauth2.profile_api_url = https://YOUR_OAUTH_SERVICE/user
		ckan.oauth2.client_id = YOUR_CLIENT_ID
		ckan.oauth2.client_secret = YOUR_CLIENT_SECRET
		ckan.oauth2.scope = profile other.scope
		ckan.oauth2.rememberer_name = auth_tkt
		ckan.oauth2.profile_api_user_field = JSON_FIELD_TO_FIND_THE_USER_IDENTIFIER
		ckan.oauth2.profile_api_fullname_field = JSON_FIELD_TO_FIND_THE_USER_FULLNAME
		ckan.oauth2.profile_api_mail_field = JSON_FIELD_TO_FIND_THE_USER_MAIL
		ckan.oauth2.authorization_header = OAUTH2_HEADER
		ckan.oauth2.profile_api_groupmembership_field = JSON_FIELD_TO_FIND_USER_ROLES_IN_ORGANIZATIONS
		ckan.oauth2.sysadmin_group_name = NAME_OF_THE_SYSADMIN_GROUP


The profile_api_groupmembership_field should be a list of strings or JSON objects with the org field representing 
the organization, and role representing the role. The first case is for compatibility wirth the original plugin. 
May contain the sysadmin_group_name value to map the user into the CKAN sysadmin role.

The role value should be one of admin, editor, or member. If the value is different, it will be cast to member.

You can also use environment variables to configure this plugin, the name of the environment variables are:::

		CKAN_OAUTH2_REGISTER_URL
		CKAN_OAUTH2_RESET_URL
		CKAN_OAUTH2_EDIT_URL
		CKAN_OAUTH2_AUTHORIZATION_ENDPOINT
		CKAN_OAUTH2_TOKEN_ENDPOINT
		CKAN_OAUTH2_PROFILE_API_URL
		CKAN_OAUTH2_CLIENT_ID
		CKAN_OAUTH2_CLIENT_SECRET
		CKAN_OAUTH2_SCOPE
		CKAN_OAUTH2_REMEMBERER_NAME
		CKAN_OAUTH2_PROFILE_API_USER_FIELD
		CKAN_OAUTH2_PROFILE_API_FULLNAME_FIELD
		CKAN_OAUTH2_PROFILE_API_MAIL_FIELD
		CKAN_OAUTH2_AUTHORIZATION_HEADER
		CKAN_OAUTH2_PROFILE_API_GROUPMEMBERSHIP_FIELD
		CKAN_OAUTH2_SYSADMIN_GROUP_NAME


The callback URL that you should set on your OAuth 2.0 is: https://YOUR_CKAN_INSTANCE/oauth2/callback, 
replacing YOUR_CKAN_INSTANCE by the machine and port where your CKAN instance is running.