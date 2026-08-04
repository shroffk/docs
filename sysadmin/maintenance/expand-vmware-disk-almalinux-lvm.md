# Expanding a VMware Virtual Disk on AlmaLinux (LVM + XFS)

After resizing a VM's virtual disk in the VMware console, the guest OS does **not** automatically see the new space. The following steps grow the partition, the LVM physical volume, and the logical volume / filesystem online — no reboot required.

## Context

- Guest: AlmaLinux, XFS on LVM
- Layout: `/dev/sda1` (EFI), `/dev/sda2` (`/boot`), `/dev/sda3` (LVM PV) → VG `almalinux` → LVs `root`, `swap`, `home`
- Example: expanded VMware disk from 172 GB → 1 TB, all new space targeted at `/` (root was at 82% used)

## Steps

### 1. Rescan the SCSI device so the kernel sees the new size

```bash
echo 1 | sudo tee /sys/class/block/sda/device/rescan
```

Verify with `lsblk` — `sda` should now show the new total size.

### 2. Grow partition 3 to fill the disk

```bash
sudo growpart /dev/sda 3
```

If `growpart` is missing: `sudo dnf install cloud-utils-growpart`.

### 3. Resize the LVM physical volume

```bash
sudo pvresize /dev/sda3
```

Confirm free space is now in the VG:

```bash
sudo vgs
```

### 4. Decide where the space should go

Check current usage before extending:

```bash
df -h
```

Send all free space to whichever LV needs it. To grow root:

```bash
sudo lvextend -r -l +100%FREE /dev/almalinux/root
```

Other examples:

```bash
# Add a fixed amount
sudo lvextend -r -L +200G /dev/almalinux/root

# Split between LVs
sudo lvextend -r -L +200G /dev/almalinux/root
sudo lvextend -r -l +100%FREE /dev/almalinux/home
```

The `-r` flag grows the filesystem in the same step (works for XFS and ext4).

### 5. Verify

```bash
df -h
sudo lvs
```

## Notes & Gotchas

- **XFS can grow online but cannot shrink** — be deliberate about which LV gets the space.
- If step 1 still shows the old size, check for an active VMware snapshot — the disk won't actually be larger until the snapshot is consolidated.
- No reboot is required at any point.
