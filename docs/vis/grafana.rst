Grafana
===========

| Grafana is the open-source platform for monitoring and observability.
| Grafana allows you to query, visualize, alert on and understand your metrics no matter where they are stored. 
| Create, explore, and share dashboards with your team and foster a data driven culture.
| It is a tool for beautiful monitoring and metric analytics & dashboards for Graphite, InfluxDB, Prometheus and more

The official Grafana documentation is available at the following link: https://grafana.com/docs/grafana/latest/

Installation
--------------

| It is possible to install Grafana in different modes on different systems. 
Referring to the documentation https://grafana.com/docs/grafana/latest/installation/ , Grafana is very easy to install and run using the official Docker container::

		$docker run -d -p 3000:3000 grafana/grafana


Features
-----------

- **Visualize**: Fast and flexible client side graphs with a multitude of options. Panel plugins for many different way to visualize metrics and logs.
- **Dynamic Dashboards**: Create dynamic & reusable dashboards with template variables that appear as dropdowns at the top of the dashboard.
- **Explore Metrics**: Explore your data through ad-hoc queries and dynamic drilldown. Split view and compare different time ranges, queries and data sources side by side.
- **Explore Logs**: Experience the magic of switching from metrics to logs with preserved label filters. Quickly search through all your logs or streaming them live.
- **Alerting**: Visually define alert rules for your most important metrics. Grafana will continuously evaluate and send notifications to systems like Slack, PagerDuty, VictorOps, OpsGenie.
- **Mixed Data Sources**: Mix different data sources in the same graph! You can specify a data source on a per-query basis. This works for even custom datasources.

Concepts
---------


Dashboard
"""""""""""
| The dashboard is where it all comes together. A dashboard is a set of one or more panels organized and arranged into one or more rows.
| The time period for the dashboard can be controlled by the Time range controls in the upper right of the dashboard.
| Dashboards can use templating to make them more dynamic and interactive.
| Dashboards can use annotations to display event data across panels. This can help correlate the time series data in the panel with other events.
| Dashboards can be shared easily in a variety of ways.
| Dashboards can be tagged, and the dashboard picker provides quick, searchable access to all dashboards in a particular organization.

Data source
""""""""""""""""""""""
| Grafana can visualize, explore, and alert on data from many different databases and cloud services. Each database or service type is accessed from a data source.
| Each data source has a specific query editor that is customized for the features and capabilities that the particular data source exposes. The query language and capabilities of each data source are | | obviously very different. You can combine data from multiple data sources into a single dashboard, but each panel is connected to a specific data source that belongs to a particular organization.

Organization
""""""""""""""""""""""
| Grafana supports multiple organizations in order to support a wide variety of deployment models, including using a single Grafana instance to provide service to multiple potentially untrusted organizations.
| Each organization can have one or more data sources.
| All dashboards are owned by a particular organization.
| Note: Most metric databases do not provide per-user series authentication. This means that organization data sources and dashboards are available to all users in a particular organization.

Panel
"""""""""""
| The panel is the basic visualization building block in Grafana. Each panel has a Query Editor specific to the data source selected in the panel. 
| The query editor allows you to extract the perfect visualization to display on the panel.
| There are a wide variety of styling and formatting options for each panel. Panels can be dragged and dropped and rearranged on the Dashboard. They can also be resized.
| Panels can be made more dynamic with Dashboard Templating variable strings within the panel configuration. The template can include queries to your data source configured in the Query Editor.
| Panels can be shared easily in a variety of ways.

Query editor
"""""""""""
| The query editor exposes capabilities of your data source and allows you to query the metrics that it contains.
| Use the query editor to build one or more queries in your time series database. The panel instantly updates, allowing you to effectively explore your data in real time and build a perfect query for that particular panel.
| You can use template variables in the query editor within the queries themselves. 
| This provides a powerful way to explore data dynamically based on the templating variables selected on the dashboard.
| Grafana allows you to reference queries in the query editor by the row that they’re on. If you add a second query to graph, you can reference the first query by typing in #A. This provides an easy and convenient way to build compound queries.

Row
"""""""""""
| A row is a logical divider within a dashboard. It is used to group panels together.
| Rows are always 12 “units” wide. These units are automatically scaled dependent on the horizontal resolution of your browser. 

User
"""""""""""
| A user is a named account in Grafana. 
| A user can belong to one or more organizations and can be assigned different levels of privileges through roles.
| Grafana supports a wide variety of internal and external ways for users to authenticate themselves. 
| These include from its own integrated database, from an external SQL server, or from an external LDAP server.


Integration
------------


.. toctree::
  grafana_integrate

