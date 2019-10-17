OData Data Source
=================

The Odata Data Source loads data via HTTP GET requests to Odata services. It is a more specialized alternative to the JSON Data Source.

Please note that it may be necessary to set the optional *Proxy* property in the Data Source in order to access web sites or services in other domains. This property lists other Cyclotron servers which may have the required connectivity. By default, this Data Source uses the current Cyclotron environment to proxy the request.

Configuration
-------------

The following table illustrates the configuration properties available:

=========================== ======================= ======= =============
Property                    JSON Key                Default Description
=========================== ======================= ======= =============
**URL**                     **url**                         OData Service URL (required)
Response Adapter            responseAdapter         ``raw`` How the response will be converted to Cyclotron's data format
Query Parameters            queryParameters                 Optional query parameters. If there are already query parameters in the URL, these will be appended. The keys and values are both URL-encoded.
Auto-Refresh                refresh                         Number of seconds before Data Source reload
Pre-Processor               preProcessor                    JavaScript function that can inspect and modify the Data Source properties before its execution. It takes the Data Source object as an argument returns it modified or a new one.
Post-Processor              postProcessor                   JavaScript function that can inspect and modify the result before returning it
Proxy Server                proxy	                          Alternative proxy server to route the requests through
Options                     options                         Optional request parameters that are passed to the library making the request
AWS Credentials             awsCredentials                  AWS IAM signing credentials for making authenticated requests. If set, the request will be signed before it is sent.
OAuth2.0 Client Credentials oauth2ClientCredentials         OAuth2.0 client credentials for making authenticated requests. If set, the request will be added an Authorization header before it is sent.
=========================== ======================= ======= =============

URL
***

The *URL* property specifies the OData service URI for the request. Query parameters can either be part of the URL or added via the Query Parameters property, however the alternative is not valid for system query options (i.e. those prefixed with the dollar sign $), which must be written in the URL.

Response Adapter
****************

The *responseAdapter* property is used to correctly read the result returned by OData. By default its value is ``raw``, which means that no adaptation occurs and the result is returned as it is, for further elaboration in the *post-processor* or in case the response contains a single raw value (e.g. produced by $count or $value query options).

If an Entity Set is requested to OData, then the ``entity set`` response adapter may be used to retrieve the array of objects returned by OData (i.e. the "value" of the JSON response in OData V4).

If a Single Entity is requested, then the ``single entity`` adapter may be used to remove OData metadata from the result and retrieve the single object.

If a Primitive Property is requested, then the ``primitive property`` adapter may be used to retrieve from the result the raw value of the property.

The data that is sent to the *Post-Processor* will depend on the setting of the response adapter, in that the function receives the output of whichever *responseAdapter* has been selected. In order to manually inspect the entire response, set it to ``raw``.
