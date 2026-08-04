# REAR - Machine Backup and Restore

## Background

Susheng Xin requested:

> If you could clone the current H-coating workstation (eiclin07), that would be perfect for our local development, then all the IOCs could be copied over to the remote server and run from there.

## Solution

Use [Relax-and-Recover (ReaR)](http://relax-and-recover.org/), a disaster recovery and system migration utility, to create:

* A bootable rescue ISO
* A compressed backup archive

The target machine can then:

1. Boot from the rescue ISO
2. Recreate the original storage layout
3. Restore system and user files


## Detail Steps

In this example, I want to replicate **AlmaLinux 9.5** workstation **eiclin07** as **eiclin08**. Both are physical machines.

---

## I. Create Boot ISO on the Original Machine (e.g., eiclin07)

### Check Whether the System Uses BIOS or UEFI

```bash
test -d /sys/firmware/efi && echo "UEFI" || echo "BIOS"
lsblk -f
```

### Install ReaR

For a BIOS system:

```bash
yum install rear
```

For a UEFI system:

```bash
yum install rear grub2-efi-x64-modules
```

### Configuration 1 - Required 

**For BIOS systems**

Before running `rear -v mkrescue`, ensure that:

```text
/etc/rear/local.conf
```

is the default empty file.

**For UEFI systems**

Before running `rear -v mkrescue`, ensure that `/etc/rear/local.conf` contains:

```bash
SECURE_BOOT_BOOTLOADER="/boot/efi/EFI/redhat/shimx64.efi"
```

### Configuration 2 - Optional: Configure Networking in the Rescue Environment

You may configure networking through `local.conf`, or manually configure it later from the recovery console.

References:

* https://access.redhat.com/solutions/6812411
* https://github.com/rear/rear/issues/2272

To use the same IP address as the original machine:

```bash
USE_STATIC_NETWORKING=y
```

To assign a different IP address and hostname to the target machine:

> You need to know the target machine's network interface name. If it differs from the original machine, you can adjust it later from the recovery console.

```bash
USE_STATIC_NETWORKING=y

NETWORKING_PREPARATION_COMMANDS=(
  'ip addr add 130.199.97.106/23 dev eth0'
  'ip link set dev eth0 up'
  'ip route add default via 130.199.96.24'
  'hostname eiclin08.eic.bnl.gov'
)
```

### Configuration 3 - Optional: Use a Custom `disklayout.conf`

A custom `disklayout.conf` can be used to allocate a larger root (`/`) partition (for example, 25 GB).

> If the original root partition is too small, you may encounter a **"No space left"** error during recovery because `/mnt/local` simulates the restored root filesystem.

Relax-and-Recover allows you to define the target disk layout before the backup is created.

Copy the generated layout file:

```bash
cp /var/lib/rear/layout/disklayout.conf \
   /etc/rear/disklayout.conf
```

Then edit `/etc/rear/disklayout.conf` as needed.

Benefits:

* The file will not be overwritten by future backup runs.
* ReaR will use this layout during recovery instead of the original system layout.
* Editing the layout before creating the backup is much easier than modifying it during recovery.

Reference:

* https://github.com/rear/rear/blob/master/doc/user-guide/06-layout-configuration.adoc

See **Appendix I - disklayout.conf** for details.


### Create the Rescue ISO

```bash
rear -v mkrescue
```

### Result

This command creates:

```text
/var/lib/rear/output/rear-eiclin07.iso
```

Typical size:

```text
~900 MB
```

---

## II. Create Backup Archive on the Original Machine (e.g., eiclin07)

### Configure Backup Settings

Edit:

```bash
gedit /etc/rear/local.conf
```

Append:

```bash
BACKUP=NETFS
BACKUP_URL=file:///home/here/backup/
BACKUP_PROG_INCLUDE=( '/home/*' )
```

### Create the Backup Archive

```bash
rear -v mkbackuponly
```

#### Result

This command creates:

```text
/home/here/backup/eiclin07/backup.tar.gz
```

Typical size:

```text
~10 GB
```

### Optional: Split a Large Backup Archive

If `backup.tar.gz` is too large to transfer or upload, it can be split into smaller files and later reassembled.

Reference:

* https://www.tecmint.com/split-large-tar-into-multiple-files-of-certain-size/

---

## III. Copy the ISO and Backup Archive to a Central Location

### Store both the rescue ISO and backup archive in a shared location.  

```text id="8nqoq8"
/eic/opt/installation/ReaR
```

```text id="w6kttd"
-rwxrwxrwx. 1 root root  952M Jun 17 12:10 rear-eiclin07.iso
-rw-r--r--. 1 root root   11G Jun 17 10:01 backup-eiclin07.tar.gz
```

### Make the ISO Bootable for USB Media  
> Before writing the ISO to a USB device, run `isohybrid`.
>
> Without this step, the ISO may not be bootable from a USB device.

#### BIOS Systems

```bash id="saxzk9"
isohybrid rear-eiclin07.iso
```

#### UEFI Systems

```bash id="o0h40r"
isohybrid -u rear-eiclin07.iso
```

---

## IV. Create Boot Media - Write the ISO to a USB Device

Identify the correct USB device name (for example, `/dev/sdb`) and write the ISO image to it.

> **WARNING**
>
> This command will overwrite the target device completely.
> Verify the device name carefully before proceeding.

```bash id="xk2q7u"
dd if=rear-eiclin07.iso \
   of=/dev/sdX \
   bs=4M \
   status=progress \
   oflag=sync
```

Replace:

```text id="uy0x1z"
/dev/sdX
```

with the actual USB device.

Example:

```bash id="4y9iqf"
dd if=rear-eiclin07.iso \
   of=/dev/sdb \
   bs=4M \
   status=progress \
   oflag=sync
```

Once the command completes, the USB device can be used to boot the target machine into the ReaR recovery environment.

---

## V. Replicate a New Machine

### Prepare the Target Machine

Before starting the recovery process:

1. Ensure the target machine's MAC address is registered.

   * https://netapps.bnl.gov/networkregistration/admin/reports/

2. If possible, power off the original machine (`eiclin07`).

### Network Considerations

If the original machine cannot be powered off:

#### Option 1 - Network Not Preconfigured

If you did **not** preconfigure networking in `/etc/rear/local.conf` when creating the rescue ISO:

* Boot the new machine **without network connectivity**.
* After the rescue environment starts, configure the hostname and IP address manually before connecting the network cable.

#### Option 2 - Network Preconfigured

If you configured networking in `/etc/rear/local.conf` when creating the rescue ISO:

* The new machine may be booted with the network connected.

---

### Boot the Target Machine

Power on the new machine and boot from the ReaR USB media.

Select the appropriate menu entry:

#### BIOS Systems

```text id="1e4cwl"
Recover eiclin07
```

#### UEFI Systems

```text id="bq7nbg"
Relax-and-Recover (Secure Boot)
```

---

### Network Interface Replacement Prompt

During recovery, ReaR may ask whether you want to replace one or more network interfaces.

This happens when the target machine uses different network hardware than the source machine.

Simply:

* Answer **Yes**, or
* Select the appropriate replacement interface.

> You may be prompted multiple times if the machine contains multiple NICs.

---

### Log In

Log in as:

```text id="a74gqe"
root
```

No password is required in the rescue environment.

---

### Optional: Configure Networking Manually

If:

* The original machine is still online, and
* Networking was not preconfigured in `local.conf`

then configure networking manually before connecting the machine to the network.

#### Configure the Interface

```bash id="0szv3n"
hostname eiclin08.eic.bnl.gov

ip link
```

Identify the interface name (for example, `eth0`, `eno1`, or `enp2`).

Configure networking:

```bash id="g3bz90"
hostname  eiclin08.eic.bnl.gov
ip addr add 130.199.97.106/23 dev eth0
ip link set eth0 up
ip route add default via 130.199.96.24

ip addr
```

#### Connect the Network

Once the interface has been configured:

* Physical machine: Plug in the network cable.

This avoids IP conflicts with the original machine because the interface remains down until configured.

---

### Start the Recovery Process

```bash id="qrdz7v"
rear recover
```

---

### Optional: Enable Remote SSH Access

Once networking is operational, you can SSH into the recovery environment and continue the recovery remotely.

If SSH access fails due to missing authorized keys:

```bash id="v5tw5q"
mkdir -p /root/.ssh
chmod 700 /root/.ssh

scp root@eiclin07.eic.bnl.gov:/root/.ssh/authorized_keys \
    /root/.ssh

chmod 600 /root/.ssh/authorized_keys
```

---

### Verify Available Space

Check the restored filesystems:

```bash id="x8f2fa"
df -kh
```

Verify that:

```text id="l6e79g"
/mnt/local
```

(the restored root filesystem) is large enough to hold `backup.tar.gz`.

> If `/mnt/local` is too small, you may encounter a **"No space left"** error during restore.

#### Recommended Solution

Use a custom `disklayout.conf` during backup creation to allocate a larger root partition.

See **Appendix I - disklayout.conf**.

#### Alternative Solution

Create or modify:

```text id="69f1ah"
/etc/rear/disklayout.conf
```

during recovery and increase the root partition size manually.

---

### Restore the Backup Archive

Copy the backup archive from the source machine:

```bash id="kewtvf"
scp root@eiclin07.eic.bnl.gov:/eic/opt/installation/ReaR/backup-eiclin07.tar.gz /mnt/local/
```

Extract it:

```bash id="l2ow7v"
tar xvf /mnt/local/backup-eiclin07.tar.gz -C /mnt/local/
```

Remove the archive:

```bash id="ixmn6s"
rm -f /mnt/local/backup-eiclin07.tar.gz
```

Enable SELinux relabeling on first boot:

```bash id="gbr74h"
touch /mnt/local/.autorelabel
```

---

### Optional: Update Hostname and Networking Before Reboot

You may perform these changes now or after the first boot.

Set hostname:

```text id="qk9nwl"
/mnt/local/usr/bin/hostname  [newHostname.eic.bnl.gov]
```

Update:

```text id="qk9nwl"
/mnt/local/etc/hosts
/mnt/local/etc/hostname
```

Edit the NetworkManager profile:

```bash id="nh89vb"
vi /mnt/local/etc/NetworkManager/system-connections/xxx.nmconnection
```

Verify:

* Interface name matches the target machine.
* IP address is correct.

> If the network interface name changed (for example, `eno1` → `enp2`), update both:
>
> * The `xxx.nmconnection` filename
> * The `interface-name=` entry inside the file

---

### Ensure No Stopped Jobs Exist

Check for suspended jobs:

```bash id="9xwddj"
jobs
```

If any exist:

```bash id="36dk5m"
kill %1
kill %2
```

If a job cannot be killed:

```bash id="n1v2g9"
fg %1
```

then terminate it normally.

---

> **IMPORTANT**
>
> The following steps must be performed from the local console of the target machine.

### Complete the ReaR Recovery Process

Exit the recovery shell:

```bash id="nq3w0t"
exit
```

When prompted:

```text id="t9cm1m"
answer yes
select 1
```

Wait for:

* Recreating directories
* Building initramfs (`mkinitrd` / `dracut`)

This may take several minutes.

---

### Reboot the System

Remove the USB drive.

Reboot:

```bash id="frv6hm"
reboot
```

> The first boot performs a complete SELinux relabel because:
>
> ```bash
> touch /mnt/local/.autorelabel
> ```
>
> was created during recovery.
>
> This process may take **10 minutes or longer** depending on the system.
>
> Be patient.

---

## VI. Post-Configuration and Verification

After the recovery process completes and the system has successfully rebooted, perform the following post-configuration tasks.

### Connect to the New Machine

```bash id="lzy9lb"
ssh root@newhostname.eic.bnl.gov
```

### Update Hostname

Set the permanent hostname:

```bash id="saz6j7"
/usr/bin/hostnamectl set-hostname newhostname.eic.bnl.gov
```

### Verify Storage Configuration

Review the LVM and boot configuration to ensure all components reference the same volume group (VG).

* vgs
* lvs
* cat /etc/fstab
* cat /etc/kernel/cmdline
* cat /etc/default/grub 

Example:
VG Name: almalinux_eiclin07

All boot-critical configuration files should reference the same VG name.

> **IMPORTANT**  
> Do not perform a blanket hostname replacement such as:  
> sed -i "s/eiclin07/eiclin08/g;" $(find /etc/ -type f)  
> This may unintentionally modify LVM, GRUB, kernel, or other boot-related configuration files and render the system unbootable.  


### Verify Network Configuration

Review the network configuration and ensure that:

* Hostname is correct
* IP address is correct 
* MAC address is correct 
* Network interface names match the new hardware 

### Rejoin Centrify / Active Directory

Leave the existing Centrify domain membership:

```bash id="j9x7w5"
/usr/sbin/adleave -r -u eicadmin
```

Join the new machine to Active Directory:

```bash id="dkth7u"
/usr/sbin/adjoin -V \
  -c "OU=EIC-CON,OU=EIC,OU=CCM,DC=bnl,DC=gov" \
  --zone "OU=EIC-CON,OU=EIC,OU=CCM,DC=bnl,DC=gov" \
  --user eicadmin \
  --force \
  bnl.gov
```

> **IMPORTANT**
>
> Joining the cloned machine to Centrify may remove the original machine from the Centrify zone.
>
> If that occurs, run adleave and adjoin on the original machine to restore its domain membership. 

### Update Ansible

Add the new machine to the appropriate Ansible inventory.


### Final Verification

* Filesystems are mounted correctly
* Network connectivity is functional
* Hostname is correct
* Required services start successfully
* Active Directory authentication works (adinfo)
* New machine in Ansible inventory 

At this point, the cloned machine should be fully operational and ready for production use.

---


## Appendix I - `disklayout.conf`

### Disk Layout Considerations During Recovery

In this example:

* The original system disk (`/dev/sda`) is **256 GB**.

* The root partition (`/dev/sda8`) is only **8 GB**.

* During recovery, the backup archive (`backup.tar.gz`) is typically copied to /mnt/local

* Because `/mnt/local` represents the restored root filesystem, an 8 GB root partition is not large enough to hold the backup archive during the recovery process.

### Solution

To provide sufficient space for the restore process:

* Increase the target disk size from **256 GB** to **356 GB**.
* Increase the root partition size from **8 GB** to **92 GB**.

This provides enough temporary space for:

* `backup.tar.gz`
* extraction of the archive
* SELinux relabeling
* recovery operations

> **NOTE**
>
> Based on testing, a single partition defined in `disklayout.conf` may fail if it exceeds **100 GB**.
>
> If larger storage is required, consider:
>
> * Keeping individual partitions below 100 GB
> * Splitting data across multiple partitions
>
> In most cases, a root (`/`) partition of **25 GB** is sufficient for the recovery process.

---

### Original Layout

```text id="3h2r5u"
# Disk /dev/sda
# Format: disk <devname> <size(bytes)> <partition label type>

disk /dev/sda 256060514304 msdos

# Partitions on /dev/sda
# Format: part <device> <partition size(bytes)> <partition start(bytes)> <partition type|name> <flags> /dev/<partition>

part /dev/sda 2147483648   1048576        primary boot  /dev/sda1
part /dev/sda 64424509440  2148532224     primary none  /dev/sda2
part /dev/sda 32212254720  66573041664    primary none  /dev/sda3
part /dev/sda 157274865664 98785296384    extended none /dev/sda4
part /dev/sda 32212254720  98787393536    logical none  /dev/sda5
part /dev/sda 12884901888  131000696832   logical none  /dev/sda6
part /dev/sda 12884901888  143886647296   logical none  /dev/sda7
part /dev/sda 8589934592   156772597760   logical none  /dev/sda8
```

Root partition size:

```text id="k4hpmw"
/dev/sda8 = 8 GB
```

---

### Modified Layout

```text id="g7hj7q"
# Disk /dev/sda

disk /dev/sda 356060514304 msdos

# Partitions on /dev/sda
# Format: part <device> <partition size(bytes)> <partition start(bytes)> <partition type|name> <flags> /dev/<partition>

part /dev/sda 2147483648   1048576        primary boot  /dev/sda1
part /dev/sda 64424509440  2148532224     primary none  /dev/sda2
part /dev/sda 32212254720  66573041664    primary none  /dev/sda3
part /dev/sda 157274865664 98785296384    extended none /dev/sda4
part /dev/sda 32212254720  98787393536    logical none  /dev/sda5
part /dev/sda 12884901888  131000696832   logical none  /dev/sda6
part /dev/sda 12884901888  143886647296   logical none  /dev/sda7
part /dev/sda 92374182400  156772597760   logical none  /dev/sda8
```

Root partition size:

```text id="tv0c3g"
/dev/sda8 = 92 GB
```

This layout provides sufficient space for the backup archive and recovery operations while remaining within the practical limits observed during testing.

---

## Appendix II - Creating ReaR Rescue and Backup Media Directly on a USB Drive

Reference:

* http://relax-and-recover.org/documentation/getting-started

This method creates both the ReaR rescue environment and backup archive directly on a USB device.

### Prepare the USB Device

Format the USB device.

```bash id="z2d3bq"
rear format /dev/sdb
```

Label the device:

```text id="z6m74p"
REAR-000
```
---

### Configure ReaR

Edit /etc/rear/local.conf:

```bash id="aj4mwy"
OUTPUT=USB
BACKUP=NETFS
BACKUP_URL=usb:///dev/disk/by-label/REAR-000
```
---

### Create the Rescue Environment

Generate the rescue image:

```bash id="tn8h7m"
rear -v mkrescue
```

This writes the rescue environment directly to the USB device.

---

### Test the Rescue USB

Reboot the machine and boot from the USB device.

Verify that:

* The system boots successfully into the ReaR rescue environment.
* Storage devices are detected correctly.
* Network interfaces are available if required.

---

### Create a Full Backup

Once the USB rescue environment has been verified, create a full backup:

```bash id="4nl1y9"
rear -v mkbackup
```

This stores the backup archive on the USB device specified by:

```bash id="17o4eo"
BACKUP_URL=usb:///dev/disk/by-label/REAR-000
```

---

### Recovery

In the event of a disk failure:

1. Replace the failed disk.
2. Boot from the ReaR USB device.
3. Follow the standard ReaR recovery procedure.

The USB device contains both:

* The rescue environment
* The backup archive

allowing the system to be restored without requiring additional media or network storage.

> At this point, your hard disk can safely fail.

---
