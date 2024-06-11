# Infrastructure

### 1. Network Configuration

#### 1.1 Interface 1

* **IP Range:** 130.199.96.0/23
* **DHCP Services:** Provided by ITD

#### 1.2 Interface 2

* **IP Range:** 172.16.52.0/24
* **DHCP Services:** Provided by pi
* **Note:** This network doesn't have access to BNL resources; aiming to replace with a real network segment assigned to EIC by ITD.

### 2. Hosts

| Hostname              | IP            | Main User/Owner | OS Version | Use                                        |
| --------------------- | ------------- | --------------- | ---------- | ------------------------------------------ |
| alarmdemo.eic.bnl.gov | 130.199.97.28 | seth            | CentOS 9   | Alarm services                             |
| demo01.eic.bnl.gov    | 130.199.97.30 | Kunal           | CentOS 9   | Ansible controller, Phoebus Alarm Services |
| demo02.eic.bnl.gov    | 130.199.97.36 | Kunal           | CentOS 8   | EPICS base + basic support modules         |
| demo03.eic.bnl.gov    | 130.199.97.37 | free            | CentOS 8   | (Specify the use)                          |
| demo04.eic.bnl.gov    | 130.199.97.38 | free            | CentOS 8   | (Specify the use)                          |
| demo05.eic.bnl.gov    | 130.199.97.71 | Jen             | CentOS 8   | MLflow testing / Pheobus testing           |

### 3. User Credentials

* **User:** eicuser
* **Password:** XXXXXX

### 4. VM Specifications

Each VM is allocated:

* **RAM:** 8GB
* **Cores:** 4
* **Storage:** 55GB

Feel free to adjust and add more details based on your specific requirements. If you have additional information or specific areas you'd like to focus on, let me know!\
