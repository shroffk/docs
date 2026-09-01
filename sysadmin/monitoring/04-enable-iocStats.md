# EIC Monitoring Deployment - Enable EPICS iocStats on IOCs

## Overview

The `iocStats` module provides runtime statistics (CPU, memory, heartbeat, etc.) from EPICS IOCs.  
Enabling it requires changes inside each IOC, so responsibilities are split between the EPICS engineer and the Linux SysAdmin.

These metrics provide deeper insight into IOC health beyond simple process monitoring and can be visualized in Grafana.

Key capabilities:
- CPU usage by IOC tasks  
- number of Channel Access (CA) clients and connections  
- number of records on the IOC  
- file descriptor usage  
- suspended tasks  
- memory fragmentation (largest free block)  
- EPICS environment variables  
- IOC restart control/status  

Reference documentation:  
https://github.com/epics-modules/iocStats  
https://www.slac.stanford.edu/grp/ssrl/spear/epics/site/devIocStats/  

---

## Part 1 — EPICS Engineer Responsibilities (IOC-Level Configuration)

The EPICS engineer is responsible for integrating and enabling `iocStats` within each IOC.

### Tasks
- Add `iocStats` module to the IOC build system
- Include required database files (e.g., `iocAdmin.db`)
- Modify IOC startup script (`st.cmd`) to:
  - Load `iocStats`
  - Load the database
- Rebuild the IOC (if necessary)
- Restart the IOC

### Verification
Ensure the following Process Variables (PVs) are available:
- `$(IOC):HEARTBEAT`
- `$(IOC):LOAD`
- `$(IOC):MEM`

### Notes
- Naming conventions should be consistent across all IOCs
- Ensure PVs are accessible over the EPICS Channel Access (CA) network

---

## Part 2 — Linux SysAdmin Responsibilities (Infrastructure & Monitoring)

The Linux SysAdmin supports the environment where IOCs and monitoring systems run.

### Tasks
- Ensure network connectivity for EPICS:
  - Allow Channel Access ports (e.g., `5064`, `5065`)
- Configure firewall rules (`iptables`, `firewalld`, etc.)
- Deploy and maintain monitoring stack:
  - Prometheus
  - Grafana
  - Any required exporters
- Ensure system reliability:
  - Service management (systemd)
  - Logging and monitoring
- Maintain DNS/hostname consistency

### Optional (Depending on Team Structure)
- Assist with building the `iocStats` module
- Automate deployment across multiple IOCs (e.g., Ansible)
- Help standardize configurations

---

## Part 3 — Data Flow (IOC → Archiver → Grafana)

The `iocStats` module exposes IOC metrics as EPICS Process Variables (PVs).

### Data Pipeline
1. **IOC (iocStats)**
   - Exposes runtime metrics as PVs (e.g., heartbeat, CPU load, memory)

2. **EPICS Archiver Appliance**
   - Collects and stores PV time-series data
   - Must be configured to archive the relevant `iocStats` PVs

3. **Grafana (via archiverappliance-datasource plugin)**
   - Queries archived PV data from the Archiver Appliance
   - Visualizes historical trends and metrics

### Important Notes
- PVs must be explicitly added to the Archiver Appliance; they are **not archived automatically**
- There may be a delay between IOC startup and data availability in Grafana (due to archiving latency)
- Ensure consistent PV naming for easier dashboard creation

---

## End-to-End Flow Summary

```
IOC (iocStats PVs)
        │
        ▼
EPICS Archiver Appliance
        │
        ▼
Grafana (Archiver Appliance Data Source Plugin)

```


## Architecture Roles Summary

| Component                         | Responsibility                          | Owner            |
|----------------------------------|------------------------------------------|------------------|
| IOC + iocStats                   | Exposes runtime metrics as PVs           | EPICS Engineer   |
| EPICS Archiver Appliance         | Collects & stores PV time-series data    | EPICS Engineer   |
| Grafana                          | Visualization layer (all data sources)   | SysAdmin         |
| Prometheus                       | Collects infrastructure/system metrics   | SysAdmin         |
| Linux OS & Networking            | Infrastructure layer                     | SysAdmin         |

