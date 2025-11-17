---
title: "OEM Fresh New LOOK Configuration"
date: "2025-11-17"  
permalink: "/posts/2025/11/OEM Fresh New LOOK Configuration-202511/"  
tags:
  - OEM Fresh Configuration
  - silent_installation
---

OEM Fresh New LOOK Configuration 

Drafting Guideline ver 202511

## Phase 1: Initial System Preparation & Stability Test

**Objective**: Bypass NRO (Network Registration OOBE), power check, and activation detection to ensure hardware and system stability.

### Operational Procedure

1.  **Bypass Microsoft Account Login during OOBE** using `Shift + F10` command prompt.
    *   Simple OOBE Bypass:
        ```bat
        oobe\bypassnro
        ```
    *   OOBE Bypass via Registry:
        ```bat
        reg add "HKLM\SYSTEM\Setup\LabConfig" /v BypassNRO /t REG_DWORD /d 1 /f
        shutdown /r /t 0
        ```
    *   Local Account Only Method:
        ```bat
        start ms-cxh:localonly
        ```

2.  **Hardware Stability Test** using Tools like "TuBa Toolbox" or "/Stress Test":
    *   Confirm hardware stability, then record OS version, BIOS, and firmware information.
    *   CPU/GPU Stress Test (e.g., "FurMark", "AIDA64").
    *   Power Load Stability Test.
    *   Temperature & Voltage Monitoring.
    *   Screen Backlight Bleeding Inspection.
    *   **Prerequisite: Have the device's rated power specifications available for reference during testing.**

3.  **Generate Battery Usage Report** using `powercfg`:
    ```bat
    powercfg /batteryreport /output "%userprofile%\Desktop\BatteryReport.html"
    ```
    The report file is generated on the Desktop. Analyze the `BatteryReport.html` to check power-on time, which can indicate if the system was pre-installed and used previously.

4.  **Proceed only after all checks above are confirmed.** Otherwise, initiate the return process.

5.  **Note**: The system is **NOT connected to the internet nor activated** in this phase.
-----

## Phase 2: OEM Software Analysis and Cleanup

**Objective**: Proactively identify, evaluate, and clean up useless OEM pre-installed software, while understanding its recovery mechanism to lay the groundwork for a thorough solution.

### Operational Procedure

#### 1. Comprehensive Inventory of Pre-installed Software

- **Method A (System Interface)**: Navigate to `Settings` > `Apps` > `Installed apps` to view the complete list.
- **Method B (PowerShell)**: Run PowerShell as Administrator and execute the following command to obtain a more detailed list than the GUI provides:
  ```powershell
  Get-Package | Select-Object Name, Version, ProviderName | Sort-Object ProviderName | Export-Csv -Path "$env:USERPROFILE\Desktop\OEM_Software_Inventory.csv" -NoTypeInformation
  ```
- **Deliverable**: Generate a complete pre-installed software inventory (`OEM_Software_Inventory.csv`).

#### 2. Identification and Judgment (Establishing Classification Criteria)

##### Designated Retention Items

1.  **Core Drivers & Firmware**: Essential drivers, BIOS/firmware utilities, and their companion tools that ensure proper hardware operation and performance.
2.  **Exclusive Functionality Components**: OEM-provided software that integrates deeply with the hardware and offers added value (e.g., unique performance modes, hardware status monitoring, or diagnostic tools).

##### Designated Cleanup Items

1.  **Trialware & Feature-Limited Software**: Clearly labeled trial versions, subscription-based software, or pre-installed applications whose core features require additional payment to unlock.
2.  **Third-Party Promotional Content**: Third-party commercial promotions and applications pre-installed by the OEM that are non-essential for system or hardware operation.
3.  **Redundant Functional Software**: Non-essential applications developed by the OEM whose functionality can be replaced by built-in OS components or superior professional software available on the market.

##### System Optimization Items

1.  **Interface & User Experience**: Disable or remove built-in system promotional content, non-essential notifications, and visual effects to enhance the purity of the user experience.
2.  **Privacy & Updates**: Adjust system privacy settings and Windows Update behavior according to policy, balancing security and convenience.
3.  **Power & Performance**: Create or select a power plan suitable for the usage scenario, and optimize system startup items and services to improve responsiveness and energy efficiency.

#### 3. Execution of Cleanup

- **Primary Method**: Locate target software in `Settings` > `Apps` > `Installed apps` and **Uninstall** directly.
- **Bulk/Stubborn Software**: Use **Dism++**'s "APPX Management" and "Driver Management" features, or specialized third-party uninstaller tools for a thorough cleanup.
- **Note**: For uncertain software, it is recommended to first search online for its name and function to confirm it is useless before uninstalling.

#### 4. Documenting OEM Recovery Sources (Key Insight)

- **Core Discovery**: Inform the user that the current uninstallation of OEM software is not permanent. OEMs deploy this software during system initialization using specific PPKG pre-provisioning packages.
- **File Locations**: These critical PPKG packages are typically located in:
    - The System Recovery partition
    - `C:\Recovery`
    - `C:\Windows\Provisioning\Packages`
- **Important Note**: This means that if the **"Recovery" or "Reset this PC"** function is used in the future, the system will **automatically reinstall all OEM pre-installed software based on these PPKG packages**. The current manual cleanup is only a surface-level treatment; these PPKG packages are the root cause that needs further research and handling.

#### 5. Post-Cleanup and Documentation

- After cleanup is complete, restart the computer and confirm the system operates normally without any critical function loss.
- **Output Documentation**:
    - **`OEM_Software_Cleanup_List`**: Documents the names of uninstalled software and the rationale for removal.
    - **`Recommended_Retention_Software_List`**: Documents OEM software recommended for retention and the reasons why.
    - **`Key_Findings_Record`**: Clearly records the location of PPKG files and the important conclusion that "Reset this PC" will restore the software, serving as a warning for the user or a basis for further in-depth handling.

----

## Phase 3: Software Installation and Environment Configuration

**Objective**: Install necessary software and configure the user environment.

### Operational Procedure

#### 1. Execute Installation Scripts
- Run installation scripts in the order specified in the setup documentation.

#### 2. Offline Activation
- **Note**: This phase can be completed offline. All software should be activated using offline methods.

### Key Focus Areas

#### 1. Critical Driver Installation
- **Verify GPU & CUDA Compatibility**: First, run `nvidia-smi` in Command Prompt to confirm GPU model and current driver version.
- Install CUDA and CUDNN Drivers.

#### 2. Essential Windows 11 Configuration
- Apply Group Policy settings (Home Edition workarounds)
- Disable automatic updates indefinitely
- **Relocate User Folders**: Use Windows 11 Settings (`Settings > System > Storage > Advanced storage settings > Where new content is saved`) to redirect Downloads, Documents, etc., from C: to D: drive.
- Disable BitLocker encryption

#### 3. System Optimization
- Apply additional system optimizations using Dism++

#### 4. Startup Management
- Disable auto-start items for installed applications using AutoRun

#### 5. Browser Configuration
- Configure Edge browser policies


----


# Phase 4: PE Offline Backup Guide

## 1. Backup Core Concepts

### 1.1 The Nature of Windows System Backup
When Windows system is running:
- System files cannot be directly copied
- Registry hive files remain open
- Drivers are loaded into memory

### 1.2 Backup Format Analysis

**WIM Format Advantages**:
- File-based image format, not sector-level
- Supports incremental backup, saves storage space
- Can be mounted to view and modify content
- High compression rate, supports LZX compression algorithm

## 2. Backup Environment Selection

### 2.1 PE Environment vs Online System

| Environment | Backup Integrity | Use Case | Risk Level |
|-------------|------------------|----------|------------|
| **PE Environment** | 100% Complete | Critical system backup, factory state preservation | Low Risk |
| **Online System** | Relies on VSS shadow copy | Temporary snapshots, daily backups | Medium Risk |

### 2.2 Why PE Environment Backup is Recommended
- **No file locking**: System partition completely unmounted
- **No runtime interference**: Avoids system service and application impacts
- **Registry integrity**: Registry hive files are closed and fully backed up
- **Activation status preservation**: OEM activation information remains intact

## 3. Dism++ Backup Details

### 3.1 Tool Preparation

```
Essential components:
├── MicroPE bootable USB (recommended: WePE_64_V2.3.iso)
├── Dism++ 10.1.100.2 or newer version
└── External storage device (NTFS format, capacity ≥32GB)
```

### 3.2 Backup Naming Convention
```
[Brand_Series]_[OS_Version]_[Version_Number]_[Status]_[Date].wim

Examples:
├── LEGION_Win11_24H2_Clean_20251019.wim
├── LEGION_Win11_24H2_Soft_NoActivate_20251019.wim
└── LEGION_Win11_24H2_Full_Activate_20251019.wim

Naming field description:
• Brand Series: LEGION, ThinkBook, YOGA, etc.
• OS Version: Win11, Win10
• Version Number: 24H2, 23H2, etc.
• Status: Clean (clean system), Soft (software configured), Full (fully activated)
• Date: Backup creation date (YYYYMMDD)
```

### 3.3 Backup Type Definitions

| Backup Type | Content | Purpose |
|-------------|---------|---------|
| **Clean System** | OEM factory state, no modifications | System restoration baseline |
| **Soft Configuration** | Drivers + essential software, not activated | Quick deployment environment |
| **Full Complete** | All software + configuration + activation | Complete working environment |

## 4. Practical Operation Process

### 4.1 Entering PE Environment
1. Boot from MicroPE bootable USB
2. Wait for PE system to fully load
3. Open Dism++ tool

### 4.2 Dism++ Backup Settings

**PE Environment Settings**:
- ✅ Bootable image: Ensures system can boot after recovery
- ✅ Shadow copy: Ensures backup consistency
- 🔄 Compression level: Choose based on requirements
  - Fast: Faster backup speed, larger file size
  - Maximum: Slower backup speed, smallest file size (recommended LZX algorithm)

**Online System Backup Settings**:
- ✅ Bootable: Must be checked
- ✅ Shadow copy: Must be checked
- ⚠️ Note: May cause data inconsistency due to file occupation

### 4.3 Executing Backup
```
Backup path: D:\Backup\  # Recommended external hard drive
Backup format: WIM
Key options:
  • Shadow copy: Enabled
  • Bootable: Enabled  
  • Compression algorithm: LZX (best compression rate)
```

## 5. Key Technical Points

### 5.1 OEM Activation Protection
- PE backup completely preserves digital license activation
- Avoids breaking OEM activation chain that may occur with online backup
- No reactivation required after recovery

### 5.2 System Consistency Assurance
PE environment backup ensures:
- No pending system updates
- No partially written registry entries
- No locked system files

### 5.3 Backup Verification Methods
After backup completion, verify:
1. WIM file can be mounted to view content
2. File size meets expectations
3. Successful restoration test in PE environment

## 6. Recovery Strategy

### 6.1 Recovery Environment Requirements
- Same or updated MicroPE environment
- Dism++ tool
- Sufficient disk space

### 6.2 Recovery Precautions
- Backup important data before recovery
- Ensure correct target disk partition
- Repair boot record after recovery

## 7. Summary Recommendations

**Scenarios requiring PE environment backup**:
- Initial backup of OEM factory system
- Baseline backup after system configuration
- System states requiring long-term preservation

**Scenarios suitable for online backup**:
- Temporary system snapshots
- Daily backups of non-critical data

Using Dism++ with WIM format backup in PE environment is the most reliable solution for ensuring Windows system integrity and recoverability.

---

## Important Notes

1. **OEM System Reset Mechanism**: Default reset typically involves ppkg rollback, which will restore some OEM software and auto-start items, not completely equivalent to a "clean system"

2. **Backup Timing**: Backing up WIM before connecting to internet ensures having an "unactivated/unconnected" baseline image that can be restored at any time

3. **Activation Status**: Pay attention to maintaining or recording system activation status at each phase



