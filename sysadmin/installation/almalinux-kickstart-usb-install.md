## EIC Linux System Installation
****

## **AlmaLinux 9.5 Kickstart Using USB Boot ISO Installation**

## Overview

This procedure creates a customized AlmaLinux 9.5 boot ISO that supports fully automated installation using a Kickstart file.   
The customized image is based on the standard AlmaLinux boot ISO (~1 GB) with modifications to include the Kickstart configuration and installation parameters.

The base AlmaLinux boot ISO is unpacked and modified to include configuration for:
- HTTP installation source
- NFS-hosted Kickstart file
- Network configuration passed from the boot command line
- Disk configuration that ignores the USB device during partitioning

A new boot ISO is then generated, converted to a hybrid image, and written to a USB device.

When booting from this USB device, the installation can run completely unattended.   
Only the hostname and network settings (IP, netmask, gateway) need to be specified on the boot command line.

## Custom Boot ISO and Kickstart

The following files are available under:

`/eic/opt/installation/`

- **Boot ISO**  
  `bootIso/EIC-AlmaLinux-9.5-x86_64-boot.iso`

- **Kickstart file**  
  `kickstart/ks-almalinux95-base-usb.cfg`


**Detailed Steps:**
-------------------

**I.  Customize AlmaLinux Boot ISO**

- Download the AlmaLinux boot ISO and checksum:
    - https://vault.almalinux.org/9.5/isos/x86_64/AlmaLinux-9.5-x86_64-boot.iso
    - https://vault.almalinux.org/9.5/isos/x86_64/CHECKSUM

- Save ISO to NFS folder
    - `/eic/opt/packages/AlmaLinux/AlmaLinux-9.5-x86_64-boot.iso` 

- Mount the boot iso at some location
  ```bash
  # mkdir /mnt/alma95iso
  # mount -o loop /eic/opt/packages/AlmaLinux/AlmaLinux-9.5-x86_64-boot.iso /mnt/alma95iso
  ```

- Copy over the boot iso image to local
  ```bash
  # mkdir /tmp/alma95boot
  # shopt -s dotglob
  # cp -avRf /mnt/alma95iso/* /tmp/alma95boot/
  ```
- First, let's add a stanza for BIOS machine. Add the stanza in /tmp/alma95boot/isolinux/isolinux.cfg file as follows. Just add it above the Linux stanza (then you can remove the Linux stanza) :
  ```bash
  ######################################################################################################################              
  label EIC Kickstart - AlmaLinux 9.5  
    menu label ^Install AlmaLinux 9.5 using USB Kickstart file
    menu default
    kernel vmlinuz
    append initrd=initrd.img net.ifnames.prefix=enp inst.stage2=https://vault.almalinux.org/9.5/BaseOS/x86_64/os/ inst.ks=nfs:devnas01.eic.bnl.gov:/mnt/nfsstore/pool1/opt/installation/kickstart/ks-almalinux95-base-usb.cfg ip=130.199.XXX.XXX::130.199.96.24:255.255.254.0:eiclinXX.eic.bnl.gov:enp0:none nameserver=130.199.1.1               
  #####################################################################################################################
  ```

- Remove "menu default" from the below "Test this ^media & install AlmaLinux 9.5 " stanza. Otherwise this stanza will be the default instead  of your stanza above.
  ```bash
  ######################################################################################################################
  label check
    menu label Test this ^media & install AlmaLinux 9.5
    menu default  --> removed!
    kernel vmlinuz
    append initrd=initrd.img inst.stage2=hd:LABEL=AlmaLinux-9-5-x86_64-dvd rd.live.check quiet
  ######################################################################################################################
  ```
     
- (Optional) Modify the layout
  - I noticed that the boot console command line might not display correctly if it needs to wrap the line.
  - So the layout is updated to provide more space for the command line:
    ```bash
    display boot.msg
    menu vshift 8 -> 4
    menu rows 18 ->10
    menu margin 8 ->10
    menu helpmsgrow 15 -> 12
    menu tabmsgrow 13 -> 11
    ```                
  
- Then, let's add a stanza for UEFI machine. In /tmp/alma95boot/EFI/BOOT/grub.cfg file,  modify the first "menuentry" as below:
    ```bash
    ######################################################################################################################      
    menuentry 'Install AlmaLinux 9.5 USB Kickstart' --class fedora --class gnu-linux --class gnu --class os {
    linuxefi /images/pxeboot/vmlinuz net.ifnames.prefix=enp inst.stage2=https://vault.almalinux.org/9.5/BaseOS/x86_64/os/ inst.ks=nfs:devnas01.eic.bnl.gov:/mnt/nfsstore/pool1/opt/installation/kickstart/ks-almalinux95-base-usb.cfg ip=130.199.XXX.XXX::130.199.96.24:255.255.254.0:eiclinXX.eic.bnl.gov:enp0:none nameserver=130.199.1.1
    initrdefi /images/pxeboot/initrd.img
    }
    ######################################################################################################################
    ```

- Create a new iso and isohybrid it (otherwise it won't be bootable for USB device)
  ```bash
  # cd /tmp/alma95boot
  # mkisofs -U -r -v -T -J -joliet-long -V "AlmaLinux-9.5 Workstation.x86_64" -volset "AlmaLinux-9.5 Workstation.x86_64" -A "AlmaLinux-9.5 Workstation.x86_64" -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -eltorito-alt-boot -e images/efiboot.img -no-emul-boot -o ../EIC-AlmaLinux-9.5-x86_64-boot.iso .
  # isohybrid /tmp/EIC-AlmaLinux-9.5-x86_64-boot.iso
  ```

- Confirm the LABEL of the boot iso
  ```bash
  # blkid /tmp/EIC-AlmaLinux-9.5-x86_64-boot.iso
  ```

- It is also very useful to implant the md5 checksum and verify the implanted MD5sum:
  ```bash
  # implantisomd5 /tmp/EIC-AlmaLinux-9.5-x86_64-boot.iso
  # checkisomd5 --verbose /tmp/EIC-AlmaLinux-9.5-x86_64-boot.iso
  ```
               
- Saved this new iso in template area
  - `/eic/opt/installation/bootIso/EIC-AlmaLinux-9.5-x86_64-boot.iso`


**II.  Create AlmaLinux Kickstart File to Support USB Boot**

 - https://access.redhat.com/labs/kickstartconfig/
   
 - Add ignoredisk for usb
   - https://access.redhat.com/solutions/66801
   - Since we have "clearpart --all" and "zerombr" in kickstart, it will wipe the USB device partition too. As a result, your USB device will be bootable only once and you have to run the dd command again.
   - The solution is to add this line in kickstart:
     ```bash
     ignoredisk --drives=/dev/disk/by-id/usb*
     ```
   - But the above command will fail and halt the install if there are no USB disks attached to the system. To accommodate that, the one line can be expanded to a few more, including a prescript.
   - The %include will fail if /tmp/ignoredisk doesn't exist. That's why this file needs to be touched for the no usb case.
     ```bash
     %include /tmp/ignoredisk
      %pre  
          # Detect USB disk(s) for ignoredisk
          if ls /dev/disk/by-id/usb* &>/dev/null; then
            echo "ignoredisk --drives=/dev/disk/by-id/usb*" >/tmp/ignoredisk
          else
            touch /tmp/ignoredisk
          fi
        end
     ```

- Determine Installation Disk
  - We want to make the kickstart file robust for both sdX and NVMe disks.
  - The %pre script previously relied on list-harddrives to determine the installation disk. In USB boot environments, this command can occasionally return no output (due to timing or device enumeration), resulting in an empty variable. This caused the ignoredisk --only-use= directive to be invalid, triggering a kickstart parsing error.
  - To make disk selection deterministic and hardware-agnostic, we instead select the largest non-USB disk using lsblk. This avoids dependency on device ordering and ensures the installer does not mistakenly select the USB boot device.
    ```bash
      # --- Find largest non-USB disk ---
      d1=$(lsblk -b -dn -o NAME,SIZE,TYPE,TRAN | \
           awk '$3=="disk" && $4!="usb" {print $1, $2}' | \
           sort -k2 -nr | \
           head -1 | \
           awk '{print $1}')
      
      # --- Safety check ---
      if [[ -z "$d1" ]]; then
          echo "ERROR: No suitable install disk found!" >&2
          exit 1
      fi
      
      # --- Generate disk config ---
      cat > /tmp/diskconfig <<EOF
      zerombr
      clearpart --all --drives=$d1 --initlabel
      autopart --type=lvm
      EOF
     ```




- Pass arguments from the boot command line to a kickstart %pre script
  - During boot, we edit the boot arguments with machine hostname/ip/netmask/gateway. This network configuration are saved in /proc/cmdline and need to be passed to the kickstart file.
  - So in the kickstart file, in the %pre section, create /tmp/networkconfig by reading network configuration from /proc/cmdline if the /proc/cmdline contains the word "ip" (that's one of the keywords we place on the command line).
  - In the command section, use %include /tmp/networkconfig to get the settings you need.
  - https://www.redhat.com/archives/kickstart-list/2012-April/msg00017.html
  - https://www.redhat.com/archives/kickstart-list/2007-July/msg00034.html
  - https://www.centos.org/forums/viewtopic.php?t=49935
  - https://serverfault.com/questions/372609/how-do-i-pass-arguments-from-the-pxe-command-line-to-a-kickstart-pre-post-scr

  - Sample Boot console format in AlmaLinux 9.5
    ```bash
    ip=130.199.104.36::130.199.104.24:255.255.254.0:acnlin52.pbn.bnl.gov:enp4s0:none nameserver=130.199.1.1
    ```
    
  - Sample Kickstart for AlmaLinux 9.5
    ```bash
    network --device=$interface --hostname=$hostname --bootproto=static --ip=$ip --netmask=$netmask --gateway=$gateway --nameserver=130.199.1.1
    ```
 
   - Final Kickstart File to parse kernel boot parameters for ip= line
     ```bash
      %include /tmp/networkconfig
      %pre                    
      # Parse kernel boot parameters for ip= line
        ip_param=""
        for param in $(cat /proc/cmdline); do
            if [[ $param == ip=* ]]; then
                ip_param="${param#ip=}"
                break
            fi
        done
        
        if [[ -n "$ip_param" ]]; then
            # Split by colon
            IFS=':' read -r newip _ newgateway newnetmask newhostname newinterface _ <<< "$ip_param"
            echo "network --device=$newinterface --hostname=$newhostname --bootproto=static --ip=$newip --netmask=$newnetmask --gateway=$newgateway --nameserver=130.199.1.1" > /tmp/networkconfig
        fi
        end
      ```



**III. Use this USB device to install AlmaLinux 9.5**

- For physical machine installation, copy the customized AlmaLinux 9.5 USB Boot ISO to USB device
  ```bash
  # Use dd command
  dd if=/eic/opt/installation/bootIso/EIC-AlmaLinux-9.5-x86_64-boot.iso of=/dev/sdX bs=4M status=progress oflag=sync

  # Use script  
  /eic/opt/installation/bootIso/copy-bootiso-to-usb.sh 

  # NOTE: if you get error "find: ‘/usr/lib/syslinux/’: No such file or directory", follow https://github.com/jsamr/bootiso/issues/29
  mkdir -p /usr/lib/syslinux/bios
  rsync -av /usr/share/syslinux/ /usr/lib/syslinux/bios/
  ```

- For VM installation, upload the customized AlmaLinux 9.5 USB Boot ISO to your VM storage domain (e.g., Proxmox Storage)
  - Login to the Proxmox Web UI: https://prox01.eic.bnl.gov:8006
  - Navigate to: Datacenter → Folder View → Storage → cephfs001
  - Click ISO Images → Upload → Select File → Upload
  - The ISO will be stored at: /mnt/pve/cephfs001/template/iso/EIC-AlmaLinux-9.5-x86_64-boot.iso  

- Go to the machine’s local console, insert the USB device into an available USB port, and power on the system.

- Depending on it is physical machine or virtual machine
  - If it is physical machine
    - Power on and press F11 to get the boot options. Select "USB Boot" or something similar (On HP Mimi machines, choose Legacy Boot)
    - If you don't see "USB boot", you need to enable the USB Boot option in BIOS Setup (F2 or F10).
  - If it is virtual machine
    - When creating the VM, add EIC-AlmaLinux-9.5-x86_64-boot.iso as boot media.
      <img width="724" height="177" alt="image" src="https://github.com/user-attachments/assets/19ba9c79-2487-4685-b7ec-92a75361f9d4" />
    - After VM is created, go to Options -> Boot Order. 
      <img width="657" height="169" alt="image" src="https://github.com/user-attachments/assets/e870f699-e5aa-4de7-8fdf-1a05b2e2c079" />                      

- Press Tab key to edit network configuration  
  <img width="530" height="400" alt="image" src="https://github.com/user-attachments/assets/7d3c49b0-b659-4a4a-9beb-e9e4256475eb" />  

- Replace XX with your actual hostname and IP address.  You can move the cursor backwards (The screenshot is for BIOS machine. For UEFI it looks different, but the idea is the same):  
  <img width="536" height="404" alt="image" src="https://github.com/user-attachments/assets/aca47733-dd21-44aa-8e34-cf9182b2b970" />
        
- Press Enter. The installation will start automatically.  

- Remove the USB stick after you see the installation console being launched (Be graceful......)  
  <img width="655" height="412" alt="image" src="https://github.com/user-attachments/assets/c5b6153b-26fc-47a0-9994-d9108eea3081" />
    
----

**Appendix - Debug Process**

Enable debug during boot: rd.debug initcall_debug log_buf_len=10M

**1. Issue: how could I know which interface to specify for  ip::gateway:netmask:hostname:interface:none?**

- Problem
  - https://access.redhat.com/support/cases/#/case/03339575
  - There are significant syntax changes with RHEL 8 kickstarts. You need to specify the interface name now.
  - In old RH 7.6, I don't need to specify any interface. I only need to use option "ksdevice=link" and the boot iso will find the interface automatically.
  - Namely when booting from an ISO file and starting a kickstart, this syntax for "ip=" is used for static: ip::gateway:netmask:hostname:interface:none.
  - The problem for me is that I don't know which interface to specify.How about if I have a completely new physical machine and I don't know its interface name?
- Solution
  - https://access.redhat.com/articles/4153931
  - Use kernel cmd param net.ifnames.prefix= to change the NIC names from systemd naming scheme to custom names like enp[0,1,2,....]
  - append initrd=initrd.img net.ifnames.prefix=enp ip=130.199.XXX.XXX::130.199.XXX.24:255.255.XXX.0:acnlinXX.pbn.bnl.gov:enp0:none nameserver=130.199.1.1 


**2. Issue: After I created my own boot iso, the stanza I updated doesn't show up?**

- Problem
  - It turns out that that stanza is not the default
- Solution
  - Need to add the line "**menu default**" to this stanza and removed from the "Test and Install" stanza.           

 

**3.  The network configuration format changed and need to update ks pre script to capture the new format**

- Problem
  - There are significant syntax changes with RHEL 8 kickstarts. Namely when booting from an ISO file and starting a kickstart, this syntax for "ip=" is used for static: ip::gateway:netmask:hostname:interface:none
  - Old boot console format in RH 7.6 - #nameserver=130.199.1.1 netmask=255.255.254.0 gateway=130.199.104.24 ip=130.199.104.36 hostname=acnlin52.pbn.bnl.gov
  - New boot console format in RH 96 - #ip=130.199.104.36::130.199.104.24:255.255.254.0:acnlin52.pbn.bnl.gov:enp1s0:none nameserver=130.199.1.1
  - Old kickstart for RH 7.6 - #network --device=link --bootproto=static --ipv6=auto --activate --nameserver=130.199.1.1 --hostname=$hostname --ip=$ip --netmask=$netmask --gateway=$gateway
  - New kickstart for RH 96 - #network --device=$interface --hostname=$hostname --bootproto=static --ip=$ip --netmask=$netmask --gateway=$gateway --nameserver=130.199.1.1  

- Solution
  - Update ks pre script to capture new format of network information
    ```bash
     # Parse kernel boot parameters for ip= line
        ip_param=""
        for param in $(cat /proc/cmdline); do
            if [[ $param == ip=* ]]; then
                ip_param="${param#ip=}"
                break
            fi
        done
        echo "network --device=$newinterface --hostname=$newhostname --bootproto=static --ip=$newip --netmask=$newnetmask --gateway=$newgateway --nameserver=130.199.1.1" >  /tmp/networkconfig
     ``` 

**Appendix - Document Reference**

- RH 96 Installing RHEL using Kickstart - https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/performing_an_advanced_rhel_9_installation/index
- RHEL 96 Network boot options - https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/boot_options_for_rhel_installer/index#network-boot-options_kickstart-and-advanced-boot-options
- Kickstart Generator - https://access.redhat.com/labs/kickstartconfig/
- What to do if a server fails to boot? - https://access.redhat.com/articles/5250481
- How to create a modified Red Hat Enterprise Linux ISO with kickstart file or modified installation media? -https://access.redhat.com/solutions/60959
- https://www.cadops.bnl.gov/Controls/doc/SA/RH96/RH96-Documents/doc-installation/Install-RHEL9-usbboot.html




