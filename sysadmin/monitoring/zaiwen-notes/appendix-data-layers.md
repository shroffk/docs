## Layers of Monitoring Data
     
<img width="402" height="151" alt="image" src="https://github.com/user-attachments/assets/816d604f-4d8a-414a-bca0-4537ca92d699" />

      
---

# 1️⃣ Infrastructure (host-level health)

This is NOT EPICS-specific, but it directly impacts EPICS reliability.

## What to monitor:
- CPU usage  
- memory usage  
- disk I/O / space  
- network I/O / latency  
- process status (IOC running or not)  

## Tools:
- Prometheus  

## NOTE: 

> EPICS IOC (Quick Summary)    
> An EPICS IOC (Input/Output Controller) is a software process running on a Linux machine that interfaces with hardware and serves EPICS data. At the infrastructure level, it is treated like any other service to ensure it is running and stable.  
>
> IOC responsibilities:  
> interfaces with hardware (devices, sensors, controllers)  
> processes control logic  
> exposes data as PVs (Process Variables)    
>   
> Example IOC distribution   

<img width="368" height="216" alt="image" src="https://github.com/user-attachments/assets/6091f700-e673-4e03-8658-fa14baa48eec" />


---

# 2️⃣ EPICS System Health (IOC / control layer)

IOC acts as a PV server and exposes system state via Channel Access.

## What to monitor:

### IOC status
- running state  
- uptime  
- restart frequency  

### Channel Access / PV connectivity
- connected clients  
- disconnected PVs  
- connection errors  

### Performance
- scan rates  
- processing delays  
- queue backlogs  

### Alarms
- MAJOR / MINOR / INVALID  
- alarm frequency  
- unacknowledged alarms  

### Gateway / services
- CA gateway status  
- pvAccess services  
- archiver health  

## Tools:
iocStats (devIocStats)
https://github.com/epics-modules/iocStats

Provides:
- CPU usage per task  
- file descriptors  
- CA clients / connections  
- record counts  
- memory usage  
- restart control  
- EPICS environment variables  

---

# 3️⃣ EPICS Data (PV layer — physics / operations)

This is the core operational and physics data layer.

## What to monitor:

### PV values
- beam current  
- magnet settings  
- temperatures  
- voltages  

### Trends
- time evolution  
- drift and stability  

### Thresholds
- out-of-range values  
- safety limits  

### Derived metrics
- rates of change  
- correlations between PVs  

## Tools:
- EPICS Archiver Appliance + Grafana plugin  
- InfluxDB (optional alternative)  

---

# 4️⃣ Logs / Events (observability layer)

## What to monitor:
- IOC logs (errors, warnings)  
- startup/shutdown events  
- communication failures  
- device errors  
- network issues  

## Tools:
- Logstash  
- Elasticsearch  

---

# 📊 Full Layered Architecture

<img width="449" height="470" alt="image" src="https://github.com/user-attachments/assets/b20cad58-ccb5-4e4d-ab5e-4bf248f964c0" />

---

