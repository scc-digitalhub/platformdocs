
Data Model
-----------------

Users
^^^^^^^^^^
TODO

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
    
Resources (APIs)
^^^^^^^^^^^^^^^^^^
TODO

