# Migration: OpenMediaVault 7 to 8 with ZFS & PVE Kernel

This document records the major upgrade process for an OpenMediaVault (OMV) server using ZFS as the primary filesystem.  
Since ZFS has high kernel dependencies, a strategy of uninstalling and reinstalling the plugin was adopted to prevent kernel module (DKMS) conflicts during the upgrade process.

## 🛠️ Environment

- **OS: Debian 12 (Bookworm) with OMV 7**  
- **Kernel:** 6.11.11-2-pve
- **Filesystem:** ZFS
- **Key Services:** Services on Docker, SMB, SFTP

## 🎯 Target

**Upgrade Debian 12 with OMV 7 system to Debian 13 with OMV 8**  
**Upgrade Proxmox VE 8 (6.11) Kernel to Proxmox VE 9 (6.14) Kernel**  

## 🗿 Migration Strategy

**To resolve dependency issues between ZFS kernel modules and the new OS kernel, proceed with the following steps.**  

1. [Service & Data Isolation: Stop SMB, SFTP, and Docker containers](#phase-1-service--data-isolation)

2. [ZFS Protection: Unmount and Export Pool to ensure data integrity](#phase-2-zfs-protection)

3. [Dependency Removal: Pre-emptively remove openmediavault-zfs plugin to prevent DKMS errors during upgrade](#phase-3-dependency-removal)

4. [OS Release Upgrade: Migration to OMV 8 and Debian 13](#phase-4-os-release-upgrade)

5. [Infrastructure Rebuild: Install latest PVE Kernel (6.14) and restore ZFS environment](#phase-5-infrastructure-rebuild)

### Phase 1. Service & Data Isolation

**Block data write operations and free up system resources during the upgrade.**  

* **Stop SMB/SFTP Services:** Disable sharing services using the Web GUI  

* **Stop Docker Containers:** Stop containers mounting ZFS datasets  

```bash
# When using Docker Compose
docker-compose down
# Docker Container
docker stop <CONTAINER_1> <CONTAINER_2> <CONTAINER_3> ...
```

### Phase 2. ZFS Protection

**Safely detach the Pool to prevent potential data corruption during kernel module updates.**  

* **ZFS Pool Export:** Unmount all pools and safely switch them to exported state from the system  

```bash
sudo zpool export -a
```

### Phase 3. Dependency Removal

**Prevent build conflicts (DKMS Error) between old ZFS modules and the new kernel during the major upgrade.**  

* **Remove ZFS Plugin:** Remove the plugin to break kernel module dependencies, while keeping configuration values  

```bash
# Remove ZFS Plugin
sudo apt remove openmediavault-zfs
# Remove dependent packages
sudo apt autoremove
```

### Phase 4. OS Release Upgrade

**Transition from Debian 12 (Bookworm) base to Debian 13 (Trixie) base OMV 8.**  

* **Release Upgrade:** Execute OMV official upgrade tool  

```bash
sudo omv-release-upgrade
```

* **Reboot system after all processes are complete**  
```bash
sudo reboot now
```

* **Update OMV-Extras Repository:** Reset plugin repository matching the new OS version (Synchrony)  

```bash
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
```

* **[Troubleshooting: Ping Permission Issue](./Troubleshooting/00_Ping_Permission_Issue.md)**  

* **[Troubleshooting: Network Issue](./Troubleshooting/01_Network_Issue.md)**  

### Phase 5. Infrastructure Rebuild

**Restore the latest kernel and storage environment according to the new system environment.**

* **System Update**  

```bash
sudo apt update && sudo apt upgrade -y
```

* **[Troubleshooting: Docker Repository 404 Not Found](./Troubleshooting/02_404_Not_Found.md)**  
    > **Note:** Confirmed that this issue no longer occurs as of January 3, 2026

* **Install PVE 9 (6.14) Kernel:** Install Proxmox VE Kernel from `Web GUI System > Kernel` (Install PVE 9.1 / 6.17 if needed) and reboot  

* **Reinstall ZFS Plugin:** Reinstall OMV 8 ZFS plugin to build modules for the new kernel  

```bash
sudo apt install openmediavault-zfs
```

* **ZFS Pool Import:** Import the exported pools back into the system  

```bash
# Search ZFS Pool (Verify all ZFS Pools are visible)
sudo zpool import
# Import specific ZFS Pool
sudo zpool import <POOL_NAME>
# Import all ZFS Pools
sudo zpool import -a
```

* **Modify Boot Symbolic Links:** Manually update `/vmlinuz` and `/initrd.img` to ensure they do not point to old kernel versions  

```bash
sudo ln -sf /boot/vmlinuz-6.14.11-5-pve /vmlinuz
sudo ln -sf /boot/initrd.img-6.14.11-5-pve /initrd.img
sudo update-grub
```
