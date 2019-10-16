Cyclotron
=============

Cyclotron is included in the platform as a fork of `Expedia's software <https://github.com/ExpediaInceCommercePlatform/cyclotron/tree/1.48.0>`_. Head over to https://www.cyclotron.io/ for an in-depth description of its functionalities. The fork introduces the following features:

- security based on Authentication and Authorization Control Module (AAC)
- new widgets: time slider, OpenLayers map, Google charts, Deck.gl, widget container
- new data sources: OData
- parameter-based interaction between dashboard components

.. toctree::
   cyclotron_aac
   cyclotron_slider
   cyclotron_openlayers
   cyclotron_googlecharts
   cyclotron_deckgl
   cyclotron_container
   cyclotron_odata
   cyclotron_parameters

Installation
------------

Note that the detailed installation procedure is only summarized here and is better described on `Expedia's page <https://github.com/ExpediaInceCommercePlatform/cyclotron>`_.

Requirements:

- `AAC <https://github.com/smartcommunitylab/AAC>`_
- Node.js
- MongoDB 2.6+ (`installation instructions <http://docs.mongodb.org/manual/installation/>`_)

Installation steps:

0. Ensure that MongoDB and AAC are running. Cyclotron will automatically create a database named "cyclotron".
1. Clone this repository.
2. Install the REST API and create the configuration file ``cyclotron-svc/config/config.js``. Paste in it the content of ``sample.config.js``, which contains the configurable properties of the API, such as MongoDB instance and AAC endpoints.
3. Install Cyclotron website.
4. Start both:

   - API: from ``cyclotron-svc`` run the command ``node app.js``
   - website: from ``cyclotron-site`` run the command ``gulp server``

Now Cyclotron is running with its default settings and authentication is disabled.

Deployment
----------

TODO
