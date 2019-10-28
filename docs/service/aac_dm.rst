
Data Model
-----------------

Users
^^^^^^^^^^
In AAC the user management may be federated with the other Identity Providers. Apart from the built-in user management,
where the user is identified by the registration email and authenticated by password, it is possible to authenticate with
external standard-based IdPs, e.g., OAuth2.0, SAML, etc. 

Each user is, therefore, associated with the account type used at the authentication. The users managed internally, have
``internal`` account type, the users authenticated with Google, have ``google`` type, etc. The mechanism for defining different
IdP depends on the specific protocol; all the supported IdPs (authentication authorities) should be declared in
authority mapping file (``it/smartcommunitylab/aac/manager/authorities.xml``), where for each IdP it is necessary to specify

* identifying attributes (used to uniquely identify the user in this IdP)  
* list of supported attributes specific to IdP. Some of the attributes may be annotated in order to extract common values (e.g., name and surname)
* mapping between the IdPs to automatically map the same user across IdPs using common attribute values (e.g., internal email and Google email).
 
Common user information is captured by the basic user profile, while account information associated with the user
is represented with the account profile data (see., Profile API). The standard ways to capture these attributes is represented
with the OpenID Connect Claims extracted from the user profile, account, and role information.  

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
    
In AAC the roles are contextualized in order to support various multitenancy scenarios. The general idea is 
that the roles are associated to the role *spaces* uniquely identified by their namespace and managed
by the space *owners*. A user may have different roles in different spaces and the authorization
control for individual organizations, components, and deployment may be performed within the corresponding space. 
More specifically, each role is represented as a tuple with

* *context*, defining the "parent" space of roles (if any)
* *space*, defining the specific role space value. Together with the context form the unique role namespace 
* *role*, defining the specific role value for the space.

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

User Claims and Claim Management
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Both for the OpenID connect User Info endpoint and for the JWT access tokens, AAC supports a set of claims representing the user
data. The set of returned claims depend on the scopes requested by the client app by default or during the authorization phase. 
To obtain the access to these claims, the user will be requested an explicit approval.

AAC manages the following standard OIDC claims:

* sub (scopes ``openid``, ``profile``, ``profile.basicprofile.me``)
* username (scopes ``openid``, ``profile``, ``profile.basicprofile.me``)
* preferred_username (scopes ``openid``, ``profile``, ``profile.basicprofile.me``)
* given_name (scopes ``openid``, ``profile``, ``profile.basicprofile.me``)
* family_name (scopes ``openid``, ``profile``, ``profile.basicprofile.me``)
* email (scopes ``openid``, ``profile``, ``email``, ``profile.basicprofile.me``)

Additionally, AAC supports the following custom claims 

* ``accounts``: represent the associated user accounts (requires ``profile.accountprofile.me`` scope).
* ``authorities``: list of the roles associated to the user (requires ``user.roles.me`` scope).
* ``groups``: list of role contexts the user has a role (requires ``user.roles.me`` scope)
* ``resource_access``: roles associated to various role contexts (requires ``user.roles.me`` scope).

** Claim Customization **

When a client app requires specific custom claims to be presented, it is possible for the client app to customize the representation of the user claims (but not the predefined ones
like ``sub``, ``aud``, etc.). The mapping is defined in the client app configuration **Roles&Claims** section
as a JavaScript function that should provide the customized set of claims given the pre-defined ones.

In this way, a client app may map predefined claim values to the expected ones; reduce the number of claims or add new ones. 

** Roles Disambiguation **

Frequently, different multitenant application does not allow the user to belong to more than one tenant. AAC, however, does not
have this restriction and the user may have roles in different contexts associated to that application. To avoid the customization of these applications to deal with the users associated to multiple tenants, AAC allows for the role
disambiguation during the authorization step. That is, after authenticating the user, AAC will ask (together with the request for the scope approval) the user to select a single tenant for the required role context. 

To configure this behavior, the client app should list the contexts, for which the user should be asked to select (**Roles&Claims** configuration of the client app). For example, if the client app is associated with the context ``components/grafana``, the context should be specified in the **Unique Role Spaces** list. During the authorization request, if the user is associated with more than one space within that context, he/she will be asked to select a single value to proceed.
    
