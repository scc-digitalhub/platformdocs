Deck.gl Widget
==============

The Deck.gl Widget wraps `Deck.gl <https://deck.gl/#/>`_ to create a stack of visual layers, usually representing geographical data.

Configuration
-------------

The following table illustrates the configuration properties available.

========================== ====================== ===================
Property                   JSON Key               Description
========================== ====================== ===================
Data Source                dataSource             Name of the Data Source providing data for one layer
View State                 viewState              ``viewState`` property (see `viewState`_). Zoom defaults to 0.
Additional Deck Properties additionalDeckProps    Additional supported Deck properties (e.g. ``getTooltip``). See `Deck`_
Layers                     layers                 Deck layers
========================== ====================== ===================

Layers
******

Layers property defines a set of transparent overlays, each one representing a data collection that will be added to the deck. Layers can be interactive if the appropriate event (e.g. on click, on hover, on drag) handlers are configured (see `Adding interactivity <https://deck.gl/#/documentation/developer-guide/adding-interactivity>`_).

Deck.gl provides several layer types. The widget currently supports all the `Core Layers <https://deck.gl/#/documentation/deckgl-api-reference/layers/overview>`_.

======================== ================== =========== ===================
Property                 JSON Key           Default     Description
======================== ================== =========== ===================
**Type**                 **type**                       Layer type (required)
Configuration Properties configProperties               Object with layer properties. Despite having the structure of a JSON object in the editor, it is a JavaScript string that is evaluated at dashboard loading, therefore it can contain JavaScript (e.g. functions). See `supported properties`_ (note that they may vary depending on the layer type).
Use Data Source          useDataSource      ``false``   Use the Data Source to provide data for this layer (alternative to ``data`` property). The data will be used for the first layer found with this property set to true.
======================== ================== =========== ===================

Examples
--------

Basic ScatterplotLayer
**********************

This example adapts the snippet used on Deck.gl `Getting Started <https://deck.gl/#/documentation/getting-started/using-standalone>`_ page. The Data Source provides data to create a circle whose color is changed at every Data Source refresh (5 seconds).

Create a new Data Source:

- Type: JavaScript
- Processor:

::

  e = function(promise){
      return [{
          "position": [-122.45, 37.8],
          "color": [Math.floor(Math.random() * Math.floor(256)), 0, 0], //random red
          "radius": 100000
      }];
  }

- Auto-Refresh: 5

Create a new Deck.gl widget:

- Data Source: the one you just created
- Longitude: -122.45
- Latitude: 37.8
- Zoom: 4
- ScatterplotLayer:

  - Configuration Properties (functions can also be defined in the Scripts section of the dashboard and referenced here):
  
  ::
  
    {
        "getFillColor": d => d.color,
        "getRadius": d => d.radius
    }
  
  - Use Data Source: true

Adding a tooltip
****************

To add a tooltip (or any other Deck event handler) you can use *Additional Deck Properties* property (see documentation about the `picking info object <https://deck.gl/#/documentation/developer-guide/adding-interactivity?section=the-picking-info-object>`_):

::

  {
      "getTooltip": tooltipFunc
  }

In the Scripts section, define a function ``tooltipFunc`` that returns a div with the mouse position:

::

  tooltipFunc = function(info){
      return {
          html: ('<div>' + info.x + ', ' + info.y + '</div>'),
          style: {'background-color': 'yellow'}
      }
  }

Multiple interactive layers
***************************

This example implements Deck.gl `basic example <https://github.com/uber/deck.gl/blob/master/examples/get-started/scripting/basic/index.html>`_, which displays airports around the world. Whenever an airport is clicked, an alert will pop up with the airport name.

Create a new Deck.gl widget with the following *View State*:
::

  {
      "bearing": 0,
      "latitude": "51.47",
      "longitude": "0.45",
      "pitch": 30,
      "zoom": 4
  }

Add a ScatterplotLayer with the following *Configuration Properties*:

::

  {
      "id": "base-map",
      "data": "https://d2ad6b4ur7yvpq.cloudfront.net/naturalearth-3.3.0/ne_50m_admin_0_scale_rank.geojson",
      "stroked": true,
      "filled": true,
      "lineWidthMinPixels": 2,
      "opacity": 0.4,
      "getLineDashArray": [3, 3],
      "getLineColor": [60, 60, 60],
      "getFillColor": [200, 200, 200]
  }

Add another ScatterplotLayer with the following *Configuration Properties*:

::

  {
      "id": "airports",
      "data": "https://d2ad6b4ur7yvpq.cloudfront.net/naturalearth-3.3.0/ne_10m_airports.geojson",
      "filled": true,
      "pointRadiusMinPixels": 2,
      "opacity": 1,
      "pointRadiusScale": 2000,
      "getRadius": f => (11 - f.properties.scalerank),
      "getFillColor": [200, 0, 80, 180],
      "pickable": true,
      "autoHighlight": true,
      "onClick": info => info.object && alert(`${info.object.properties.name} (${info.object.properties.abbrev})`)
  }

Add an ArcLayer with the following *Configuration Properties*:

::

  {
      "id": "arcs",
      "data": "https://d2ad6b4ur7yvpq.cloudfront.net/naturalearth-3.3.0/ne_10m_airports.geojson",
      "dataTransform": d => d.features.filter(f => f.properties.scalerank < 4),
      "getSourcePosition": f => [-0.4531566,51.4709959],
      "getTargetPosition": f => f.geometry.coordinates,
      "getSourceColor": [0, 128, 200],
      "getTargetColor": [200, 0, 80],
      "getWidth": 1
  }

.. _viewState: https://deck.gl/#/documentation/deckgl-api-reference/deck?section=views-array-
.. _Deck: https://deck.gl/#/documentation/deckgl-api-reference/deck
.. _supported properties: https://deck.gl/#/documentation/deckgl-api-reference/layers/overview
