 
https://proceedings.jacow.org/icalepcs2021/papers/mobl04.pdf  

KARABO DATA LOGGING: InfluxDB BACKEND AND GRAFANA UI  

---


This 2021 paper shows a time-series database approach where EPICS PV updates are collected via a dedicated data logging layer and stored in InfluxDB, with Grafana used purely as a visualization layer.  

InfluxDB + Grafana used for EPICS-based control system data logging and visualization.  

**Core architecture:**

Control System (Karabo / EPICS-like devices)  
↓  
Data Logger (EPICS PV / device updates)  
↓  
InfluxDB (time-series storage)  
↓  
Grafana (visualization + dashboards)  


**What makes this architecture different**

- They use a dedicated data logging layer
  - They introduce a system of:
    - data logger devices
    - distributed collectors
    - message broker system
  - So EPICS-like devices do NOT directly talk to InfluxDB
    - There is a middleware logging system in between

- Central time-series database
  - InfluxDB is used as:
    - unified storage for all PV updates
    - long-term historical storage (years of data)
    - optimized for time-series queries

- Grafana is again just the UI layer
  - Grafana is used for:
    - dashboards
    - trend analysis
    - operational monitoring
  - But it does NOT directly talk to EPICS

---
 <img width="502" height="466" alt="image" src="https://github.com/user-attachments/assets/1679624e-2bb8-414e-bc1f-997a22bb5be4" />


