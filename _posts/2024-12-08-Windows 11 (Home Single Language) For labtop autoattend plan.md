---
title: "Windows 11 Home oobe unattended plan"
date: "2024-12-08"  
permalink: "/posts/2024/12/windows11_home_oobe_unattended/"  
tags:
  - oobe_unattended
  - windows 11 home
---

**Plan of the Unattended OOBE Script for Windows 11 Home**

## Objective

The unattended OOBE script automates the initial setup of **Windows 11 Home (Single Language edition), pre-installed on laptops**, to simplify the configuration process and enhance efficiency during the first boot. This is particularly useful for system administrators or users aiming to:

	•	Predefine system settings.
 
	•	Configure user accounts.
 
	•	Optimize setup processes for a consistent experience.

## Key Features of the Script

1.	Language and Region Configuration:

	•	Automatically sets up the system language, region, and input method based on predefined values.

	•	Example: For laptops in China, it sets zh-CN as the language and region.

3.	Network and Account Bypass:

	•	**Skips unnecessary OOBE screens such as Microsoft account login** （Support Home Single Language Version）

	•	Enables local account creation directly.

4.	User Account Setup:

	•	Automatically creates a local administrator account with pre-configured credentials.

	•	Optionally enables auto-login for the first boot.

5.	System Customizations:

	•	Applies registry tweaks for bypassing hardware checks (e.g., TPM, Secure Boot).

	•	Removes pre-installed bloatware using PowerShell scripts.

	•	Configures system preferences like privacy settings and feature optimizations.

6.	First Logon Commands:

	•	Executes custom scripts (e.g., software installation, environment configurations) during the first logon.

## Plan 

1.  Code Generator 

    __Done__

    [Windows Unattend Generator](https://schneegans.de/windows/unattend-generator/)

2.  Analyze and revise for proper use 

    __Done__

    •	Remove Key: For laptops, please be aware of the OEM license terms. If the laptop has an embedded serial number, it does not need to be manually entered.

    •	Disk Configuration: Not applicable for pre-installed OEM machines, which already come with partitioning and a licensed operating system, such as those from brands.

    •   Audit process creation events: Each time a new process is created, Windows writes an event to the Security log. This is a powerful tool for troubleshooting.

    •	Other configuration settings for the autounattend.xml: Please ask ChatGPT for help.

3. Prepare the USB Drive 

    __Done__

	•	Format the USB Drive: Ensure the USB drive is formatted as FAT32.

	•	Copy the autounattend.xml file in the root directory of the USB drive.

	•	Optionally, include any other scripts or files needed for customization.

	•	No additional steps are required for creating a bootable USB drive.


4. Installation Process:

    •	Before preparing to install Windows 11, please insert the USB drive into the laptop *for first start-up*. 

    •	During the Windows 11 Home installation, the system will automatically recognize the file and use it for configuration.

5. Observe and Debug

	•	Run automatically without error.

	•	Configures language, region, and system settings.

	•	Sets up a local administrator account.

	•	Bypasses unnecessary OOBE (Out-of-Box Experience) screens.

	•	The process completes, and the laptop boots directly into the desktop environment.

6. Post-Installation Customization (First Logon Commands):

	•	The FirstLogonCommands section allows for running additional commands or scripts during the first login.

	•	Install software with my project [silent installation for lab](https://sajimjc.github.io/posts/2024/11/silent_install/).

	•	Configure system settings. Especially, improper uses of Windows 11.

	•	Remove unwanted bloatware.

## Attention

Section 4, 5, and 6 Need Iteration

Completion Time to be Determined

This project will not include the full code, but only the detailed requirements. 

The removal of certain components is based on individual needs and can be adjusted accordingly. 

If you have any questions or require further information, please feel free to contact me.

---

