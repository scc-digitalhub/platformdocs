Data Services Server Usage
===========================

OData requests
---------------

1.If the data service is inside the super tenant: ::

	https://mydomain.com/dss_proxy_context_path/odata/ODataSampleService/default/$metadata

2.If the data service is inside a specific tenant: ::

	https://mydomain.com/dss_proxy_context_path/odata/t/domain.com/ODataSampleService/default/$metadata

REST Requests
--------------
1.If the data service is inside the super tenant: ::

	https://mydomain.com/dss_proxy_context_path/services/AccountDetailsService/Account?AccountId=1

2.If the data service is inside a specific tenant: ::

	https://mydomain.com/dss_proxy_context_path/services/t/domain.com/AccountDetailsService/Account?AccountId=1

Securing Private Data Services
-------------------------------
In this version of Data Services Server it is taken in consideration the security policy of accessing data services exposed over REST or OData format.
For this purpose inside the configuration of a service it is added new parameter to specify if it is going to be a public or private service.

After being specified as private, the data service is going to be accessed in two ways:

1.Providing Authorization Token Header ::

	https://mydomain.com/dss_proxy_context_path/odata/ODataSampleService/default/$metadata 
	Authorization Header: Bearer $token$

2.Providing ApiKey Parameter::

	https://mydomain.com/dss_proxy_context_path/odata/ODataSampleService/default/$metadata?apikey=$apikey$
