https://github.com/sasaki77/archiverappliance-datasource    

EPICS Archiver Appliance Datasource  
Visualize EPICS Archiver Appliance on Grafana

---

It’s a Grafana datasource plugin that reads EPICS data from the Archiver Appliance

**Features**

- Select multiple PVs by using Regex (Only supports wildcard pattern like PV.* and alternation pattern like PV(1|2))
- Legend alias with regular expression pattern
- Data retrieval with data processing (See Archiver Appliance User Guide for processing of data)
- Using PV names for Grafana variables
- Transform your data with processing functions
- Live update with stream feature
- Find and notify problems with alerting feature


**Explanation**

The Archiver Appliance datasource plugin allows Grafana to directly query archived EPICS PV data, so we can keep EPICS data in its native archiving system and integrate it at the visualization layer.  

This avoids pushing EPICS data into Prometheus and keeps the architecture loosely coupled.  

The trade-off is that it introduces a dependency on an external plugin and separate data sources, so we’d need to evaluate its maturity and long-term support.

**Analysis**

**Step 1 — Architecture**
              
<img width="544" height="350" alt="image" src="https://github.com/user-attachments/assets/0c6ad181-7577-496f-8f89-415d4ecba73f" />

**Step 2  — What data we actually get**  

- The plugin works with EPICS PVs (Process Variables).
- From the repo:
  - You query using PV names
  - You can select multiple PVs using regex like:
    - PV.*
    - (PV1|PV2)
  - So in Grafana, instead of Prometheus queries like:
    - node_cpu_seconds_total
  - You’ll write something like:
    - SR:C01-BI:G2{CURRENT}


**Step 3 — Important features **

- Native Grafana integration
  - Appears as a normal Grafana datasource
  - Can be used in dashboards alongside Prometheus

- Streaming (real-time updates)
  - Supports live updates without refreshing dashboards

- Built-in data processing
  - Supports operators like:
    - mean
    - raw
    - last

- Variables support (very important for dashboards)
  - You can dynamically select PVs using variables
  - Example:
    - ${group}:.*

- Alerting support
  - “Find and notify problems with alerting feature”
  - BUT subtle point:
    - This is Grafana-level alerting, not EPICS alarm system

