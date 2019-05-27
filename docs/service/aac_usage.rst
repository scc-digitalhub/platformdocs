User Guide
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
  
  
Using AAC for Securing Resources and Services
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In this scenario the goal is to restrict access to the protected resources (e.g., an API endpoint). Also in this case
the scenario relies on the use of OAuth2 protocol. The two cases are considered here:

* The protected resources deal with user-related data or operation. In this case (according to OAuth2.0), the access to the API should be accompanied with the ``Authorization`` header that contains the access token obtained via Implicit Flow (or via Authorization Code Flow).
* The protected resource does not deal with user-related data or operation and is not performed client-side. In this case, the access to the API should also be accompanied with the ``Authorization`` header that contains the access token obtained via OAuth2.0 client credentials flow.
  
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
    
