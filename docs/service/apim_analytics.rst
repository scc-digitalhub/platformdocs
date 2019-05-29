API Analytics
=========================

In order to enable and configure the API-M analytics for API Publisher and Subscriber, follow the `official instructions <https://docs.wso2.com/display/AM210/Analytics>`_.

Some important performance tricks:

* Disable Analytics indexing. This can be done adding the following parameter to the Analytics start command: ``-DdisableIndexing=true``
* Change the default configuration for the anaytics scripts: ``Analytics Carbon Console > Manage > Batch Analysis > Scripts``. Set the appropriate CRON configurations for the script execution (e.g., each hour ``0 0 * ? * * *`` instead of each 2 mins). Remember that the configurations are reset after the server restart