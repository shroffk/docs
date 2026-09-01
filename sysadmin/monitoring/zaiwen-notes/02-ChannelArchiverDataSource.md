https://github.com/KeckObservatory/epics-grafana-datasource  

EPICS IOCs → Channel Archiver → Grafana datasource plugin → Grafana UI

---

In this model, EPICS IOCs generate PV data from hardware, which is collected and stored by the Channel Archiver. Grafana then queries this historical data through a dedicated datasource plugin to visualize trends and system behavior.

<img width="517" height="492" alt="image" src="https://github.com/user-attachments/assets/6d63f567-da3f-41a7-b3b7-a903af674251" />

---

Both plugins appear to target EPICS archiving systems rather than live EPICS IOCs. One integrates with the newer Archiver Appliance, while the other supports the older Channel Archiver. So the decision is less about live vs archived EPICS, and more about which EPICS archiving backend we standardize on.

So both plugins are actually in the SAME architectural family

Comparison:

✔️ Plugin 1 (archiverappliance-datasource)
EPICS Archiver Appliance (modern archiver)
Grafana reads archived PVs

✔️ Plugin 2 (epics-grafana-datasource)
EPICS Channel Archiver (older archiver system)
Grafana reads archived PVs
