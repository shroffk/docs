license01license01.eic.bnl.gov Installation Notes 

FlexLM license installation 

K. Mernick 

February 20, 2024 

    Download “License Management Tools” from https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/2022-2.html. This is at the bottom of the page. It says there is a tar/gzip file for linux, but it is actually a zip file. 

    The ld-lsb library is not part of CentOS 9 installation. This library is dynamically linked with the FlexLM tools. Follow instructions at https://community.flexera.com/t5/FlexNet-Publisher-Knowledge-Base/Workaround-for-LSB-Component-in-RHEL/ta-p/276473 to create symlink to work around this1. 

    Create directory for installation: 

    sudo mkdir /opt/Xilinx 

    sudo chgrp -R developer.con /opt/Xilinx/ 

    sudo chmod g+ws /opt/Xilinx/ 

    mkdir -p /opt/Xilinx/Vivado_lic/bin 

    Unzip the downloaded license tools: 

    cd /opt/Xilinx/Vivado_lic/bin 

    unzip ~/linux_flexlm_v11.17.2.0.zip 

    Fix permissions on unzipped files: 

    mv lin_flexlm_v11.17.2.0/lnx64.o/* . 

    rm -r lin_flexlm_v11.17.2.0 

    chmod +x lmgrd lmutil xilinxd 

    Get the hostid by running lmutil lmhostid 

    Generate license file on Xilinx site 

    Fill in required information on the host, you should see a summary screen like the one below. 

    License file will be emailed. Copy to /opt/Xilinx/Vivado_lic/Xilinx.lic. 

Graphical user interface, text, application

Description automatically generated 

    Create directory for log files: 

    mkdir -p /opt/Xilinx/Vivado_lic/logs 

    chmod g+w /opt/Xilinx/Vivado_lic/logs/ 

    Run the license server manually to check that it works: 

    /opt/Xilinx/Vivado_lic/bin/lmgrd -c /opt/Xilinx/Vivado_lic/Xilinx.lic -l /opt/Xilinx/Vivado_lic/logs/log_`date "+%F-%H%M%S"`.txt 

    Stop the license server: 

    /opt/Xilinx/Vivado_lic/bin/lmutil lmdown -c 2100@localhost 

 

Installation Notes  

    Kyle had to change default firewall settings on CentOS to allow connections. 

 

Systemd Configuration 

These steps set up systemd so that the license manager will be started automatically. A dedicated user is created so that the server runs as that user. Information for this comes from https://gist.github.com/kalebo/fd39edb6c6e4ebed41f7eab2d9925ebc and https://github.com/prehensilecode/flexlm-systemd-service. 

    Create flexlm user and group 

    sudo groupadd flexlm --system 

    sudo useradd flexlm -c "FlexLM User" -s /sbin/nologin -g flexlm –system 

    sudo chown -R flexlm /opt/Xilinx/Vivado_lic/ 

    Create the lmgrd.service file in /etc/systemd/system/ 

    Copy the following contents to the file2: 

[Unit] 

Description=FlexLM license server daemon 

After=network-online.target 

 

[Service] 

Type=simple 

User=flexlm 

WorkingDirectory=/opt/Xilinx/Vivado_lic/ 

ExecStart=/opt/Xilinx/Vivado_lic/bin/lmgrd -z -local -c /opt/Xilinx/Vivado_lic/Xilinx.lic -l +/opt/Xilinx/Vivado_lic/logs/log.txt 

SuccessExitStatus=15 

Restart=always 

RestartSec=30 

 

[Install] 

WantedBy=multi-user.target 

 

    Enable and start the service 

    sudo systemctl enable lmgrd.service 

    sudo systemctl start lmgrd.service 

 

Usage Notes 

    The lmutil utility can be used for a bunch of diagnostics and other interactions with the license server. It is installed with the Xilinx tools (in a subdirectory that is not part of the path, but you can run it from there) or with the license manager. Note that the ld-lsb library workaround in step 2 of the installation instructions is required to get lmutil to run. 

    Stopping or restarting the license server can be done with lmutil, but it must be run on the same machine as the license server (assuming it is started with the -local option as in the systemd configuration above). 

    lmutil lmdown -c 2100@localhost will stop the license server3 

    lmutil lmreread -c 2100@localhost will update the available licenses after rereading the license file 

    lmutil lmstat will provide information about the server status. The -a option will list all license features available (including number of seats available and checked out). 

November 2024 License Update 

I generated and installed a new license file on 11/8 with 10 seats of Vivado/Vitis. The old license file was copied to Xilinx.lic.old. The new license should be valid for versions released up to September 2025. 

 
