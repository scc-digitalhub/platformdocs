Dremio
======================

Dremio is a *Data-as-a-Service* platform, which enables data analysts and scientists to autonomously explore, validate and curate data from a variety of sources, all in a single, unified and coherent interface.
Built for teams, Dremio leverages spaces and virtual datasets to offer a data platform with the following features.

Website https://www.dremio.com

Data sources
----------------
Support for modern data lakes built on a variety of different systems: 

- native integrations with major RDBMS such as PostgresSQL, MySQL, MS SQL, IBM DB2, Oracle
- NoSQL integration for modern datastores as MongoDB, Elasticsearch
- support for file based datasources, cloud storage systems, NAS

Data exploration
-----------------
Dremio offers an unified view across all datasets connected to the platform, with :

- live data visualization during queries preparation and execution, with dynamic previews
- optimized query pushdown for all sources in native query language
- virtual datasets based on complex queries available as sources for analytics and external tools


Cloud ready
------------
Dremio is architected for cloud environments, with elastic compute abilities and dynamic horizontal scaling.
Data reflections can be stored into distributed storage platforms such as *S3*, *HDFS*, *ADLS*.


Screenshots
-------------
*from website*

.. image:: ../assets/dremio/dremio_9.png
    :width: 200px

.. image:: ../assets/dremio/dremio_2.png
    :width: 200px

.. image:: ../assets/dremio/dremio_13.png
    :width: 200px

.. image:: ../assets/dremio/dremio_35.png
    :width: 200px        


Installation
----------------
Dremio is a Java software, and requires a compatible JDK installed.
Current version supports only ``OpenJDK 1.8`` and ``Oracle JDK 1.8``.

System requirements are :

- a supported Linux distribution: ``RHEL/CentOS 6.7+/7.3+``, ``SLES 12+``, ``Ubuntu 14.04+``, ``Debian 7+``
- at least ``4 CPU cores`` and ``8GB RAM`` for starting the software.

Given the nature of data analysis, and the distributed design of the software, production deployments should follow the following indications.

=============== ===============================
Node role       Hardware required
=============== ===============================
Coordinators    8 CPU/16GB RAM recommended
Executors       4 CPU/16GB RAM minimum
                16 CPU/64GB RAM recommended
=============== ===============================


Read more at https://docs.dremio.com/deployment/rpm-tarball-install.html 



Platform fork
----------------------

The open source version of Dremio lacks some *enteprise* features, such as external authentication and permission/roles system.
As such, we have forked the source code and modified both the backend and the frontend UI in order to:

- add external OAuth2 support, with access via the secure *authorization_code* flow and a native Dremio token integration for UI
- add roles for *admin/user* distinction, with the exclusive ability for admins to manage data sources, users and spaces
- implement automatic user creation upon valid OAuth2 access, with personal space definition and resource creation
- introduce role mapping from OAuth2 userInfo claims, with dedicated configuration items (available via ENV) for deployment flexibility
- update UI to add OAuth2 login and to properly hide admin actions and menus to unprivileged users
- disable by default upstream *support service*, which exposes metrics, interactive chat and debug information to Dremio.com for licensed enterprise environments

The last item should be reviewed in privacy-sensitive environments, since the complete deactivation of user and session data leakage to dremio.com and its partners requires the explicit configuration of various properties in ``dremio.conf``.


The new properties introduced by the fork are:

.. code-block:: javascript

    services.coordinator.web.auth: {
        type: "oauth", # Possible values are "internal", "oauth"
        oauth: {
            authorizationUrl: ""
            tokenUrl: ""
            userInfoUrl: ""
            callbackUrl: ""
            clientId: ""
            clientSecret: ""
            roleField: ""
            roleUser: "user"
            roleAdmin: "admin"
        }
    }

Additionally, to fully disable dremio.com intercom set the following:

.. code-block:: javascript

   services.coordinator.web.ui {
        intercom: {
            enabled: false
            appid:  ""
        }
   }




Building from source
--------------------

Dremio is a *maven* project, and as such can be properly compiled, along with all the dependencies, via the usual ``mvn`` commands:

    mvn clean install

Since some modules require license acceptance and checks, in automated builds it is advisable to skip those checks to avoid a failure:

    mvn clean install DskipTests -Dlicense.skip=true 

The *skipTests* flags is useful to speed up automated builds, for example for Docker container rebuilds, once the CI has properly executed all the tests.


During development of new modules or modifications, it is advisable to disable the *style-checker* via the ``-Dcheckstyle.skip`` flag.
In order to build a single module, use the following syntax:

    mvn clean install -pl :dremio-common -DskipTests -Dlicense.skip=true -Dcheckstyle.skip

where *dremio-common* is the module.
To test the build, we can execute only the *distribution* tasks, which will produce a complete 
distribution tree under the ``distribution/server/target`` folder, and a **tar.gz** with the deployable package named as *version-date-build*, for example ``./distribution/server/target/dremio-community-3.2.1-201905191350330803-1a33f83.tar.gz``.

    mvn clean install -pl :dremio-distribution -DskipTests -Dlicense.skip=true

The resulting archive can be installed as per upstream instructions.    