# EIC Monitoring Deployment - Deploy Prometheus


## Overview

We propose a unified monitoring architecture built around Prometheus for metrics collection, alerting, and time-series storage, with Grafana providing a centralized and user-friendly dashboard layer.  

Linux system metrics (CPU, memory, storage, and network I/O) will be collected by Prometheus using exporters (e.g., node-level collectors).   
Prometheus will handle scraping, storage, and alert evaluation, enabling proactive monitoring and issue detection.  

Reference documentation:   
https://prometheus.io/docs/prometheus/latest/installation/   

---

## Detail Steps   

**I. Install Prometheus**

- Download Prometheus
  - Go to the official releases page: https://prometheus.io/download/
  - Or directly from CLI:
    ```bash
    cd /eic/opt/packages/prometheus/
    wget https://github.com/prometheus/prometheus/releases/download/v3.11.2/prometheus-3.11.2.linux-amd64.tar.gz
    ```

- Extract it
  ```bash
  tar xvf prometheus-3.11.2.linux-amd64.tar.gz
  cd prometheus-3.11.2.linux-amd64
  ```

- Move files to proper locations
  ```bash
  mkdir /etc/prometheus
  mkdir /var/lib/prometheus
  cp prometheus promtool /usr/local/bin/
  cp prometheus.yml /etc/prometheus/
  ```

- Create a dedicated user (best practice)
  ```bash
  useradd --no-create-home --shell /bin/false prometheus
  ```
  
- Set permissions
  ```bash
  chown -R prometheus:prometheus /etc/prometheus
  chown -R prometheus:prometheus /var/lib/prometheus
  chown prometheus:prometheus /usr/local/bin/prometheus
  chown prometheus:prometheus /usr/local/bin/promtool
  ```

- Run Prometheus (manual test first)
  ```bash
  prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus
  ```
  <img width="819" height="103" alt="image" src="https://github.com/user-attachments/assets/f8df4430-84dc-4368-9d3a-9e040c9018f7" />
  

- Systemd Service Setup for Prometheus
  - To run Prometheus as a background service and ensure it starts automatically on boot, we configure it as a systemd service.
  - Create the systemd service file (template saved in /eic/opt/packages/prometheus/prometheus.service)
     ```bash
     gedit /etc/systemd/system/prometheus.service

     #Add the following content:
     
      [Unit]
      Description=Prometheus Monitoring System
      Wants=network-online.target
      After=network-online.target
      
      [Service]
      User=prometheus
      Group=prometheus
      Type=simple
      
      ExecStart=/usr/local/bin/prometheus \
        --config.file=/etc/prometheus/prometheus.yml \
        --storage.tsdb.path=/var/lib/prometheus \
        --web.listen-address=0.0.0.0:9090
      
      Restart=always
      
      [Install]
      WantedBy=multi-user.target
     ```

  - Reload systemd configuration
    ```bash
    systemctl daemon-reexec
    systemctl daemon-reload
    ```
  - Enable service at boot
    ```bash
    systemctl enable prometheus
    ```
   
  - Start Prometheus service
    ```bash
    systemctl start prometheus
    ```
    
  - Check service status
    ```bash
    systemctl status prometheus
    ```

- Verify
  - From your browser: http://your-server-hostname:9090
  - You should see the Prometheus UI 🎯
     <img width="857" height="536" alt="image" src="https://github.com/user-attachments/assets/c9c1c112-5976-47fa-9ed6-103a6e908862" />

---

**II. Install node_exporter (Linux metrics)**

- Explanation
  - Prometheus acts as the central brain that "pulls" (scrapes) data from targets, while Node Exporter is a lightweight agent that gathers hardware and OS metrics from a machine and exposes them for the server to collect.
  - Unlike most monitoring tools where the client "pushes" data to the server, Prometheus works on a Pull Model.
    - Node Exporter stays idle until it gets a request.
    - Prometheus reaches out to the Node Exporter every few seconds (the "scrape interval").
    - Node Exporter sends back a snapshot of the current system stats. 
    
- Download node_exporter
  ```bash
  cd /eic/opt/packages/prometheus
  wget https://github.com/prometheus/node_exporter/releases/download/v1.11.1/node_exporter-1.11.1.linux-amd64.tar.gz 
  ```

- Extract
  ```bash
  tar xvf node_exporter-1.11.1.linux-amd64.tar.gz 
  cd node_exporter-1.11.1.linux-amd64
  ```

- Install binary
  ```bash
  cp node_exporter /usr/local/bin/
  ```

- Create system user
  ```bash
  useradd --no-create-home --shell /bin/false node_exporter
  ```

- Create systemd service
  ```bash
  gedit /etc/systemd/system/node_exporter.service

  #Add the following content:
    [Unit]
    Description=Node Exporter
    Wants=network-online.target
    After=network-online.target
    
    [Service]
    User=node_exporter
    Group=node_exporter
    Type=simple
    ExecStart=/usr/local/bin/node_exporter
    
    Restart=always
    
    [Install]
    WantedBy=multi-user.target
  ```
    
- Start service
  ```bash
  systemctl daemon-reload
  systemctl start node_exporter
  systemctl enable node_exporter
  systemctl status node_exporter
  ```

- Test
    ```bash
    curl http://localhost:9100/metrics
    ```

**III. Add node_exporter to Prometheus**

- Edit Prometheus config
  ```bash
  vi /etc/prometheus/prometheus.yml

  #Add scrape job
  #Find the section: scrape_configs:
  #Add this new job under it:

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
   ```

- Validate config
  ```bash
  promtool check config /etc/prometheus/prometheus.yml
  ```

- Restart Prometheus
  ```bash
  systemctl restart prometheus
  ```

- Verify in UI
  - http://your-server-hostname:9090/targets
  - node_exporter = UP (green)

- Quick test query
  - In Prometheus UI → open **Query** page
  - Run the following example queries in the query input box:
    - `node_cpu_seconds_total`
    - `node_memory_MemAvailable_bytes`
  - After executing each query:
    - Switch the result view to **Graph** to visualize the time series
    - Optionally use **Table** view to inspect raw metric values    

       <img width="867" height="665" alt="image" src="https://github.com/user-attachments/assets/d1f4a688-2320-4245-b720-fab10613f3cf" />

