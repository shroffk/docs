https://proceedings.jacow.org/icalepcs2019/papers/wepha134.pdf  

MONITORING SYSTEM FOR IT INFRASTRUCTURE AND EPICS CONTROL SYSTEM AT SuperKEKB

---

The monitoring system for metrics and logs have been deployed. Visualized data helps us to understand the current and past status of the control system. Notifications of a system problem from Zabbix allows early response to its problem.

In order to expand monitoring system, we have developed EPICS PV monitoring system with Zabbix and data visualization system with pvAccess RPC and Grafana. These systems provide consolidated monitoring for various types of data. We have applied it for monitoring IOC performance and status monitoring such as CSS alarm status.

<img width="668" height="420" alt="image" src="https://github.com/user-attachments/assets/ca5493aa-216f-484c-9b7c-b55256a35ccf" />

---

**What the paper actually implemented**

From the paper, there are indeed two distinct EPICS integration paths:

1. EPICS PV → Zabbix (metrics integration)   

- They built:
  - “an EPICS Channel Access client application that sends PV values to Zabbix server”
- What this means
  - They treat EPICS PVs as metrics
  - They push those PV values into Zabbix
  - Then:
    - Zabbix stores them
    - Grafana visualizes them (via Zabbix plugin)
- Architecture
  - EPICS PVs → Channel Access client → Zabbix → Grafana
- Purpose
  - Monitor IOC health
  - Examples:
    - CPU usage
    - memory usage
    - number of CA clients  
- This is operational monitoring of EPICS itself, not physics data analysis


2. pvAccess RPC → Grafana (custom visualization path)   

- They also built:
  - “Grafana datasource plugin and HTTP / pvAccess API gateway to visualize the data from pvAccess RPC servers”
- What this means
  - They created:
    - a custom Grafana datasource plugin
    - an API gateway (HTTP + pvAccess)
  - This allows Grafana to directly query:
    - pvAccess RPC servers
    - structured EPICS data (e.g., alarm tables)
- Architecture
  - EPICS (pvAccess RPC) → API Gateway → Grafana plugin → Grafana
- Purpose
  - Visualize arbitrary EPICS data
  - Example:
    - CSS alarm status tables
    - structured control system data

---

**Key difference between the two paths**

| Path                     | Data Type              | Purpose                         | Integration Style                          |
|--------------------------|------------------------|----------------------------------|---------------------------------------------|
| EPICS → Zabbix           | PV metrics             | IOC/system monitoring           | reuse existing monitoring tool             |
| pvAccess RPC → Grafana   | structured EPICS data  | visualization (alarms, tables)  | custom plugin + API                        |


**Final takeaway**

✔ Yes — they built two integrations
✔ One = metrics (Zabbix)
✔ One = custom visualization (Grafana plugin + API)
✔ This is an early-stage, multi-path integration approach

