************************************************
Visualization, Dashboards, and Data Applications
************************************************

Data visualization is used to provide an intuitive insight on the data phenomena that has been elaborated on top of platform data and services. The visualization is provided in a form of various dashboards and dashboard-based applications and may be used in different scenarios.
Common to the visualization problem are the following requirements:

-	Possibility to have a wide range of visualization types (charts, maps) and interactions (filters, details, drill down)
-	Possibility to control the access to the dashboards, ranging from public to private based on the user roles and permissions. Related to this is the possibility to share the dashboard.
-	Integration with the other platform components, both for data sources and for access control.  

Visualization tools that can access Digital Hub may be divided in two categories:

-	Tools that work directly with data sources, such as “online” storages (e.g., PostgreSQL) or data lake (e.g., file in MinIO). 
This includes tools sucha SQLPad and Grafana.
-	Tools that connects to the services and data exposed as “Data Interfaces”: DSS, GeoService, API REST, etc. For this purpose the platform adopts Cyclotron.


.. toctree::
  data/sqlpad
  vis/grafana
  vis/cyclotron
  