AAC API
----------
  
The Swagger UI for the AAC API is available at ``http://localhost:8080/aac/swagger-ui.html``.   
  
Profile API  
^^^^^^^^^^^^

To obtain the basic user data the following call should be performed (scope ``profile.basicprofile.me``):  :: 

    GET /aac/basicprofile/me HTTPS/1.1 
    Host: aacserver.com 
    Accept: application/json 
    Authorization: Bearer <token-value>  
  
If the token is valid, this returns the user data: ::

    {
    "name": "Mario",
    "surname": "Rossi",
    "userId": "6789",
    "username": "mario@gmail.com"
    }  

To obtain the account-related data (e.g., the Identity Provider-specific attributes),  the following call should be performed  (scope ``profile.accountprofile.me``): ::  

    GET /aac/accountprofile/me HTTPS/1.1 
    Host: aacserver.com 
    Accept: application/json 
    Authorization: Bearer <token-value>  
  
If the token is valid, this returns the user data, e.g., ::

    {
      "name": "Mario",
      "surname": "Rossi",
      "username": "rossi@gmail.com",
      "userId": "1",
      "accounts": {
        "google": {
          "it.smartcommunitylab.aac.surname": "Rossi",
          "it.smartcommunitylab.aac.username": "rossi@gmail.com",
          "it.smartcommunitylab.aac.givenname": "Mario",
          "email": "rossi@gmail.com"
        }
      }
    }

Token API
^^^^^^^^^^^^^^^^^^^^

ion API (ITEF RFC 7662). The call should be provided with the Client Credentials (e.g., as BasicAuth, with ``client_id:client_secret``) ::

    POST /aac/token_introspection?token=<token-value> HTTPS/1.1 
    Host: aacserver.com 
    Accept: application/json 
    Authorization: Basic <client-credentials>  

The request returns a JSON token with standard claims regarding the user, the client, and the token. The returned claims
rappresent the standard token claims (see ITEF RFC 7662 for details) as well as custom AAC claims (prefixed with ``aac_``). ::

        {
          "active": true,
          "client_id": "7b4f9b2a-71f6-412d-93e6-030c14910083",
          "scope": "profile.basicprofile.me profile.accountprofile.me openid"
          "username": "admin@carbon.super",
          "token_type": "Bearer",
          "sub": "8",
          "iss": "https://aac.example.com",
          "aud": "7b4f9b2a-71f6-412d-93e6-030c14910083", 
          "exp": 123456789,
          "iat": 123450000,
          "nbf": 123456788,
          "aac_user_d": "8",
          "aac_grantType": "implicit",
          "aac_applicationToken": false,
          "aac_am_tenant": "tenant1.com"
        }

Role API
^^^^^^^^^^

The role API allows for the checking the role of the specific users. More details can be found on the
Swagger documentation.

OpenID API
^^^^^^^^^^

AAC provides support for some of the OpenID Connect functionalities. The OIDC metadata information is available at
the  ``/.well-known/openid-configuration`` endpoint, where the supported features and relevant endpoints are captured.


In particular, the OpenID userinfo endpoint allows for getting the standard user info claims (scopes ``profile``, ``email``). The response is provided
in the form of JSON object or JWT token. ::

    GET /aac/userinfo HTTPS/1.1 
    Host: aacserver.com 
    Accept: application/json 
    Authorization: Bearer <token-value>  

The data provided represents the subset of standard OpenID claims. ::

      {
        "sub": "123456789",
        "name": "Mario Rossi",
        "preferred_username": "rossi@mario.com",
        "given_name": "Mario",
        "family_name": "Rossi",
        "email": "rossi@mario.com",
      }

