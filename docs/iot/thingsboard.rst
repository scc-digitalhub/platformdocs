ThingsBoard
==============

ThingsBoard represents a software platform that exposes the APIs to 
communicate with the devices and collect the data, to configure and manage 
the IoT applications (devices, events, commands, data routes) and perform basic IoT 
data monitoring activities. The collected data may be stored within the platform or 
routed to external storage (e.g., to the Data Lake via corresponding data stream connectors).

More specifically, the features of the ThingsBoard platform include

- Wide range of connectivity options, from REST API, to MQTT, CoAP, etc.
- Extensibility options for connectivity (e.g., LoRaWAN, NBIoT, etc), rule engine, server integrations.
- Provision devices, assets and customers and define relations between them.
- Collect and visualize data from devices and assets.
- Analyze incoming telemetry and trigger alarms with complex event processing.
- Control your devices using remote procedure calls (RPC).
- Build work-flows based on device life-cycle event, REST API event, RPC request, etc
- Design dynamic and responsive dashboards and present device or asset telemetry and insights to your customers
- Enable use-case specific features using customizable rule chains. 
- Push device data to other systems.

What is important for the DigitalHub platform and its architecture, ThingsBoard supports multitenancy
out of the box. This allows for defining centrally the ThingsBoard tenants using the correspnoding
admininstration API, create and associate users, and bootstrap applications automatically.

By default, DigitalHub considers ThingsBoard Community Edition distributed under Apache License. However, the 
possibility to integrate the Enterprise Edition is possible as well.

Full documentation of the platform is available here: https://thingsboard.io/docs/.

The installation and integration instructions are available under the platform :ref:`installation section <thingsboard-installation>`.


