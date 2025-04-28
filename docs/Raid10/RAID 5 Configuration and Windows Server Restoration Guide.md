# RAID Configuration and Backup Restoration Guide

*A comprehensive guide for configuring RAID 5 and restoring Windows backup*

## 📋 Table of Contents

- [Overview](#overview)
- [System Specifications](#system-specs)
- [RAID Options Comparison](#raid-comparison)
- [RAID 5 Configuration](#raid5-config)
- [Disk Initialization](#disk-init)
- [Backup Restoration](#backup-restore)
- [Troubleshooting](#troubleshooting)
- [Performance Optimization](#performance)
- [Maintenance Best Practices](#maintenance)

<a id="overview"></a>
## 🔍 Overview

This guide provides detailed instructions for configuring a RAID 5 array on a Dell PowerEdge server and restoring a Windows backup that was originally created on a RAID 10 configuration. RAID 5 has been selected as the optimal solution that balances simplicity, storage efficiency, and adequate redundancy.

[Back to TOC](#-table-of-contents)

<a id="system-specs"></a>
## 💻 System Specifications

Based on the system information:

- **Server Model**: Dell PowerEdge R740xd
- **Processor**: Intel Xeon Gold 6130 CPU @ 2.10GHz (64 CPUs)
- **Memory**: 786GB RAM
- **Operating System**: Windows Server 2019 Datacenter 64-bit
- **Available Drives**: 8 drives @ 1.2TB each (6 drives will be used for RAID)
- **Previous Configuration**: RAID 10
- **BIOS Version**: 2.2.11

[Back to TOC](#-table-of-contents)

<a id="raid-comparison"></a>
## ⚖️ RAID Options Comparison

| RAID Level | Usable Space (with 6 x 1.2TB drives) | Redundancy | Performance | Complexity |
|------------|--------------------------------------|------------|-------------|------------|
| RAID 0     | 7.2TB (6 x 1.2TB)                   | None       | Excellent   | Low        |
| RAID 1     | 3.6TB (6 x 1.2TB ÷ 2)               | Excellent  | Good        | Low        |
| RAID 5     | 6.0TB (5 x 1.2TB)                   | Good       | Good        | Medium     |
| RAID 6     | 4.8TB (4 x 1.2TB)                   | Excellent  | Moderate    | Medium     |
| RAID 10    | 3.6TB (6 x 1.2TB ÷ 2)               | Very Good  | Excellent   | High       |

**Recommendation**: RAID 5 provides the best balance of usable space, redundancy, and simplicity for your requirements.

[Back to TOC](#-table-of-contents)

<a id="raid5-config"></a>
## ⚙️ RAID 5 Configuration

### Step 1: Access RAID Controller BIOS
1. **Power on** the server
2. During POST, press the appropriate key combination:
   - For Dell PERC controllers: `Ctrl+R`
   - For LSI controllers: `Ctrl+C`
   - For other controllers: Check the boot screen for the correct key

### Step 2: Navigate RAID Configuration Utility
1. Select the appropriate RAID controller from the list if multiple controllers are present
2. Navigate to **"Controller Management"** or **"Configuration Management"**
3. Select **"Create Virtual Disk"** or equivalent option

### Step 3: Configure the RAID 5 Array
1. Select **RAID level**: Choose `RAID 5` from the available options
2. **Select physical disks**:
   - Mark 6 of the 1.2TB physical disks
   - Ensure all selected disks are in "Ready" state
3. **Configure Virtual Disk parameters**:
   - **Stripe Size**: Set to `64 KB` (optimal for general server use)
   - **Read Policy**: Set to `Adaptive Read Ahead`
   - **Write Policy**: Set to `Write Back with BBU` if available, otherwise `Write Through`
   - **I/O Policy**: Set to `Cached I/O`
   - **Disk Cache Policy**: Set to `Enabled`
   - **Initialize**: Select `Background Initialization`
4. **Name** the virtual disk (e.g., "RAID5_System")
5. **Confirm** and save the configuration

### Step 4: Finalize RAID Creation
1. The controller will begin initializing the RAID array
2. **Exit** the RAID configuration utility
3. **Restart** the server

> ⚠️ **Note**: RAID initialization may take several hours to complete, but the system can be used during this time with slightly reduced performance.

[Back to TOC](#-table-of-contents)

<a id="disk-init"></a>
## 💽 Disk Initialization

### Step 1: Boot to Windows Environment
1. Boot using Windows Server installation media or recovery environment
2. Select **"Repair your computer"** if using installation media

### Step 2: Initialize Disk via Command Line
1. Open **Command Prompt** from the recovery environment
2. Launch **Diskpart** by typing:
   ```
   diskpart
   ```
3. View available disks:
   ```
   list disk
   ```
4. Select the new RAID virtual disk (verify by size, should be approximately 6TB):
   ```
   select disk N
   ```
   (where N is the disk number from the list)
5. Clean the disk:
   ```
   clean
   ```
6. Convert to GPT partition style:
   ```
   convert gpt
   ```
7. Create a primary partition:
   ```
   create partition primary
   ```
8. Format the partition:
   ```
   format quick fs=ntfs label=System
   ```
9. Assign a drive letter:
   ```
   assign letter=C
   ```
10. Exit Diskpart:
    ```
    exit
    ```

> 💡 **Tip**: If you need multiple partitions, you can create them now using the `create partition primary size=XXXX` command, where XXXX is the size in MB.

[Back to TOC](#-table-of-contents)

<a id="backup-restore"></a>
## 🔄 Backup Restoration

### Step 1: Prepare for Restoration
1. Ensure the backup media (external drive, network location, etc.) is accessible
2. If using a network backup, connect the server to the network
3. Boot to the **Windows Recovery Environment** using Windows Server installation media

### Step 2: Access System Image Recovery
1. From the Windows Setup or recovery environment, select **"Repair your computer"**
2. Choose **"Troubleshoot"**
3. Select **"System Image Recovery"**

### Step 3: Locate and Select Backup
1. When prompted, choose **"Select a system image"**
2. Browse to your backup location and select the system image
3. Choose the most recent backup if multiple backups are available

### Step 4: Configure Restoration Options
1. When prompted about disk configuration differences (since you're moving from RAID 10 to RAID 5):
   - Select the option to **"Format and repartition disks"**
   - If available, check the option **"Only restore system drives"**

2. Review the restoration settings and click **"Next"**

### Step 5: Complete Restoration
1. Confirm the restoration by clicking **"Finish"**
2. Acknowledge any warnings that appear
3. The restoration process will begin, which may take several hours depending on the size of the backup
4. Once completed, the system will restart automatically

> ⚠️ **Important**: Do not interrupt the restoration process once it has begun.

[Back to TOC](#-table-of-contents)

<a id="troubleshooting"></a>
## 🔧 Troubleshooting

### Issue: Backup Not Recognized
- **Solution**: Ensure the backup media is properly connected
- Verify that backup files are not corrupted
- Try using Windows Backup and Restore from within a temporary Windows installation

### Issue: RAID Controller Not Detecting All Drives
- **Solution**: Check physical connections
- Update RAID controller firmware
- Verify drive compatibility with the controller

### Issue: RAID Initialization Fails
- **Solution**: Check drive health status
- Replace any faulty drives
- Restart the initialization process

### Issue: Windows Won't Boot After Restoration
- **Solution**: Boot into recovery environment
- Use **"Startup Repair"** option
- Verify boot configuration data using `bcdedit` command

[Back to TOC](#-table-of-contents)

<a id="performance"></a>
## ⚡ Performance Optimization

### RAID Controller Settings
- **Stripe Size**: 64KB is optimal for general use; increase to 256KB for large file operations
- **Read Policy**: Adaptive Read Ahead for mixed workloads
- **Write Policy**: Write Back with BBU for optimal performance (requires battery backup)

### Windows Optimization
- **Disable** unnecessary services
- **Configure** power settings for high performance
- **Align** partitions properly (modern tools do this automatically)

[Back to TOC](#-table-of-contents)

<a id="maintenance"></a>
## 🔒 Maintenance Best Practices

### Regular Tasks
- **Schedule** weekly consistency checks
- **Monitor** drive health using management tools
- **Update** RAID controller firmware regularly

### Backup Strategy
- **Continue** regular system backups
- **Test** backup restoration periodically
- **Store** backups in multiple locations

### Failure Preparation
- **Keep** spare drives available
- **Document** the RAID configuration
- **Create** a bootable recovery media

[Back to TOC](#-table-of-contents)