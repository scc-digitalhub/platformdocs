Time Slider Widget
==================

The Time Slider widget wraps `noUiSlider <https://refreshless.com/nouislider/>`_ JavaScript range slider. The following table illustrates the configuration properties available (required properties are marked in **bold**).

======================= ================ ========================= ==================
Property                JSON Key         Default                   Description
======================= ================ ========================= ==================
**Minimum date-time**   **minValue**                               Starting value of the slider in the same format as *momentFormat*
**Maximum date-time**   **maxValue**                               Ending value of the slider in the same format as *momentFormat*
Date-time Format        momentFormat     ``YYYY-MM-DD HH:mm``      Any valid `moment.js <https://momentjs.com/docs/#/displaying/format/>`_ format
Step                    step             ``1``                     Number of time units (e.g. days) between slider values
Direction               direction        ``ltr``                   Left-to-right or right-to-left
Orientation             orientation      ``horizontal``            Horizontal or vertical slider
Player                  player                                     Play/pause button for automatic sliding (more details below)
Time Unit               timeUnit         ``days``                  Each slider value corresponds to a minute, hour, day or month
Pips                    pips                                       Scale/points shown near the slider (more details below)
Tooltips                tooltips         ``false``                 Show tooltips
Show Two Handles        twoHandles       ``false``                 Show two handles instead of one. In this case automatic sliding cannot be used. The handles will be placed at the first and last steps unless you specify a different position with Initial Handle Position.
Initial Handle Position handlePosition   ``minValue[, maxValue]``  Initial position of the handle, in the same format as *momentFormat*. If the slider has two handles, use two comma-separated values. Inline JavaScript can also be used, e.g. "${Cyclotron.parameters.handlePos}".
Specific Events         specificEvents                             Array of parameters (defined in the Parameters section) and widget events that can trigger their change (more details below).
======================= ================ ========================= ==================

Player
------

Player property specifies an optional play/pause button that will cause the slider to slide automatically after each interval of the specified length. The button is displayed before (if horizontal) or on top of (if vertical) the slider.

=================== ================== =========== =============
Property            JSON Key           Default     Description
=================== ================== =========== =============
Show Player         showPlayer         ``false``   Show player
Interval            interval           ``1``       Seconds between each sliding
Start On Page Ready startOnPageReady   ``false``   Start the player automatically when the page is loaded
=================== ================== =========== =============

Pips
----

Pips are the points or scale displayed along the slider. Mode property configures the way pips are displayed:

* ``Range`` mode places a pip for every point specified in the range
* ``Steps`` mode places a pip for every step
* ``Positions`` mode places pips at percentage-based positions (percentages can be specified in Values option)
* ``Count`` mode generates a fixed number of pips (specified in Values option)
* ``Values`` mode is the same as Positions mode but with values instead of percentages

More details and examples are provided in the `noUiSlider documentation <https://refreshless.com/nouislider/pips/>`_.

======== ========== =========== =============
Property JSON Key   Default     Description
======== ========== =========== =============
Mode     mode       ``steps``   Determines where to place pips
Values   values                 Additional options for Positions, Count and Values mode. For Count mode provide an integer. For Positions and Values mode provide comma-separated integers.
Stepped  stepped    ``false``   Match pips with slider values
Density  density    ``1``       Number of pips in one percent of the slider range
Format   format     ``true``    Show pip values in the same format as slider format
======== ========== =========== =============

Specific Events
---------------

Events generated exclusively by the Slider Widget (see `Parameter-based Interaction <https://digitalhub.readthedocs.io/en/latest/docs/vis/cyclotron_parameters.html>`_). They produce a value that will be stored in the given Parameter. Note that Parameters must be defined beforehand in the Parameters section of the dashboard.
If the slider has two handles, you can use the *Section* property to specify which handle will generate the event.

================== ==========
Event              Value
================== ==========
``dateTimeChange`` Date or time selected with a handle
================== ==========

Examples
--------

Basic slider with four pips matching slider values
**************************************************

::

  {
      "maxValue": "2018-03-30",
      "minValue": "2017-09-10",
      "momentFormat": "YYYY-MM-DD",
      "pips": {
          "density": 2,
          "mode": "count",
          "stepped": true,
          "values": "4"
      },
      "widget": "slider"
  }

Vertical slider with Player and Tooltips
****************************************

::

  {
      "direction": "rtl",
      "maxValue": "2018-03-30",
      "minValue": "2017-09-10",
      "momentFormat": "YYYY-MM-DD",
      "orientation": "vertical",
      "player": {
          "interval": 2,
          "showPlayer": true
      },
      "tooltips": true,
      "widget": "slider"
  }

Slider with two handles and Parameter generation
************************************************

::

  {
      "maxValue": "2017-09-10 23:00",
      "minValue": "2017-09-10 00:00",
      "momentFormat": "YYYY-MM-DD HH:mm",
      "specificEvents": [{
          "event": "dateTimeChange",
          "paramName": "lowerValue",
          "section": "first"
      }, {
          "event": "dateTimeChange",
          "paramName": "upperValue",
          "section": "second"
      }],
      "timeUnit": "hours",
      "tooltips": true,
      "twoHandles": true,
      "widget": "slider"
  }
