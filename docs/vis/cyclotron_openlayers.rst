OpenLayers Widget
=================

The OpenLayers Map Widget wraps `OpenLayers version 4.6.5 <https://openlayers.org/en/v4.6.5/apidoc/>`_ to create dynamic maps and represent geographical data.

Configuration
-------------

The following table illustrates the configuration properties available (required properties are marked in **bold**).

================== ============================ =============== ===========================
Property           JSON Key                     Default         Description
================== ============================ =============== ===========================
**Center**         **center.x**, **center.y**                   X and Y coordinates to center the map on
**Zoom**           **zoom**                                     Zoom level (0 is zoomed out)
Data Source        dataSource                                   Name of the Data Source providing data for the overlays
Layers             layers                       One OSM layer   Array of layers that will be added to the map (more details below).
Overlay Groups     overlayGroups                                Groups of overlays that will be displayed over the map, attached to a given position (more details below)
Controls           controls                                     Array of controls that will be added to the map (more details below)
DataSource Mapping dataSourceMapping                            Mapping between overlay properties and Data Source fields (more details below)
Specific Events    specificEvents                               Array of parameters (defined in the Parameters section) and widget events that can trigger their change (more details below)
Projection         projection                   ``EPSG:3857``   Projection
================== ============================ =============== ===========================

Layers
******

Layers property defines the layers that will be added to the OpenLayers map. Every layer must have a source, except VectorTile layers. Note that although OpenLayers provides many optional properties to configure layers, the current implementation of the OpenLayers Map Widget does not support all of them. If no layer type nor source are specified, a single Tile layer with an OSM (Open Street Map) source will be added to the map.

The following table lists which sources are valid for each type of layer:

+------------+-----------------+------------------------+--------------------------------------+
| Type       | Source          | Configuration Required | Notes                                |
+============+=================+========================+======================================+
| Image      | ImageArcGISRest | No                     |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | ImageCanvas     | Yes                    |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | ImageMapGuide   | No                     |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | ImageStatic     | Yes                    |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | ImageWMS        | Yes                    |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | Raster          | Yes                    |                                      |
+------------+-----------------+------------------------+--------------------------------------+
| Tile       | BingMaps        | Yes                    |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | CartoDB         | Yes                    |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | OSM             | No                     |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | Stamen          | Yes                    |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | TileArcGISRest  | No                     |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | TileDebug       | Yes                    |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | TileJSON        | Yes                    |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | TileUTFGrid     | Yes                    |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | TileWMS         | Yes                    |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | WMTS            | Yes                    |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | XYZ             | No                     |                                      |
+            +-----------------+------------------------+--------------------------------------+
|            | Zoomify         | Yes                    |                                      |
+------------+-----------------+------------------------+--------------------------------------+
| Heatmap    | Cluster         | Yes                    | Requires *Weight* property           |
+            +-----------------+------------------------+--------------------------------------+
|            | Vector          | No                     | Requires *Weight* property           |
+------------+-----------------+------------------------+--------------------------------------+
| Vector     | Cluster         | Yes                    | Can use *`Style function`_* property |
+            +-----------------+------------------------+--------------------------------------+
|            | Vector          | No                     | Can use *`Style function`_* property |
+------------+-----------------+------------------------+--------------------------------------+
| VectorTile | VectorTile      | No                     |                                      |
+------------+-----------------+------------------------+--------------------------------------+

The configuration of sources is described in the documentation of `OpenLayers API <https://openlayers.org/en/v4.6.5/apidoc/ol.source.html>`_. Note that the *Configuration* property can contain JavaScript code (see examples).

Overlay Groups
**************

Overlays are elements that will be displayed over the map and attached to a given position. Each of them can have its own style, positioning and content. They can be interactive, that is, you can click on them and use Parameters to store the Name of the selected overlays (see Specific Events).

Every overlay must be part of a *group*, even if there is only one group on the map, and each group of overlays can have one overlay selected at a time. Let's say you want to place some red circles on cities and change them to blue when you click on them: if you want to have only one blue city at a time, you can have all the overlays in one group; if you want to have any number of blue cities at a time, you must add every city in a different group.

Note that overlays within the same group have their own template and style. The concept of group is only related to selection.

Each group has the following properties (required properties are marked in **bold**):

========================== =================== ==============
Property                   JSON Key            Description
========================== =================== ==============
**Name**                   **name**            Unique identifier for the group
Overlay Initially Selected initiallySelected   Name of the overlay that will be assigned the *CSS Class On Selection* when the page is loaded. If not provided, all overlays will have the regular *CSS Class* until one is clicked. To use this field, you must provide names for the overlays.
Overlays                   overlays            Array of overlays
========================== =================== ==============

Each overlay has the following properties (required properties are marked in **bold**):

====================== ================ ============== ==============
Property               JSON Key         Default        Description
====================== ================ ============== ==============
Name                   name                            Unique identifier for the overlay
**CSS Class**          **cssClass**                    CSS class name (defined beforehand in the Styles section of the dashboard)
CSS Class On Selection cssClassSelected                Optional CSS class name (defined beforehand in the Styles section of the dashboard) to apply when the overlay is selected
Position               position                        X and Y coordinates where the overlay will be attached
Positioning            positioning      ``top-left``   Where the overlay is placed with respect to *Position*
Template               template                        HTML template for overlay content
====================== ================ ============== ==============

Note that *Name* property is not required, but it will be generated automatically if not provided, therefore if you want it to be stored in a Parameter you will not recognize which overlay the name belongs to if you do not provide it.

Instead of configuring the overlays in the Overlay Groups property, it is possible to provide them via a Data Source (see Data Source Mapping).

Controls
********

Controls are graphical elements on a fixed position over the map. They can provide user input or information.

============= =============
Control       Description
============= =============
Attribution   Informational button showing the attributions for the map layer sources
MousePosition Shows the coordinates of the mouse cursor
OverviewMap   Shows an overview of the main map
ScaleLine     Shows a scale line with rough Y-axis distances
Zoom          Buttons for zoom-in and zoom-out
ZoomSlider    Slider for zooming in or out
ZoomToExtent  Button for zooming to an extent (default is zooming to zoom 0)
============= =============

More information on controls can be found in the `OpenLayers API documentation <http://openlayers.org/en/latest/apidoc/ol.control.html>`_, e.g. CSS classes to modify control styles.

Data Source Mapping
*******************

As an alternative to *Overlay Groups* property, a Data Source can also be used to provide overlays for the map. In this case, a mapping must be provided for the Widget to correctly read the dataset. The dataset structure differs a bit from the Data Sources for the other Widgets. It recalls the same structure of the *Overlay Groups* property, i.e., an array of groups, each one having a name, the optional name of an overlay pre-selected at loading and an array of overlays with their own properties (coordinates, CSS class, HTML template, etc.).

The Data Source must return a result like this:

::

  [{
      "groupID": "g1",                                   //unique group name (will be assigned randomly if missing)
      "selectedOverlay": "ov1",                          //optional
      "overlays": [{                                     //list of overlays
              "css": "my-css-class",
              "cssSelected": "my-css-class-sel",
              "id": "ov1",
              "content": "<div>OV 1</div>",
              "coordinates": [11, 46],                   //alternatively you can have separate X and Y fields
              "positioning": "top-left"
          },
          {
              "css": "my-css-class",
              "cssSelected": "my-css-class-sel",
              "id": "ov2",
              "content": "<div>OV 2</div>",
              "coordinates": [10, 45],
              "positioning": "bottom-right"
          }
      ]
  }]

You must use the *Data Source Mapping* property set to specify which dataset fields contain each piece of data necessary for configuring the overlays, that is, how to interpret the keys of the objects returned by the Data Source. The following table illustrates how to configure *Data Source Mapping* properties (required properties are marked in **bold**):

================================ ========================== =================
Property                         JSON Key                   Description
================================ ========================== =================
Identifier Field                 identifierField            Name of the field containing a unique identifier for the group. If it is not specified and the datasource provides more that one group, each group will be assigned a random ID.
Overlay Initially Selected Field initiallySelectedField     Name of the field containing the ID of the pre-selected overlay. If not provided, all overlays will have the regular CSS class until one is clicked.
**Overlay List Field**           **overlayListField**       Name of the field containing the list of overlays
**CSS Class Field**              **cssClassField**          Name of the field containing the CSS class for the overlay (must be defined beforehand in the Styles section)
CSS Class On Selection Field     cssClassOnSelectionField   Name of the field containing the CSS class for the overlay after its selection (must be defined beforehand in the Styles section)
Overlay Identifier Field         overlayIdField             Name of the field containing a unique identifier for the overlay. If it is not specified, a random ID will be assigned.
Position Field                   positionField              Name of the field containing an array with the coordinates ([x_coord, y_coord]) for the overlay. Use *xField* and *yField* if coordinates are in two separate fields.
X Coordinate Field               xField                     Name of the field containing X coordinate for the overlay
Y Coordinate Field               yField                     Name of the field containing Y coordinate for the overlay
Positioning Field                positioningField           Name of the field containing the overlay positioning
Template Field                   templateField              Name of the field containing the HTML template for the overlay
================================ ========================== =================

A widget that uses the Data Source in the example above would need the following *Data Source Mapping*:

::

  "dataSourceMapping": {
      "identifierField": "groupID",
      "initiallySelectedField": "selectedOverlay",
      "overlayListField": "overlays",
      "cssClassField": "css",
      "cssClassOnSelectionField": "cssSelected",
      "overlayIdField": "id",
      "positionField": "coordinates",
      "positioningField": "positioning",
      "templateField": "content"
  }

Note that if the Data Source provides more than one group, each object in the array must have the same keys (e.g. in every object, the group ID can be found in a field named "groupID").

Specific Events
***************

Events generated exclusively by the OpenLayers Widget (see `Parameter-based Interaction <https://digitalhub.readthedocs.io/en/latest/docs/vis/cyclotron_parameters.html>`_). They produce a value that will be stored in the given Parameter. Note that Parameters must be defined beforehand in the Parameters section of the dashboard.

The subproperty *Section* can be used to specify the name of the map portion that triggers the event. If the event is triggered by the map itself, you can leave this option empty.

=================== ============================= =====================
Event               Value                         Section
=================== ============================= =====================
``clickOnOverlay``  Name of the overlay clicked   Name of an overlay group (e.g. "g1")
``clickOnWMSLayer`` Feature Info                  Name of a WMS layer (e.g. "topp:states"), obtained with `getGetFeatureInfoUrl`_
=================== ============================= =====================

Examples
--------

Many examples of maps can be found on the `OpenLayers website <https://openlayers.org/en/v4.6.5/examples/>`_, although not all of them are reproducible with Cyclotron.

Although *Configuration* property has the structure of a JSON object in the editor, it is a JavaScript string that is evaluated at dashboard loading, therefore it can contain JavaScript code (e.g. object instantiation).

OSM layer and WMS layer with zoom control
*****************************************

::

  {
      "center": {
          "x": "11.123251",
          "y": "46.044685"
      },
      "controls": [{
          "control": "Zoom"
      }],
      "layers": [{
          "source": {
              "name": "OSM"
          },
          "type": "Tile"
      }, {
          "source": {
              "configuration": "{\r\n    \"url\": \"http://my.geoserver.com/wms\",\r\n    \"params\": {\r\n        \"FORMAT\": \"image/png\",\r\n        \"VERSION\": \"1.1.1\",\r\n        \"STYLES\": \"\",\r\n        \"LAYERS\": \"topp:states\"}\r\n}",
              "name": "ImageWMS"
          },
          "type": "Image"
      }],
      "widget": "openLayersMap",
      "zoom": 8
  }

Default OSM layer and some overlays
***********************************

This example uses air quality data stations located in Trentino to illustrate how to configure selectable overlays. Each overlay is positioned over a station and is represented as a circle whose color is assigned randomly among five options. As an overlay is clicked on, it becomes the currently selected one and the circle enlarges. The ID of the currently selected overlay is held by the Parameter "currentStation". When you select an overlay, "currentStation" is updated and such update triggers the refresh of the Data Source, which in turn changes the color and template of the overlays.

The following dashboard components are required:

- Parameter:

::

  {
      "name": "currentStation",
      "defaultValue": "7"
  }

- CSS Style:

::

  .station {
      opacity: .8;
      border-radius: 50%;
      width: 50px;
      height: 50px;
      line-height: 50px;
      text-align: center;
      font-size: 12px;
  }

  .station.sel {
      width: 70px !important;
      height: 70px !important;
      border: 2px solid #ddd;
      padding: 20px 20px;
      margin: auto;
  }

  .station.level1 {
      background-color: #7BBB6D;
  }
  .station.level2 {
      background-color: #BBCE55;
  }
  .station.level3 {
      background-color: #EDC12F;
  }
  .station.level4 {
      background-color: #F09227;
  }
  .station.level5 {
      background-color: #E64770;
  }

- JSON Data Source
  - Name: ``air-quality-stations``
  - URL: ``https://am-test.smartcommunitylab.it/dss/services/ariadb/Stations``
  - Post-Processor:
  
  ::
  
    e = function(dataSet){
        res = [];
        var stations = dataSet.Entries.Entry; //array of objects with keys: id, name, X, Y

        _.each(stations, (station) => {
            var level = Math.floor(Math.random() * (6 - 1)) + 1;
            station.css = 'station level' + level;
            station.cssSelected = 'sel';
            station.template = '<div>' + level + '</div>';
        });
        stationGroup = {};
        stationGroup.currentlySelected = '7';
        stationGroup.stationOverlays = stations;
        res.push(stationGroup);
        return res;
    }
  
  - Subscription to Parameters: ``currentStation``

- OpenLayers Map Widget:

::

  {
      "center": {
          "x": "11.123251",
          "y": "46.044685"
      },
      "dataSource": "air-quality-stations",
      "dataSourceMapping": {
          "cssClassField": "css",
          "cssClassOnSelectionField": "cssSelected",
          "initiallySelectedField": "currentlySelected",
          "overlayIdField": "id",
          "overlayListField": "stationOverlays",
          "templateField": "template",
          "xField": "X",
          "yField": "Y"
      },
      "specificEvents": [{
          "event": "clickOnOverlay",
          "paramName": "currentStation"
      }],
      "widget": "openLayersMap",
      "zoom": 11
  }



.. _Style function: https://openlayers.org/en/v4.6.5/apidoc/ol.html#.StyleFunction
.. _getGetFeatureInfoUrl: https://openlayers.org/en/v4.6.5/apidoc/ol.source.ImageWMS.html#getGetFeatureInfoUrl
