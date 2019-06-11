Server APIs
============

This section describes the APIs made available by the Org-Manager server. Reading it is not necessary to install or use Org-Manager, as the client takes care of calling APIs appropriately when operations are requested through the UI.

All requests must include the **Authorization** header, containing ``Bearer <token value>``. In case an error occurs, each API will return a response with an appropriate status code, usually with some details in the response body.

Assuming the server is being hosted on *localhost* at port *7979*, the **Swagger UI** for the Org-Manager APIs is available at **http://localhost:7979/swagger-ui.html**.

Most APIs have security restrictions that require the user to be owner of the organization they are attempting to alter, or to have administrator privileges.

The owner of a specific organization is defined as a user with the following role in AAC: ``components/<organization_slug>:ROLE_PROVIDER``

A user has administrator privileges when the access token they are using is a client token with the ``organization.mgmt`` scope, or when they have the following role in AAC:
``organizations:ROLE_PROVIDER``

Also keep in mind that, for some of these APIs to work correctly, the access token used must have the following scopes: ``profile``, `email``, ``user.roles.me``, ``profile.basicprofile.me``, ``profile.accountprofile.me``.

Create organization
------------------------

This API is used to create an organization. Its response contains a JSON object that represents the newly created organization. This response will contain an additional field, ``id``, necessary to recognize the organization and useful when calling other APIs.

Note that, since the ``email`` field will be interpreted as the starting owner of the organization, a user with its value as name will be created on the server side. This means that AAC must have a user with this username, otherwise an error will occur.

The ``name`` and ``surname`` fields inside the ``contacts`` object also must match the corresponding fields in AAC.

**Requirements**: must have administrator privileges

**End-point**: /api/organizations

**Method**: POST

**Body**: JSON object describing the organization. The following fields can be defined:

1. ``name`` – Name of the organization. Required. May contain alphanumeric characters, space, dash (-) or underscore (_). Cannot have the same name as an already existing organization, even if case is different. Any leading or trailing spaces will be ignored, and multiple consecutive spaces will be replaced with a single space.

2. ``slug`` – Defines the domain of the organization. Optional: if specified, it can only contain alphanumeric lower case characters and underscores. If left out, it will be generated from the name, converting it to lower case and replacing dashes and spaces with underscores.

3. ``description`` – Description of the organization. Required.

4. ``contacts`` – Inner JSON object describing the contacts. Required. Its 5 inner properties are:
   - ``email`` – E-mail. Required. Will be used as name of the owner of the organization.
   - ``name`` – Name of the contact. Required.
   - ``surname`` – Surname. Required.
   - ``web`` – URL. Optional.
   - ``phone`` – Array of strings for phone numbers. Optional.
   - ``logo`` – URL. Optional.
   - ``tag`` – Array of strings for tags. Optional.
   - ``active`` – Can take ``true`` or ``false`` as values. Indicates whether the organization is enabled or disabled. Optional, will default to `true` if omitted.

**Sample request body**::

	{
	  "name":"My Organization",
	  "slug":"my_org",
	  "description":"This is my test organization.",
	  "contacts": {
	    "email":"jsmith@my_org.com ",
	    "name":"John",
	    "surname":"Smith",
	    "web":"http://www.example.com",
	    "phone":["12345","67890"],
	    "logo":"http://www.example.com/images/logo.png"
	  },
	  "tag":["test","testing"],
	  "active":"true"
	}

Search organizations
--------------------

API for searching organizations. Organizations are searched by name. Responses are returned as pages of size 20.

If the authenticated user has administrator privileges, they will see all organizations, otherwise they will see only organizations they are part of.

**End-point**: /api/organizations

**Method**: GET

**Parameters**:

1. ``name`` – Name to search. Case insensitive. All organizations with a name that contains this parameter will be returned.

2. ``page`` – Page to be returned. Can be omitted, since most of the time the organizations returned will be less than 20. Starts from 0, so if you want the second page, use ``page=1``.

**Sample request URL**: ``http://localhost:7979/api/organizations?name=Company&page=3``

Update organization
-------------------

Updates an organization. Only certain fields may be updated. The ``id`` of the organization must be known, and used in the request URL.

**Requirements**: must have the administrator privileges, or be owner of the organization.

**End-point**: /api/organizations/<organization_id>/info

**Method**: PUT

**Body**: JSON with the fields to change. Only description, contacts and tags may be changed; any other field present in the request will be ignored. Fields will only be updated if present in the input, so if you do not want to change a field, simply omit it from the request.

**Sample request URL**: ``http://localhost:7979/api/organizations/1/info``

**Sample request body**::

	{
	  "description":"New description.",
	  "contacts": {
	    "web":"http://www.test.com",
	    "phone":["12345","57575"]
	  },
	  "tag":["testing"]
	}

Enable organization
-------------------

Enables an organization. Simply changes the ``active`` field to ``true``.

**Requirements**: must have administrator privileges

**End-point**: /api/organizations/<organization_id>/enable

**Method**: PUT

**Sample request URL**: ``http://localhost:7979/api/organizations/3/enable``

Disable organization
--------------------

Disables an organization. Simply changes the ``active`` field to ``false``. Other than the endpoint, it is identical to the **Enable organization** API.

**Requirements**: must have administrator privileges

**End-point**: /api/organizations/<organization_id>/disable

**Method**: PUT

**Sample request URL**: ``http://localhost:7979/api/organizations/3/disable``

Delete organization
-------------------

Deletes an organization. Also unregisters all members belonging to it, deletes all their roles within it, and deletes all tenants within it. An organization must be disabled before it can be deleted.

**Requirements**: must have administrator privileges

**End-point**: /api/organizations/<organization_id>

**Method**: DELETE

**Sample request URL**: ``http://localhost:7979/api/organizations/1``

List available components
-------------------------

Lists available components, together with a few properties for each of them.

**End-point**: /api/components

**Method**: GET

List possible roles for a component
-----------------------------------

Returns a list of strings, representing what roles may be attributed to a user when added to a tenant within a specific component.

**End-point**: /api/components/<component_id>/roles

**Method**: GET

**Sample request URL**: ``http://localhost:7979/api/components/nifi/roles``

Configure tenants for an organization
---------------------------------------

Allows configuring which tenants an organization should have.

**Requirements**: must have administrator privileges

**End-point**: /api/organizations/<organization_id>/configuration

**Method**: POST

**Body**: JSON object containing components and tenants for each component.

1. ``componentId`` – Identifies the component. Must be chosen among the values that can be found by calling the **Listing available components** API. Note that if a component is not specified in the body, it will not be altered.

2. ``tenants`` – Array of strings for the tenants. If a component previously contained tenants that are not present in this new array, those tenants will be removed.

**Sample request body**::

	[
	  {
	    "componentId":"nifi",
	    "tenants":[
	      "trento",
	      "ferrara"
	    ]
	  },{
	    "componentId":"dss",
	    "tenants":[
	      "reggio"
	    ]
	  }
	]

Note that only tenants for the ``nifi`` and ``dss`` components will be affected, as no other components are present in the input. For example, tenants for the component ``apimanager`` will not be altered, since ``apimanager`` was not specified in the body.

Display tenants of the organization
-----------------------------------

Displays the tenants that have been configured for the input organization.

**Requirements**: must have administrator privileges, or be the owner of the organization

**End-point**: /api/organizations/<organization_id>/configuration

**Method**: GET

**Sample request URL**: ``http://localhost:7979/api/organizations/1/configuration``

List users in an organization
-----------------------------

Lists users that belong to the indicated organization. The ``id`` of the organization must be known. An optional parameter may be specified to act as a filter on the desired users' names.

**Requirements**: must have administrator privileges, or be the owner of the organization

**End-point**: /api/organizations/<organization_id>/members

**Method**: GET

**Parameters**:

1. ``username``: If specified, only members whose user name contains this value (case insensitive) will be returned. If omitted, all members of the organization will be returned.

**Sample request URL**: ``http://localhost:7979/api/organizations/1/members?username=john``

Add a user to an organization
-----------------------------

Grants a user the roles listed in the request body. All roles they previously had within the organization, but that are not present in this new configuration, will be removed. The user will be added to the organization, in case they were previously not registered. This means that AAC must have a user with this username, otherwise an error will occur. The response will also contain the `id` of the member within the organization.

It is also possible, for administrators only, to grant/revoke the status of owner of the organization through this API.

**Requirements**: must have administrator privileges, or be owner of the organization (cannot grant/revoke owner status)

**End-point**: /api/organizations/<organization_id>/members

**Method**: POST

**Body**: JSON object containing the user's name and the roles they should have:

1. ``username`` – Name of the user to add. Must be a valid name recognized by the identity provider.

2. ``roles`` – Array of JSON objects representing the roles to add. Each role has 2 properties:

   - ``contextSpace`` – domain of the role. It must be one of the domains registered in the organization. It should have the following structure: ``components/<component_id>/<space name>``
   
   - ``role`` – Role of the user in the domain
   
3. ``owner`` - Boolean parameter that can only be set by administrators. If this parameter appears in a call performed without administrator rights, it will be ignored.
  
**Sample request URL**: ``http://localhost:7979/api/organizations/1/members``

**Sample request body**::

	{
	  "username":"bob@test.com",
	  "roles": [{
	    "contextSpace":"components/nifi/trento",
	    "role":"ROLE_MANAGER"
	  },{
	    "contextSpace":"components/nifi/ferrara",
	    "role":"ROLE_USER"
	  }],
	  "owner":"true"
	}

Remove a user from an organization
----------------------------------

Unregisters a user from an organization, stripping them of all roles they had within it. The ``id`` of the organization, as well as the ``id`` of the member to remove, must be known.

**Requirements**: must have administrator privileges, or be the owner of the organization

**End-point**: /api/organizations/<organization_id>/members/<member_id>

**Method**: DELETE

**Sample request URL**: ``http://localhost:7979/api/organizations/1/members/2``