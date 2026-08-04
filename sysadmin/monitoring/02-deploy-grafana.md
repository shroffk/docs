# EIC Monitoring Deployment - Deploy Grafana

## Overview  

Grafana provides a centralized, web-based visualization layer for the EIC monitoring stack.  
It enables users to explore system and service metrics through interactive dashboards and supports alerting for operational awareness.

In this deployment, Grafana will serve as the primary interface for observability data collected by Prometheus.

Key Functions  
- Connect to Prometheus as a primary data source
- Visualize infrastructure and application metrics in real time
- Provide customizable dashboards for Linux system monitoring (CPU, memory, disk, network)
- Support alerting based on metric thresholds (optional future enhancement)
- Enable a unified view of multiple hosts and services in the EIC environment

Architecture Role  
- Grafana acts as the presentation layer in the monitoring stack:
  ```bash
  Node Exporter → Prometheus → Grafana
  (metrics)       (storage)    (visualization)
  ```
  
Reference documentation:   
https://grafana.com/grafana/download?edition=oss    


---

## Detail Steps   

**I. Install Grafana**

- Download and extract Grafana
  ```bash
  #Standalone Linux Binaries
  cd /eic/opt/packages/grafana
  wget https://dl.grafana.com/grafana/release/13.0.1/grafana_13.0.1_24542347077_linux_amd64.tar.gz
  tar -zxvf grafana_13.0.1_24542347077_linux_amd64.tar.gz

  #Or yum install rpm
  yum install -y https://dl.grafana.com/grafana/release/13.0.1/grafana_13.0.1_24542347077_linux_amd64.rpm
  ```

- Start Grafana service
  ```bash
  systemctl enable grafana-server
  systemctl start grafana-server
  systemctl status grafana-server
  ```

- Open Grafana UI
  - From your browser: http://your-server-hostname:3000
  - Default login:
    - Username: admin
    - Password: admin (you’ll be forced to change it)    

**II. Connect Prometheus**
  
- Inside Grafana:
  - Left menu -> Connections -> Add new connection
  - Search for and select Prometheus
  - Click "Add new data source"
  - Configure the data source: http://prometheus-host:9090
  - Click Save & test
    <img width="1143" height="619" alt="image" src="https://github.com/user-attachments/assets/992dbf43-cf84-4f89-bf8a-a5ff80983205" />

- Import your first dashboard (IMPORTANT)
  - Left menu → Dashboards
  - Click New → Import
  - In the “Import via grafana.com” box, fill in Node Exporter ID: 1860
  - Click Load
  - Click Import
  - This gives you:
    - CPU usage
    - Memory usage
    - Disk usage and I/O
    - Network traffic
    - System load
      <img width="1743" height="755" alt="image" src="https://github.com/user-attachments/assets/b0be262e-4ed8-48c6-bbab-d12da8aa113b" />
   



---
