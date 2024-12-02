---
title: "Silent Installation Deployment Guide"
date: "2024-12-02"  
permalink: "/posts/2024/12/silent_install_guide_windows/"  
tags:
  - guide
  - silent_installation
---

This article provides the guide on the **Silent Installation Project** for lab members, including deployment experiences, lessons learned, and critical challenges during implementation.

---
## 1. Introduction

What is called "silent installation"? 

**It is a simple way without any operations for installation. 

Why Do I implement this project?

** The purpose of this project is to streamline the installation process by enabling silent installations, reducing manual intervention, improving deployment efficiency, and ensuring consistency across multiple environments. This approach minimizes installation errors, saves time, and enhances the overall user experience. 

** More importantly, lab members, including chemists and biologists, will be able to configure their software settings easily and save time, without the need to repeatedly click “Next” during the installation process.

How to Use This Guide？

** This guide is structured to help you quickly and effectively perform a silent installation. Follow the steps in sequence, or jump to specific sections for troubleshooting or advanced configuration.

## 2. Prerequisites

### Hardware Recommendation

- **Disk**: SSD,(support NVme), at least 1T for system partition Preferred 2 SSD (not 2 partitions.)
- **Memory**: at least 16GB. 

### System Requirements 
- **Operating System**: Windows 10 or later  
- **Additional Software**: .NET Framework 3.5 (netfx35) must be installed

### Necessary Permissions

- All scripts must be run with **administrator privileges**.
- To run a script with administrator privileges:
  1. Right-click the script file (e.g., .bat or .ps1).
  2. Select "Run as administrator" from the context menu.
  3. Confirm any prompts from the User Account Control (UAC) to allow the script to execute with elevated privileges.

### Software Tools

This part will provide the classification and recommendation softwares for silent installation deployment.

- **SandBox Like**: Virtual Machines (HyperV/VMWare), Windows Sandbox
- **Imaging Software Package Tools**: UltraISO, 7z
- **Code Editing**: VSCode, Notepad++ .etc
- **Analyzing Registry**:Beyond Compare, Registry Work

## 3. Installation Package Overview

Currently, many installation packages are created using popular packaging tools to streamline the deployment process. The most commonly used tools include:

- **NSIS (Nullsoft Scriptable Install System)**
- **MSI (Microsoft Installer)**
- **Inno Setup**
- **InstallShield**
- **Other**

Herein，this article could only discuss the switch of setup command, seldomly mention some reversed engineering methods. If interested, some references are given below.

https://nsis.sourceforge.io/Docs/Chapter3.html#3.2.1

https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/msiexec

https://jrsoftware.org/ishelp/index.php?topic=setupcmdline

If you are not sure, ask ChatGPT for prompt and try it.

## 4. Silent Installation Code Building Workflow

This part will module each parts as follows:

- **Platform Establishment**
: Instructions of testing environment establishment, how to build up a successful platform for further research.

- **Testing in Sandbox**
: How to test codes or software installation safe.

- **Analyzing and Coding**
: When settings completes, how should I know the full codes for building?

- **Encapusing within an Image**
: How to properly integrate those codes and softwares properly into a ISO images.

- **Distribution**
:All full files were uploaded to github and release an ISO images.

Many steps are interconnected. Please avoid viewing them in isolation or treating them as standalone processes.

## 5. Platform Establishment As Example

Herein, I will post my experiences as example.

### Lists of hardware 

- **Disk**: SSD,(support NVme), at least 1T for system partition Preferred 2 SSD (not 2 partitions.)
- **Memory**: at least 16GB. 
- **Operating System**: Windows 10 or later  

### Lists of software (Download link)

Especially, some bad search engines in China, mislead the right link.

- **7z**: https://www.7-zip.org/ 
- **UltraISO**: https://www.ultraiso.com/ 
- **VMWare Workstation**: Please use direct link to download and find key by Google.
- **VSCode**: https://code.visualstudio.com/ 
- **BeyondCompare4**: https://www.scootersoftware.com/ Only ver. 4 could copy key text for cracking.
- **RegistryWork**: https://www.torchsoft.com/en/rw_information.html 

For sake of integrity, please check SHA254 if necessary.

These softwares should install them in your host machines, rather than virtual machines.

## 6. Sandbox Environment Establishment

### Windows 10 or later running on VMWare

In this part, I will simply these operations for short. Some operations will give official references or simple codes for settings.

- **1.** Please prepare windows iso image. 

Please browse http://msdn.itellyou.cn to fetch it.

- **2.**  Creating New Virtual Machine 

Ref: https://docs.vmware.com/en/VMware-Workstation-Pro/17/com.vmware.ws.using.doc/GUID-C55E5599-346F-40D4-8ECD-7CE86964730E.html

Recommended: 75GB at least, 8GB RAM

- **3.**  Running Windows Installation 

**Username should not be any Chinese characters**

**For Windows 11 user, please use oobe to bypass.
Open cmd with Shift + F10 and type OOBE\BYPASSNRO.

- **4.** Close UAC

Ref:https://learn.microsoft.com/en-us/windows/security/application-security/application-control/user-account-control/ 

- **5.** Open Netfx35

Ref: https://learn.microsoft.com/en-us/dotnet/framework/install/dotnet-35-windows 

If you want to install through one-line command

Ref: https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/deploy-net-framework-35-by-using-deployment-image-servicing-and-management--dism?view=windows-11


- **6.** About Privilege

In Windows operating systems, the user account created during the initial setup is typically granted administrative privileges, but it is not the built-in Administrator account. The built-in Administrator is a special account with elevated permissions and fewer restrictions, which is disabled by default for security reasons.

Here is an example method given by ChatGPT

```bat
@echo off
:: Check if the script is running with administrator privileges
net session >nul 2>&1
if %errorlevel% neq 0 (
    echo Requesting administrative privileges...
    
    :: Create the embedded VBScript for elevation
    echo Set UAC = CreateObject^("Shell.Application"^) > "%temp%\getadmin.vbs"
    echo UAC.ShellExecute "%~f0", "", "", "runas", 1 >> "%temp%\getadmin.vbs"
    
    :: Run the VBScript to elevate permissions
    cscript "%temp%\getadmin.vbs"
    
    :: Exit the current script
    exit /b
)
```

## 7. Testing in Sandbox and Analyzing and Coding

### 7.1 Mentions above all

- **1.** Create a new virtual machine in VMware. 
- **2.** Install Windows 10 using English username. 
- **3.** Complete all required tasks up to Section 6, then take a snapshot by going to VM > Snapshot > Take Snapshot. 
- **4.** Name it appropriately (e.g., “Post Section 7 Setup”) for easy restoration later.

### 7.2 Coding with Analyzing

For simplicity in the following descriptions, all installation files will be referred to as `setup.exe` or `setup.msi`. If the installation files are contained within a folder, the folder will be referred to as `\setupfolder`.

#### 1. Analyzing Packages and Installation Process, 

- Some software vendors provide specific switch parameters for their installers, such as ChimeraX, Anaconda, and others.

-  When you double-click your setup.exe, and you notice a packaging software name (e.g., InstallShield) appearing in the bottom-left corner of the Windows window, this indicates the installer is running through that software.

- Switch parameter analysis is detailed in Section 3 of the official documentation. If you're unsure about specific parameters, you can display the help information for `setup.exe` by running it with the appropriate help flag (e.g., `/help` or `/?`) to get further details on available options.

- InstallShield packages often rely on `.msi` files, which can be identified through the installation progress bar. These files are usually extracted to the `%temp%` directory during installation. To preserve the `.msi` file, it must be copied before the installation completes, as it will be deleted afterward. Copy those `.msi` files, then run `msiexec.exe` to install silently.

#### 2. Ask "help" 

Based on packages type, Please prompt help command. Such references are given in Section 3.

#### 3. Key or not Key?

Determining whether a serial key is required depends on the following scenarios:

##### 1. Parameters for Transmitting Passwords or Serial Numbers
  - These are the parameters used to transmit passwords or serial numbers.

##### 2. Registry Entries for Validation
  - These registry entries are used to validate the software. You can use **Beyond Compare** to analyze registry changes. Capture a full export of the registry *both before and after* the installation to identify modifications. Some of these changes can help infer parameters related to software settings.

##### 3. Key as a File (Similar to a Standalone ZIP File)
  - The key is just a file, similar to a standalone ZIP file, and can be copied directly.

### 7.3 Coding Experiences

Herein, I would like to give some example with right or wrong codes for beginners.

#### 1. spacing and quote

- **When working with scripts or command-line operations, it’s crucial to understand how spaces, quotation, slash marks are handled in folder paths to ensure accurate parsing.**

Wrong
```
copy /y setupfolder C:Program Files 
```

Right 

```
xcopy /y /e “setupfolder“ ”C:Program Files\" 
```

#### 2. Path with Environment Variable

When working with file paths in scripts or command lines, environment variables provide a flexible way to reference system or user-specific directories. For example, `%USERPROFILE%` represents the current user's profile directory, and `%SYSTEMROOT%` points to the Windows installation folder. These variables help ensure compatibility across different systems and user configurations.

For more details on environment variables and how to use them in Windows, refer to the official [Microsoft documentation on Environment Variables](https://learn.microsoft.com/en-us/windows/win32/procthread/environment-variables).

#### 3. Powershell or Windows Batch?

Indeed, both PowerShell and Batch scripts can be used for silent installations. However, for consistency and ease of management, Batch scripts are often preferred when they suffice. If specific functionalities (e.g., uninstalling Microsoft Store apps) require PowerShell, the corresponding `.ps1` script can be executed using PowerShell.


```powershell
powershell -ExecutionPolicy Bypass -File "C:\Path\To\Script.ps1"
```

#### 4.Coding Example for simplification strategy

Ensure you have a thorough understanding of the previous sections. The following examples are provided solely to aid in comprehension.

  Install a standalone package, just copy it,including crack files copy to right path

```bat
xcopy /y /e "setupfolder" "C:\Program Files\setupfolder\"
xcopy /y /e "crack" "C:\Program Files\setupfolder\"
REM replacing running exe named a.exe
copy /y "a.exe"  ”C:Program Files\setupfolder\" 
```

  Install an msi with quite mode 
```bat
msiexec /i "setup.msi" /qb 
```

  Install package and add registry for crack 

```bat
REM Install the MSI package silently
msiexec /i "C:\Path\To\Installer.msi" /qn"

REM Add a registry key for demonstration purposes-example
reg add "HKLM\SOFTWARE\MySoftware" /v "LicenseKey" /t REG_SZ /d "1234-5678-ABCD" /f

```
  Install EXE with parameters

```bat
start /wait "" "setup.exe" /silent
```

#### 5. Assisting with ChatGPT

That is answer from ChatGPT

Advantage:

- **Automation and Scalability**: Capable of generating scripts for various installation scenarios, reducing manual effort.
- **Cross-Platform Compatibility**: Provides solutions for both Batch and PowerShell environments.
- **Customization**: Quickly adapts scripts to different software requirements, including silent installs and post-install configurations.
- **Error Reduction**: Helps identify common syntax and parameter issues in scripts.

Challenges:
- **Environment-Specific Testing**: Scripts may need adjustments based on user environments.
- **Dependency on User Input**: For highly customized setups, additional context or testing may be required.
- **Potential Overhead**: Complex installations may need detailed debugging or post-execution validation.

Recommendation:
- **Understand the Basics**: **Know the logic and structure** of what you’re coding before using AI for assistance.
- **Use GPT for Refinement**: Once you have a clear idea, GPT can help optimize, debug, or extend your scripts.
- **Attention to Detail for Beginners**: New users should pay **close attention to syntax, command arguments, and context to avoid mistakes**.

### 7.3 Testing in Sandbox

1. Make a proper snapshot for test

Before proceeding with the installation, ensure that all prerequisites are met, including system requirements, necessary permissions, and dependent software installations. Once the environment is properly configured, create a system snapshot. This allows you to restore the environment to a known good state if issues arise during testing or installation. Taking a snapshot is crucial for minimizing risks and maintaining consistency in your test setup.  

2. Remember coding and testing debug again

Whether you're working with well-established installation scripts or following a step-by-step manual process, thorough testing is essential. For every step of the installation, refer back to the analysis outlined in section 7.2, reviewing each point in detail. Debugging is an iterative process; it may require multiple rounds of testing and adjustments to ensure the installation runs smoothly and reliably. Consistent testing and refinement will help you identify and resolve any potential issues early on.


## 9. Encapsuling within an Image

### 9.1 Project File Tree

Here is my project file tree as an example:

- **Class1_Basic_Software**  
- **Class2_Optional_Software**  
- **Class3_LabSoftwareInstallation**  
- **Class4_ForAdvancedUserOnly**  
- **Class5_ForProgrammersOnly**  


### 9.2 ISO building

To access subdirectories and execute files in a single line of code within a .bat script, you can use a combination of commands that navigate and run the required file without having to change directories explicitly. 

Here’s a breakdown of what %~dp0 means:
	•	%0 refers to the batch file name.
	•	d extracts the drive letter of the script.
	•	p extracts the path without the file name.

For example, if you have a script located in C:\Scripts\install.bat, %~dp0 will return C:\Scripts\. This makes it easier to reference other files in the same directory or relative subdirectories without hardcoding paths.

**Please learn the command first!!!**

https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands

here is an example

```bat
pushd "%~dp0"
cd /d "%~dp0"
msiexec /i "%~dp0Class1_BasicSoftware\setup.msi" /qb
start /wait "%~dp0Class1_BasicSoftware\setup.exe" /silent
pause>nul
```

## 10. Distribution

### License Required 

Please be mindful of copyright protection and the importance of following licensing policies. It is essential to respect the intellectual property rights of others and to properly attribute any content used. When sharing or contributing to a project, always ensure that the correct licensing is applied, and that you comply with the terms outlined. This helps create a collaborative and legally sound environment for all contributors, and protects both the creators and users of the software.

Here is an example.

>MIT License
>
>Copyright (c) [year] [your name or organization]
>
>Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:
>
>The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.
>Any modified versions of the Software must acknowledge the original source.
>
>THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


---

Thank you for reading this article! If you have any suggestions or questions, feel free to contact me.
