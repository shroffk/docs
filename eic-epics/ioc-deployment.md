## EPICS IOC Deployment Plan

- EPICS IOC apps, iocBoot and OPIs are hosted on GitHub. They are released and deployed through GitHub CI/CD or web interface only, so that we have all releases synchronized with GitHub. Until GitHub CI/CD or web interface is set up, we need to do it manually.

- On EIC VMs, the central NFS directory for deployment is under `/eic/release/epics`. Under this NFS mounted central directory, we have

```
.
└── epics
    ├── ioc
    ├── iocBoot
    └── opi
```

- Inside, the `iocBoot` and `opi` directories, new directories can be created that reflect the tree structure of the users or group.

- The IOCs are be started/stopped/monitored using the `manage-iocs` utility or an user interface (on top of manage-iocs).
