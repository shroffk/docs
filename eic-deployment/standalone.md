## Creating a StandAlone Machine (Controls)

02/02/2017 - This document describes the steps (and scripts created) to rollout a standalone Controls machine.

#### Methodology 
A standalone control system is a single Linux host which can provide a file system, run controls servers, and act as host for user interface programs.  
This standalone host will have a stripped down version of the Controls operations file server directory structure.
The basic idea is to provide the ability to boot FECs, run client programs like pet and Gpm, and log data, running its own local database server.
Thee same basic RedHat Linux OS installation is used. There are aspects of the basic Linux setup, however, that will probably need to be different for machines outside our networks (e.g. dns, authentication).
     
Template area is located in folder /home/cfsd/sysadmin/templates/Linux/StandAlone/ 

#### Steps with Explanation
1. Before pxeboot kickstart install of the fresh OS, please review database carefully for the machine information.
	- For example, clearcase will not be needed, so cc=n
	- Hard disk need to have at least 100GB

2. Database Server Installation and Configuration
	- Sybase 15.7
	- Sybase 16.0

3. Remove machine from puppet group
	- Run script - - /home/cfsd/sysadmin/templates/Linux/StandAlone/bin/cleanupPuppet.runasroot
	- This script will remove this machine from puppet master, clean up its certificate and report files so that it won't be affected by puppet.
	- Also puppet cron job will be removed.

4. Create local accounts (Assume Centrify coexists)
	- Run script - /home/cfsd/sysadmin/templates/Linux/StandAlone/bin/createLocalAccount.runasroot
		- This script will create any local accounts necessary.
		- The local accounts need to have "-c" for user comment name (database requires it).
		- cadalone - This local cadalone user need to have the same uid 310066 as its Centrify account. Will also copy the sample home area.
		- sybase - This local sybase user need to have the same uid 217 as its Centrify account, belong to the sybase group which has gid 217, and its shell should be /bin/tcsh. Ensure the /usr/share/sybase is pointing to this local sybase home folder.

5. Create local NoMachine server
	- Run script - /home/cfsd/sysadmin/templates/Linux/StandAlone/bin/setupNomachine.runasroot
	- Copy license - copy from /home/cfsd/sysadmin/templates/NX/licenses/standalone/ folder. Ensure correct owner (nx) and permission (400).

6. Create local python
	- NOTE: if you want to install a specific python version, append "conda install python=x.x.x" to the Anaconda script.
	For RH 7.6
	- wget http://repo.continuum.io/archive/Anaconda2-5.2.0-Linux-x86_64.sh
	- Run script - /home/cfsd/sysadmin/ThirdPartyApp/Anaconda-Python/Anaconda2-5.2.0-Linux-x86_64.sh
	- To install Python3: Run script - /home/cfsd/sysadmin/ThirdPartyApp/Anaconda-Python/Anaconda3-5.1.0-Linux-x86_64.sh
	- Do you approve the license terms? yes
	- specify a different location below - /opt/anaconda2      
	- Do you wish the installer to prepend the Anaconda2 install location? no
	- After python install, fix path
	- rm  /usr/local/anaconda2
	- ln -s /opt/anaconda2 /usr/local/anaconda2
	- ln -s /opt/anaconda3 /usr/local/anaconda3
	- rm /usr/bin/python2
	- ln -s /opt/anaconda2/bin/python2.7 /usr/bin/python2
	- ln -s /opt/anaconda3/bin/python3 /usr/bin/python3
	- rm /usr/bin/python2-config
	- ln -s /opt/anaconda2/bin/python2.7-config /usr/bin/python2-config
	- ln -s /opt/anaconda3/bin/python3-config /usr/bin/python3-config
	- update /usr/bin/yum -- changing the first line from  #!/usr/bin/python to #!/usr/bin/python2.7
	- Install more modules
	- /opt/anaconda2/bin/conda install -c ska sybase=0.39
	- /opt/anaconda2/bin/conda install -c conda-forge pyepics
	- /opt/anaconda2/bin/conda install pyqtgraph
	- /opt/anaconda2/bin/conda install mysql-connector-python
	- /opt/anaconda3/bin/conda install pyqtgraph
	- /opt/anaconda3/bin/conda install -c conda-forge pyepics
	- /opt/anaconda3/bin/conda install mysql-connector-python

7. Create local file structure - most basics
	- Run script - /home/cfsd/sysadmin/templates/Linux/StandAlone/bin/makeLocalDirs.runasroot
	- This script creates the /home/here/cadLocal directory structure which, on a standalone controls host, will take the place of directories on the central file system.
	- You can safely ignore those permission denied messages for many files in usrLocals/WS/share
	- Please refer to this document for details - Local File System Created
	- Run script - /home/cfsd/sysadmin/templates/Linux/StandAlone/bin/makeLocalLinks.runasroot
	- This script creates the appropriate links to the /home/here/cadLocal directory structure on a standalone controls host, replacing the standard link to the central C-AD file system
	- Please refer to this document for details - Links to Replace


8. Create local documentation for pet/Gpm/LogView/cameraViewer
	- Run script - /home/cfsd/sysadmin/templates/Linux/StandAlone/bin/setupLocalDoc.runasroot
	- Create an internal virtual network interface for www.cadops.bnl.gov

9. Basic Controls Infrastructure for any machine will be defined
	- By default, those infrastructure are already defined in above steps
		- binaries for StartUp
		- binaries for pet
		- binaries for PrintUtility
	- managers (ados)
		- by default, simpleMan is configured in above steps

10. Default Controls Server Management
	- Run script to start/stop/restart/check all basic servers - sudo /usr/controls/scripts/easyControl
	- Basic Servers include:
		- ControlsNameServer(cns)
		- simpleMan
		- notifServer
		- cdevNameServer (cdevrsvcServer)
		- Special Servers
	- You need to review the structure to add/delete what specific servers you would like to stop or check

11. Network configuration and installation (Installation/site specific - The SysAdmin and Developer need to understand the site capabilities in order to determine the proper setup)
	- DNS
	- NIS
	- NTP
	- GATEWAY
		- If clients have both public and private networks, don't configure GATEWAY for the internal private networks!
	- If camera is being used, ensure  the network interface supports the jumbo packets.
		- MTU=8192 in ifcfg-eth0
	- Centrify (how do we handle possibility of no network, to prevent hangs)
	- Cutoff connection to CAD file system
		- Run script - /home/cfsd/sysadmin/templates/Linux/StandAlone/bin/changeOsConfigs.runasroot
		- This script will change some system configurations such as remove auto mounts to the central file system, add "cadalone" account to sudoer, enable local account login, configure ntp, etc.
		- This script will also setup a cron job for /root/machineHealth.csh to report machine status to C-AD via email (postfix is also reconfigured to use smail.bnl.gov instead of mail.pbn.bnl.gov). Details here.
		- This script will also setup a timed archives. 

12. System validation at user site
	- verify local account login
	- verify network connection
	- start/stop/restart/check all servers
	- verify basic applications (pet, StartUP, Gpm, PrintUtility)
	- (optional) Test with client's cameras on site
	- Need to update database with the new camera id (e.g., from BASLER_21839306 to Basler3)
	- WARNING - The adoCache MUST be removed before attaching an existing ado to interface with a camera.
		- This will be true anytime a camera is moved to an ebic ado that already exists.
		- /operations/app_store/adoCache/cadatf01/ebic.atf1
	- verify remote display
		- When user has problem with the machine, you can have them send the display back to your desktop.
		- On your desktop:  ssh -L 38080:localhost:38080 yourBNLusername@ssh.pbn.bnl.gov  (or .local if you are behind CAD firewall)
		- On user machine:  ssh -R 38080:localhost:22 hisBNLusename@ssh.pbn.bnl.gov (or click on connectToGateway.csh on his desktop)
		- On your desktop:  ssh -p 38080 localusername@localhost
		- Now the ssh tunnel is built and you can do
			- scp -P 38080 /home/cfsd/atestfile localusername@localhost:/tmp
			- On your desktop: launch nxclient to connect to nxserver on user machine (protocal is ssh, connect to localhost, port is 38080, nomachine login, use the local username and password). If the license expired on user's machine, you can copy it from acnuser04:/usr/NX/etc/server.lic. No restart of nxserver needed.
			- If you would like to allow other machines to ssh to the standalone machine, you need to ensure /etc/hosts.allow is allowing it.



