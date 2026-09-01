# EIC Monitoring Architecture Proposal

## Overview

We propose a unified monitoring architecture using Prometheus for infrastructure and service monitoring with alerting, while Grafana provides a single dashboard layer.     

- Linux system monitoring (CPU, memory, storage, network I/O) will be collected via Prometheus (https://prometheus.io/docs/prometheus/latest/installation/).
- EPICS IOS Status will be monitored using "iocStats - EPICS IOC Status and Control" (https://github.com/epics-modules/iocStats).
- EPICS PV data will be integrated via the EPICS Archiver Appliance using the archiverappliance-datasource plugin (https://github.com/sasaki77/archiverappliance-datasource).
- EPICS logs will be centralized using Logstash and Elasticsearch, following the LCLS-II experiment control system approach (https://proceedings.jacow.org/icalepcs2023/papers/th2bco03.pdf).

This creates a scalable **single pane of glass** that unifies metrics, EPICS data, and logs for complete observability.   

---

## Architecture Summary

This architecture separates monitoring into four layers:

### 1. Metrics & Alerting (Infrastructure)

- Prometheus collects:
  - Linux system metrics (CPU, memory, disk, network)
  - IOC process status
  - service health (archiver, gateway, exporters)

- Alerting is handled via Alertmanager

---

### 2. EPICS IOC Status (Control Layer)

Using: iocStats - EPICS IOC Status and Control (https://github.com/epics-modules/iocStats)     

devIocStats exposes IOC internal metrics as EPICS PVs, providing visibility into runtime behavior and performance.

Key capabilities:
- CPU usage by IOC tasks  
- number of Channel Access (CA) clients and connections  
- number of records on the IOC  
- file descriptor usage  
- suspended tasks  
- memory fragmentation (largest free block)  
- EPICS environment variables  
- IOC restart control/status  

These metrics provide deeper insight into IOC health beyond simple process monitoring and can be visualized in Grafana or integrated into the monitoring pipeline.

---

### 3. EPICS Data (PVs)

Using: EPICS Archiver Appliance plugin for Grafana dashboard (https://github.com/sasaki77/archiverappliance-datasource)  

This Visualizes EPICS Archiver Appliance on Grafana:   
- historical trends  
- real-time values  
- system behavior over time  

---

### 4. Logs & Events (Observability Layer)

Following the approach in "THE LCLS-II EXPERIMENT CONTROL SYSTEM using Logstash and Elasticsearch" (https://proceedings.jacow.org/icalepcs2023/papers/th2bco03.pdf)      

- Logs from IOCs and services are collected  
- Processed via Logstash  
- Stored in Elasticsearch  
- Visualized in Grafana   

This enables:
- centralized log search  
- troubleshooting  
- correlation with metrics and PV data  

---

## Key Idea

Grafana acts as a **single pane of glass**, combining:

- Prometheus metrics  
- EPICS IOC status (iocStats)  
- EPICS PV data via Archiver  
- Logs via Logstash/Elasticsearch  

---

## Outcome

This design provides:

- clear separation of concerns  
- modern, scalable monitoring (Prometheus-based)  
- seamless EPICS integration without custom development  
- unified observability across infrastructure, control system, and logs  

---

## Monitoring Architecture Diagram

**Simple Version**   
<img width="564" height="521" alt="image" src="https://github.com/user-attachments/assets/7735515a-00d5-4693-8df0-69c6313d0723" />


**Comprehensive Version**    
<img width="923" height="621" alt="image" src="https://github.com/user-attachments/assets/130e4851-02c0-44f9-828e-086c64dd0d86" />

---

 ## Next Steps

1. Deploy Prometheus + Alertmanager  
2. Deploy Grafana  
3. Integrate Archiver Appliance datasource plugin  
4. Enable iocStats on IOCs  
5. Define dashboards and alerts  
6. (Optional) Implement centralized logging pipeline  

---                        
