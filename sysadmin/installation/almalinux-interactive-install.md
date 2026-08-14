## EIC Linux System Installation
****

## **AlmaLinux 9.5 Interactive Installation**

## Overall Steps:

1. Register MAC Address
2. Prepare Boot USB
3. Boot and Install AlmaLinux
4. Post-install Access Test
5. Post Installation via Ansible  


**Detailed Steps:**
-------------------

**1. Register MAC Address**  

- [ITD Network Registration Page](https://netapps.bnl.gov/networkregistration/admin/reports/)
    - eiclin04.eic.bnl.gov, 130.199.97.160, c8:5a:cf:0c:d7:e9
    - eiclin05.eic.bnl.gov, 130.199.97.161, c8:5a:cf:0c:7a:4f
    - eiclin06.eic.bnl.gov, 130.199.97.162, 40:a8:f0:68:4e:67

**2. Prepare Boot USB**

- Download the AlmaLinux boot ISO and checksum:
    - https://vault.almalinux.org/9.5/isos/x86_64/AlmaLinux-9.5-x86_64-boot.iso
    - https://vault.almalinux.org/9.5/isos/x86_64/CHECKSUM
- Save ISO to NFS folder
    - `/eic/opt/packages/AlmaLinux/AlmaLinux-9.5-x86_64-boot.iso`
- Create bootable USB
    ```bash
    # sudo dd if=/eic/opt/packages/AlmaLinux/AlmaLinux-9.5-x86_64-boot.iso of=/dev/sdb bs=4M conv=fdatasync status=progress
    # sync
    ```

**3. Boot and Install AlmaLinux**

- Insert USB, power on, press F10 → Boot Options.
- Ensure USB is first; optional to uncheck CDROM/Network PXEboot
- Select Install AlmaLinux 9.5
- Installation Summary
    ```bash 
        Root password: set & enable “Allow root SSH login”
        Local user: eicuser (administrator)
        Installation source: Closest mirror
        Installation Destination: Automatic partitioning
        Software selection:
            Workstation + Internet Applications, Office Suite, Dev Tools, Graphical Admin Tools, System Tools
        Network
            Hostname: eiclin04/05/06.eic.bnl.gov
            Configure Ethernet → IPv4 Settings:
                Method: Manual
                IP: 130.199.97.160/161/162
                Netmask: 255.255.254.0
                Gateway: 130.199.96.24
                DNS Server: 130.199.1.1
                Search Domains: eic.bnl.gov,bnl.gov
    ```
       
- Begin installation (~5 minutes)
- Remove USB before reboot
- Reboot system → should boot from NVMe automatically

**4. Post-install Access Test**

```bash
# ssh root@eiclin04.eic.bnl.gov
# ssh root@eiclin05.eic.bnl.gov
# ssh root@eiclin06.eic.bnl.gov
```


**5. Post Installation via Ansible**

- Log onto EIC Ansible Server
    - https://ansible.eic.bnl.gov
- Add machine to “EICWorkstations” inventory if missing
- Run jobs:
    - base_os → sets up base OS configuration
    - user customization → sets up users, settings


**Doc Reference:**
-------------------

- https://github.com/eicorg/proxmox_eval/blob/main/phase5a-test-validation-vm-creation.md
- https://github.com/eicorg/proxmox_eval/blob/main/z-appendix-api-ansible.md

----
https://github.com/eicorg/proxmox_eval/blob/main/phase5a-test-validation-vm-creation.md
https://github.com/eicorg/proxmox_eval/blob/main/z-appendix-api-ansible.md
