https://proceedings.jacow.org/icalepcs2023/papers/th2bco03.pdf  

THE LCLS-II EXPERIMENT CONTROL SYSTEM  

---

This 2023 paper shows a shift toward a unified observability architecture where EPICS data, logs, and infrastructure metrics are all integrated into Grafana-based dashboards, reducing the need for custom EPICS-specific middleware and moving toward standard observability pipelines.

The paper uses Logstash and Elasticsearch to normalize EPICS-related logs into structured observability data. So the ‘normalization’ here is at the log/event level rather than transforming the EPICS PV data model itself. Grafana then consumes this as a standard observability backend.   

Compared to the earlier work, the 2023 paper is more direct in the sense that it relies on standard observability pipelines and Grafana as a central integration layer, rather than custom EPICS-specific middleware.   

EPICS is integrated into a general observability pipeline, and Grafana just consumes it.   

EPICS data is handled like this:
- EPICS logs (IOC logs, gateway logs, error logs, etc.)
- are collected from multiple sources
- sent into a central log pipeline (Logstash-based ingestion)
- then stored in a structured backend (typically Elasticsearch-style)
- then visualized in Grafana dashboards 

logs from EPICS IOC systems and control infrastructure are aggregated, processed, and stored via Logstash pipelines for centralized visualization in Grafana

So where is the “integration layer”?  

It still exists—but it’s just not EPICS-specific inside Grafana.  

Instead of:  
EPICS → custom Grafana EPICS plugin → Grafana   

It becomes:  
EPICS → log/metrics pipeline → standard datastore → Grafana datasource   

      
<img width="451" height="380" alt="image" src="https://github.com/user-attachments/assets/a064e1fe-abf8-4e04-9385-5373cf966131" />


