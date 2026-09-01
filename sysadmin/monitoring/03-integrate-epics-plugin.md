# EIC Monitoring Deployment - Integrate Epics plugin to Grafana


## Overview

We will use EPICS Archiver Appliance Datasource plugin for Grafana dashboard
 
It’s a Grafana datasource plugin that reads EPICS data from the Archiver Appliance

**Features**

- Select multiple PVs by using Regex (Only supports wildcard pattern like PV.* and alternation pattern like PV(1|2))
- Legend alias with regular expression pattern
- Data retrieval with data processing (See Archiver Appliance User Guide for processing of data)
- Using PV names for Grafana variables
- Transform your data with processing functions
- Live update with stream feature
- Find and notify problems with alerting feature

Reference documentation:   
https://github.com/sasaki77/archiverappliance-datasource    
https://sasaki77.github.io/archiverappliance-datasource/master/index.html

---

## Detail Steps   

**I. Install Plugin with Grafana CLI**

- Install latest version. You can also use this command to update the plugin to the latest version.
  ```bash
  grafana cli --pluginUrl https://github.com/sasaki77/archiverappliance-datasource/releases/latest/download/archiverappliance-datasource.zip plugins install sasaki77-archiverappliance-datasource
  ```
    <img width="979" height="308" alt="image" src="https://github.com/user-attachments/assets/34947321-0e0a-4f92-bd1d-cf7b5c0ab353" />
    
- This plugin is unsigned. It must be specially listed by name in the Grafana grafana.ini file to allow Grafana to use it.
  - Add sasaki77-archiverappliance-datasource to the allow_loading_unsigned_plugins parameter in the [plugins] section.
    ```bash
    # Enter a comma-separated list of plugin identifiers to identify plugins to load even if they are unsigned. Plugins with modified signatures are never loaded.
    allow_loading_unsigned_plugins = sasaki77-archiverappliance-datasource
    ```

  - See Configure Grafana | Grafana documentation (https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/) for more detail on grafana.ini. 
   


**II. Configuration**

- Add New Archiver Appliance Datasource
  - In the left side menu, open Connections -> Data sources
  - Click Add new data source on the right top corner
  - Select ArchiverAppliacne in the list
 

- Data Source Settings
  - Connection
    - URL: Sets the retrieval endpoint (must end with `/retrieval`)
  - Data Retrieval Options
    - Use Backend: Enables the Go backend to retrieve archive data for visualization. The archived data is processed on the Grafana server, then sent to the Grafana client.
    - Hide Invalid: Hides sample data with invalid severity and null values. This option is only effective when backend data retrieval is enabled.

        <img width="1307" height="485" alt="image" src="https://github.com/user-attachments/assets/f3dce3b0-7846-4a5e-9d04-38b787378f96" />

 

**III. Query Edit**



---
