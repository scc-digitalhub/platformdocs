PostgreSQL
=============

PostgreSQL (https://www.postgresql.org/) is a powerful, flexible and advanced open-source relational database system
which provides a secure, stable and reliable storage for structured datasets, by leveraging the 
* Relational Database Management System (RDBMS) * model to 
build a consistent, safe and performing data layer. 

At the same time, PostgreSQL is a general purpose and object-relational database management system, with
*user-defined* data types, functions and functional languages, posing as a solid and reliable platform
for many useful extensions which can add interesting functionalities to the base. 


PostGIS
-------------
PostGIS (https://postgis.net/) is an open source *OCG compliant* spatial database extender for PostgreSQL.
It add advanced spatial and geographical capabilities, and specific geometry data types to the database.

Among many capabilities, notable features are:

* processing and analytic functions for both vector and raster data 
* 3D measurement functions, 3D spatial index support and 3D surface area types
* seamless raster/vector analysis support 
* network topology support
* SQL/MM topology support
* spatial reprojection SQL callable functions
* support for importing and rendering vector data from textual formats such as KML, GeoJSON, WKT
* rendering raster data in various image formats such as GeoTIFF, PNG, JPG


In order to enable PostGIS in a specific database, after installing the plugin, it is mandatory to 
leverage the SQL console to activate the various extensions needed:

.. code-block:: none

    -- Enable PostGIS (as of 3.0 contains just geometry/geography)
    CREATE EXTENSION postgis;
    -- enable raster support (for 3+)
    CREATE EXTENSION postgis_raster;
    -- Enable Topology
    CREATE EXTENSION postgis_topology;
    -- Enable PostGIS Advanced 3D
    -- and other geoprocessing algorithms
    -- sfcgal not available with all distributions
    CREATE EXTENSION postgis_sfcgal;
    -- fuzzy matching needed for Tiger
    CREATE EXTENSION fuzzystrmatch;
    -- rule based standardizer
    CREATE EXTENSION address_standardizer;
    -- example rule data set
    CREATE EXTENSION address_standardizer_data_us;
    -- Enable US Tiger Geocoder
    CREATE EXTENSION postgis_tiger_geocoder;


DigitalHub integration
------------------------

In order to properly utilize and share the database system with the various tenants,
the hub will leverage the *ResourceManager* as the only operator able to create, alter and manage
databases within the server, and will also integrate users via dedicated read/write roles.

This approach ensures that no user will be able to interfere and access datasets of other tenants, 
while giving at the same time authorized operators the ability to autonomously create and manage
databases within their own slice of the share PostgreSQL cluster.

Credentials will be securely handled by the platform, thanks to the integration between *AAC* and *ResourceManager*,
ensuring that users and applications will use a properly scoped and limited access to the system.
