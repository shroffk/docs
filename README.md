# Documentation for EIC

Documentation repo for the EIC controls. The repo consists information about the EIC controls infrastructure, tools, ans services.

* [Infrastructure](eic-deployment/infrastructure.md)
* [Ansible](eic-deployment/ansible/ansible.md)
<<<<<<< HEAD


<<<<<<< HEAD
=======
#### 1.2 Interface 2

* **IP Range:** 172.16.52.0/24
* **DHCP Services:** Provided by pi
* **Note:** This network doesn't have access to BNL resources; aiming to replace with a real network segment assigned to EIC by ITD.

### 2. Hosts

| Hostname                     | IP            | Main User/Owner  | OS Version | Use                                        |
| ---------------------------- | ------------- | ---------------- | ---------- | ------------------------------------------ |
| alarmdemo.eic.bnl.gov        | 130.199.97.28 | seth             | CentOS 9   | Alarm services                             |
| demo01.eic.bnl.gov           | 130.199.97.30 | Kunal            | CentOS 9   | Ansible controller, Phoebus Alarm Services |
| demo02.eic.bnl.gov           | 130.199.97.36 | Kunal            | CentOS 8   | EPICS base + basic support modules         |
| demo03.eic.bnl.gov           | 130.199.97.37 | free             | CentOS 8   | (Specify the use)                          |
| demo04.eic.bnl.gov           | 130.199.97.38 | free             | CentOS 8   | (Specify the use)                          |
| sequencerdev.eic.bnl.gov     | 130.199.97.38 | Sam              | CentOS 8   | Sequencer Dev and Demo (&check; Desktop/GUI )  |
| ologdev.eic.bnl.gov          | 130.199.97.38 | Kunal            | CentOS 8   | Olog Dev and Demo ( &check; Desktop/GUI )       |

### 3. User Credentials

* **User:** eicuser
* **Password:** eicuser!!!

### 4. VM Specifications

Each VM is allocated:

* **RAM:** 8GB
* **Cores:** 4
* **Storage:** 55GB

### 5. VM Remote Access
* All servers will have basic X11 Fowarding capabilities over SSH
* All servers with Desktop/GUIs will have xrdp package installed for remote access via RDP Protocol (Microsoft RDP, remminia, xfreerdp clients compatible)
  
Feel free to adjust and add more details based on your specific requirements. If you have additional information or specific areas you'd like to focus on, let me know!\
>>>>>>> branch 'master' of git@github.com:eicorg/docs.git
=======
>>>>>>> branch 'ansible' of https://github.com/shroffk/docs.git
