Google Charts Widget
====================

The Google Charts Widget uses the wrapper directive for Google Chart Tools `angular-google-charts <http://angular-google-chart.github.io/angular-google-chart/>`_ to create several types of charts.

Configuration
-------------

This widget requires a Data Source to provide data for the chart. The following table illustrates the configuration properties available (required properties are marked in **bold**):

=============== ================ ========= =================
Property        JSON Key         Default   Description
=============== ================ ========= =================
**Data Source** **dataSource**             Name of the Data Source providing table-like data for this Widget
**Chart Type**  **chartType**              Type of the chart
Options         options                    JSON object with configuration options for the chart. Refer to the `Google Charts API documentation`_ of the selected chart type to see the list of available options.
Columns         columns                    Array of columns of the table-like dataset. If not provided, the column labels will be inferred from the keys of the first dataset object. Note that this property must be used if some columns contain dates, times or datetimes or if the column role should differ from "data" (e.g. for annotations; see `roles`_).
Formatters      formatters                 Optional JavaScript functions that will be applied to all the values of the specified datasource columns
=============== ================ ========= =================

Options
*******

Options property provides configuration options for the chart. Please refer to the `Google Charts API documentation <https://developers.google.com/chart/interactive/docs/gallery>`_ of the selected chart type to see the list of available options.

Columns
*******

Columns property specifies how to read dataset columns, i.e., their names and types. While columns with strings, numbers and booleans can be easily identified, this property is useful for datasets containing dates or times, which are otherwise not recognized by the directive.

======== ======== ========== =========================
Property JSON Key Default    Description
======== ======== ========== =========================
**Name** **name**            Indicates the name of the data source field to use for this column
Type     type     ``string`` Type of the column values
Role     role     ``data``   Optional role of the column
======== ======== ========== =========================

Note that columns with role "annotation" must immediately follow the series column that should be annotated, and columns with role "annotationText" must follow the annotated column.

Formatters
**********

Formatters property specifies an array of JavaScript functions to format data in a given dataset column.

=========== ========== =========================
Property    JSON Key   Description
=========== ========== =========================
Column Name columnName Indicates the name of the datasource column
Formatter   formatter  The function must take a single value as argument and return the new value
=========== ========== =========================

In the following example, dates in the column named Day are given a different datetime format:

::

  "formatters": [{
      "columnName": "Day",
      "formatter": "p = function(value){ return moment(value, 'YYYY-MM-DD').format('dd DD'); }"
  }]

Examples
--------

Basic column chart
******************

JavaScript Datasource:

- Name: prices
- Processor:

::

  e = function(promise){
      var data = [{
          Product: 'Book',
          Price: 15.00
      },{
          Product: 'Pencil',
          Price: 0.50
      },{
          Product: 'Water bottle',
          Price: 1.50
      },{
          Product: 'Bread',
          Price: 2.0
      },{
          Product: 'Cake',
          Price: 20.00
      }]

      return promise.resolve(data);
  }

Google Charts widget:

::

  {
      "chartType": "ColumnChart",
      "dataSource": "datasource_3",
      "options": "{\n    \"legend\": {\"position\": \"bottom\"},\n}",
      "widget": "gchart"
  }



.. _Google Charts API documentation: https://developers.google.com/chart/interactive/docs/gallery
.. _roles: https://developers.google.com/chart/interactive/docs/roles
