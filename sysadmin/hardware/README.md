## Hardware Overview

This is intended as a high-level document to cover physical machines in the EIC server room (rm. 218), their use case, and other details that are helpful to have for system administration.

# Development NAS

- OS: TrueNAS version TrueNAS-13.0-U6.2
- IP: 130.199.97.90
- Hostname: devnas01.eic.bnl.gov
- Hardware: HP EliteDesk 800 G4 SFF
- BNL Barcode: 165431
- Storage: 2x Samsung SSD 990 PRO 2TB in raid1
- SSH: No. Use web interface.

Notes
This is a SFF HP desktop that is currently (as of 8/4/26) sitting in the bottom of the rack in the server room. Effective storage space is 1.82 TiB.

Backup?
There is currently no backup...

# VMware

- OS: VMware ESXi Host Client, version 2.14.0
- IP: 130.199.97.3
- Hostname: modernbox.modernbox.c-ad.bnl.gov
- Hardware: S2600WFT
- BNL Barcode: N/A, has a red asset tag: TBD
- Storage: datastore01, total capacity 21.7TB
- SSH: No. Use web interface.

Notes
This is the Supermicro server on the cart behind the server rack. It currently hosts all of the VMs for EIC except for the fpga VMs. A Western Digital external drive is plugged into this machine, presumably for backups. I cannot find the backup mechanism, and as of last communication with Kyle he is not sure this is still working.

The underlying storage setup backing datastore01 is unknown. There isn't any information that I can find on how this is configured via the VMware GUI. It appears to be on the Avago / Megaraid controller, but disk size and raid type are not known.

# FPGA Development

- OS: Proxmox version Virtual Environment 7.4-3
- IP: 130.199.96.153
- Hostname: containers01.c-ad.bnl.gov
- Hardware: Supermicro HS219-R16H13
- BNL Barcode: 182517
- Storage: Unknown, total capacity 13.06TiB
- SSH: Yes. But use web interface.
- Web Interface: https://130.199.96.153:8006/

Notes
Machine is at the top of the first server rack in the server room. This is a stand alone Proxmox node. It is not part of a cluster. The storage backing the VMs isn't well understood. It's possible to SSH into the machine as root, and you can see 8x 1.7T nvme drives. This totals 13.6T of space, so mostly aligns with the total storage capacity as reported by the Proxmox web interface. 

Currently running 3 VMs:
- fpgadev.eic
- fpgadev02
- kevindev01

# Future Proxmox Test Cluster

- Hardware: 5x Dell R70 Servers

These machines are currently located in the first rack in the server room. 
