# EIC Monitoring Deployment - Monitor Logs & Events

## Overview

We will follow the approach in "THE LCLS-II EXPERIMENT CONTROL SYSTEM using Logstash and Elasticsearch"      

- Logs from IOCs and services are collected  
- Processed via Logstash  
- Stored in Elasticsearch  
- Visualized in Grafana   

This enables:
- centralized log search  
- troubleshooting  
- correlation with metrics and PV data  

Reference documentation:   
https://proceedings.jacow.org/icalepcs2023/papers/th2bco03.pdf  

---

## Part 1 — Log Sources (IOC & Services)

### IOC Logs
- EPICS IOC console output
- Application-specific logs
- Startup and runtime messages

### System & Service Logs
- Linux system logs (`/var/log/messages`, `syslog`, etc.)
- Service logs (e.g., Archiver Appliance, Prometheus, Grafana)

### Responsibilities
- **EPICS Engineer**
  - Ensure IOC logs are properly generated and accessible
- **SysAdmin**
  - Ensure system logs are available and readable
  - Standardize log locations where possible

---

## Part 2 — Log Collection & Processing (Logstash)

### Tasks
- Configure Logstash to ingest logs from:
  - Files (e.g., `/var/log/...`)
  - Remote sources (if applicable)
- Define pipelines to:
  - Parse log formats
  - Extract fields (timestamp, severity, IOC name, etc.)
  - Normalize data structure

### Example Processing Goals
- Convert raw logs into structured JSON
- Tag logs by source (IOC name, hostname, service)
- Filter unnecessary or noisy entries

### Responsibilities
- **SysAdmin**
  - Deploy and manage Logstash
  - Maintain pipeline configurations

---

## Part 3 — Storage & Indexing (Elasticsearch)

### Tasks
- Store processed logs in Elasticsearch indices
- Define index patterns (e.g., `ioc-logs-*`, `system-logs-*`)
- Configure retention policies (index lifecycle management)

### Responsibilities
- **SysAdmin**
  - Deploy and maintain Elasticsearch cluster
  - Manage storage, performance, and retention

---

## Part 4 — Visualization & Analysis (Grafana)

### Tasks
- Configure Elasticsearch as a data source in Grafana
- Build dashboards for:
  - Log search and filtering
  - Error tracking
  - IOC-specific log views
- Correlate logs with:
  - Prometheus metrics
  - EPICS Archiver Appliance data

### Responsibilities
- **SysAdmin**
  - Configure Grafana data sources
  - Create and maintain dashboards

---

## Benefits

- Centralized visibility across all IOCs and services
- Faster root-cause analysis
- Ability to correlate:
  - Logs (events)
  - Metrics (Prometheus)
  - PV data (Archiver)
- Scalable and extensible architecture

---

## Notes

- Time synchronization (e.g., NTP) is critical across all systems
- Log format standardization greatly improves search and correlation
- Consider log retention and storage sizing early in the design

---

**Appendix - How to Deploy and manage Logstash**

**Step 1 — Install Logstash**

- Logstash is part of the Elastic Stack.
  ```bash
  sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
  
  cat <<EOF | sudo tee /etc/yum.repos.d/logstash.repo
  [logstash-8.x]
  name=Elastic repository for 8.x packages
  baseurl=https://artifacts.elastic.co/packages/8.x/yum
  gpgcheck=1
  gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
  enabled=1
  autorefresh=1
  type=rpm-md
  EOF
  
  sudo dnf install logstash -y
  ```

**Step 2 — Understand Where Config Lives**

- Main pipeline config directory:
  - /etc/logstash/conf.d/
- You will create files like:
  - /etc/logstash/conf.d/eic-logs.conf


**Step 3 — Create Your FIRST Pipeline (Simple!)**

- Create config file
  ```bash
  sudo vi /etc/logstash/conf.d/eic-logs.conf
  Paste this:
    
    input {
      file {
        path => "/var/log/messages"
        start_position => "beginning"
        sincedb_path => "/dev/null"
      }
    }
    
    filter {
    }
    
    output {
      stdout {
        codec => rubydebug
      }
    }
  ```

 **Step 4 — Start Logstash**

```bash
sudo setfacl -m u:logstash:r /var/log/messages
sudo systemctl enable logstash
sudo systemctl start logstash
sudo journalctl -u logstash -f
sudo /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/eic-logs.conf
```

**Step 5 — Verify It Works**

  You should see output like:
  
  {
    "message" => "Apr 29 10:00:01 server sshd[1234]: Accepted password...",
    "host" => "your-host",
    "@timestamp" => ...
  }
  
  If you see this → ✅ SUCCESS



**Step 6 — Add Another Log Source (IOC Example)**

- Now simulate your EPICS IOC logs. 
- Let’s say they are in: /opt/epics/ioc/logs/ioc1.log
- Update config:
  ```bash
  input {
    file {
      path => "/var/log/messages"
      start_position => "beginning"
      sincedb_path => "/dev/null"
    }
  
    file {
      path => "/opt/epics/ioc/logs/*.log"
      start_position => "beginning"
      sincedb_path => "/dev/null"
    }
  }
  ```
- Restart:
  ```bash
  sudo systemctl restart logstash
  ```

 **Step 7 — Add Basic Tagging (VERY IMPORTANT)**

 - Now we make logs distinguishable:
   ```bash
    input {
      file {
        path => "/var/log/messages"
        type => "system"
      }
    
      file {
        path => "/opt/epics/ioc/logs/*.log"
        type => "ioc"
      }
    }
    
    filter {
    }
    
    output {
      stdout {
        codec => rubydebug
      }
    }
   ```

**Step 8A — Parse System Logs (syslog)**

```bash
sudo vi /etc/logstash/conf.d/eic-logs.conf

input {
  file {
    path => "/var/log/messages"
    start_position => "beginning"
    sincedb_path => "/dev/null"
    add_field => { "log_type" => "system" }
  }

  file {
    path => "/opt/epics/ioc/logs/*.log"
    start_position => "beginning"
    sincedb_path => "/dev/null"
    add_field => { "log_type" => "ioc" }
  }
}

filter {
  if [log_type] == "system" {
    grok {
      match => {
        "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{HOSTNAME:hostname} %{DATA:program}(?:\\[%{POSINT:pid}\\])?: %{GREEDYDATA:log_message}"
      }
    }
  }
}
```

**Step 8B — Clean Up (Optional but Good)**

Add this to remove the original noisy field:

```bash
filter {
  if [type] == "system" {
    grok {
      match => {
        "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{HOSTNAME:hostname} %{DATA:program}(?:\\[%{POSINT:pid}\\])?: %{GREEDYDATA:log_message}"
      }
    }

    mutate {
      remove_field => ["message"]
    }
  }
}
```

**Step 9 — Add Timestamp Normalization (Important)**

```bash
filter {
  if [type] == "system" {
    grok {
      match => {
        "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{HOSTNAME:hostname} %{DATA:program}(?:\\[%{POSINT:pid}\\])?: %{GREEDYDATA:log_message}"
      }
    }

    date {
      match => ["syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss"]
    }

    mutate {
      remove_field => ["message"]
    }
  }
}
```

**Step 10 — Prepare for IOC Logs (Important Design Step)**   
