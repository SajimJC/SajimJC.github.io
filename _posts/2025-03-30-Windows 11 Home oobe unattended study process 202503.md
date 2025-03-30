---
title: "Windows 11 Home oobe unattended study process"
date: "2025-03-30"  
permalink: "/posts/2025/03/windows11_home_oobe_unattended/"  
tags:
  - oobe_unattended
  - windows 11 home
---

**Windows 11 Home OOBE Unattended Study Process**

## 0. Background:

The original idea, article link: 

![See This](https://sajimjc.github.io/posts/2024/12/windows11_home_oobe_unattended/)


## 1. The First Problem to Solve:

### 1.1 Key Issue:
Why does the Out-of-Box Experience (OOBE) process install many enterprise promotions and pre-installed applications after a new OEM machine is unboxed? What is the mechanism behind this?

### 1.2 Mechanism to Understand

#### Reference Document 1: Learning the Configuration Passes Workflow
https://learn.microsoft.com/zh-cn/windows-hardware/manufacture/desktop/how-configuration-passes-work?view=windows-11

![Configuration Passes](https://learn.microsoft.com/zh-cn/windows-hardware/manufacture/desktop/images/dep-win8-l-configpassesandexes.jpg?view=windows-11)

From this image, we can draw the following conclusions:

1. Order Requirement: The execution order of settings pass="..." is fixed and must follow the sequence: windowsPE -> offlineServicing -> generalize -> specialize -> auditSystem -> auditUser -> oobeSystem.
2. For pre-installing Windows Home Edition systems, the serial number, necessary drivers, and partitioning are completed during the windowsPE phase. Pre-provisioning packages (PPKGs) may be applied during the specialize -> auditSystem -> auditUser phases. However, different manufacturers have varying strategies, and without access to their images, it's difficult to reverse-test or determine these strategies.

## 2. The Second Problem to Solve:

How can Unattended Mode be used to meet system pre-installation requirements?
The requirement is: **There exists a USB drive containing an Autounattend.xml file that ensures the installation of pre-configured critical system drivers while removing unnecessary pre-installed software.**

The current challenge is understanding how manufacturers like Lenovo, HP, Dell, Honor, and Huawei achieve the following:
1. Pre-installation of system drivers and accessories (e.g., Realtek, NVIDIA) - These must be retained.
2. Pre-installation of software - This must be prohibited.
3. Enabling my custom unattended rules by inserting a USB drive before startup and ensuring subsequent rules follow my unattended configuration.

## 3. The Third Problem to Solve:

How can testing be performed? How can the environment be simulated in a virtual machine?
On real machines, how can safety be ensured?
Ensure that even if the custom unattended setup fails, it will not destroy the pre-installed drivers of the original system.
Given that systems perform PPKG actions early during startup, how can this behavior be simulated in a virtual machine test process for different OEM systems?

## 4. Learning Insights:

### 2025.03.30
Understanding the structure and patterns of unattended.xml, the generic framework is as follows:

```xml
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State">
    <!-- Comment Section -->
    
    <!-- Offline Servicing Configuration -->
    <settings pass="offlineServicing">
    </settings>
    
    <!-- Windows PE Configuration -->
    <settings pass="windowsPE">
        <component name="Microsoft-Windows-International-Core-WinPE" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
            <SetupUILanguage>
                <UILanguage>en-US</UILanguage>
            </SetupUILanguage>
            <InputLocale>0409:{81d4e9c9-1d3b-41bc-9e6c-4b40bf79e35e}{fa550b04-5ad7-411f-a5ac-ca038ec515d7}</InputLocale>
            <SystemLocale>en-US</SystemLocale>
            <UILanguage>en-US</UILanguage>
            <UserLocale>en-US</UserLocale>
        </component>
        <component name="Microsoft-Windows-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
            <UserData>
                <AcceptEula>true</AcceptEula>
            </UserData>
            <RunSynchronous>
                <RunSynchronousCommand wcm:action="add">
                    <Order>1</Order>
                    <Path>reg.exe add "HKLM\SYSTEM\Setup\LabConfig" /v BypassTPMCheck /t REG_DWORD /d 1 /f</Path>
                </RunSynchronousCommand>
                <RunSynchronousCommand wcm:action="add">
                    <Order>2</Order>
                    <Path>reg.exe add "HKLM\SYSTEM\Setup\LabConfig" /v BypassSecureBootCheck /t REG_DWORD /d 1 /f</Path>
                </RunSynchronousCommand>
                <RunSynchronousCommand wcm:action="add">
                    <Order>3</Order>
                    <Path>reg.exe add "HKLM\SYSTEM\Setup\LabConfig" /v BypassRAMCheck /t REG_DWORD /d 1 /f</Path>
                </RunSynchronousCommand>
            </RunSynchronous>
        </component>
    </settings>

    <!-- Generalization Configuration -->
    <settings pass="specialize">
    </settings>

    <!-- Specialization Configuration -->
    <settings pass="specialize">
        <component name="Microsoft-Windows-Deployment" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
            <RunSynchronous>
                <RunSynchronousCommand wcm:action="add">
                    <Order>1</Order>
                    <Path>powershell.exe -NoProfile -Command "$xml = [xml]::new(); $xml.Load('C:\Windows\Panther\unattend.xml'); $sb = [scriptblock]::Create( $xml.unattend.Extensions.ExtractScript ); Invoke-Command -ScriptBlock $sb -ArgumentList $xml;"</Path>
                </RunSynchronousCommand>
                <RunSynchronousCommand wcm:action="add">
                    <Order>2</Order>
                    <Path>powershell.exe -NoProfile -Command "Get-Content -LiteralPath 'C:\Windows\Setup\Scripts\Specialize.ps1' -Raw | Invoke-Expression;"</Path>
                </RunSynchronousCommand>
                <RunSynchronousCommand wcm:action="add">
                    <Order>3</Order>
                    <Path>reg.exe load "HKU\DefaultUser" "C:\Users\Default\NTUSER.DAT"</Path>
                </RunSynchronousCommand>
                <RunSynchronousCommand wcm:action="add">
                    <Order>4</Order>
                    <Path>powershell.exe -NoProfile -Command "Get-Content -LiteralPath 'C:\Windows\Setup\Scripts\DefaultUser.ps1' -Raw | Invoke-Expression;"</Path>
                </RunSynchronousCommand>
                <RunSynchronousCommand wcm:action="add">
                    <Order>5</Order>
                    <Path>reg.exe unload "HKU\DefaultUser"</Path>
                </RunSynchronousCommand>
            </RunSynchronous>
        </component>
    </settings>

    <!-- Audit System Configuration -->
    <settings pass="auditSystem">
        <component name="Microsoft-Windows-Deployment" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
            <RunSynchronous>
                <RunSynchronousCommand wcm:action="add">
                    <Order>1</Order>
                    <Path>reg.exe add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Provisioning\State" /v SkipProvisioning /t REG_DWORD /d 1 /f</Path>
                </RunSynchronousCommand>
            </RunSynchronous>
        </component>
    </settings>

    <!-- Audit User Configuration -->
    <settings pass="auditUser"></settings>

    <!-- OOBE Configuration -->
    <settings pass="oobeSystem">
        <component name="Microsoft-Windows-International-Core" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
            <InputLocale>0409:{81d4e9c9-1d3b-41bc-9e6c-4b40bf79e35e}{fa550b04-5ad7-411f-a5ac-ca038ec515d7}</InputLocale>
            <SystemLocale>en-US</SystemLocale>
            <UILanguage>en-US</UILanguage>
            <UserLocale>en-US</UserLocale>
        </component>
        <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" versionScope="nonSxS">
            <UserAccounts>
                <AdministratorPassword>
                    <Value></Value>
                    <PlainText>true</PlainText>
                </AdministratorPassword>
            </UserAccounts>
            <AutoLogon>
                <Username>Administrator</Username>
                <Enabled>true</Enabled>
                <LogonCount>1</LogonCount>
                <Password>
                    <Value></Value>
                    <PlainText>true</PlainText>
                </Password>
            </AutoLogon>
            <OOBE>
                <ProtectYourPC>1</ProtectYourPC>
                <HideEULAPage>true</HideEULAPage>
                <HideWirelessSetupInOOBE>false</HideWirelessSetupInOOBE>
                <HideOnlineAccountScreens>false</HideOnlineAccountScreens>
            </OOBE>
            <FirstLogonCommands>
                <SynchronousCommand wcm:action="add">
                    <Order>1</Order>
                    <CommandLine>powershell.exe -NoProfile -Command "Get-Content -LiteralPath 'C:\Windows\Setup\Scripts\FirstLogon.ps1' -Raw | Invoke-Expression;"</CommandLine>
                </SynchronousCommand>
            </FirstLogonCommands>
        </component>
    </settings>

    <!-- Parsing and Extracting <Extensions> Node Content -->
    <Extensions xmlns="https://schneegans.de/windows/unattend-generator/">
        <ExtractScript>
param(
    [xml] $Document
);

foreach( $file in $Document.unattend.Extensions.File ) {
    $path = [System.Environment]::ExpandEnvironmentVariables(
        $file.GetAttribute( 'path' )
    );
    mkdir -Path( $path | Split-Path -Parent ) -ErrorAction 'SilentlyContinue';
    $content = $file.InnerText.Trim();
    if( $file.GetAttribute( 'transformation' ) -ieq 'Base64' ) {
        [System.IO.File]::WriteAllBytes( $path, [System.Convert]::FromBase64String( $content ) );
    } else {
        $encoding = switch( [System.IO.Path]::GetExtension( $path ) ) {
            { $_ -in '.ps1', '.xml' } { [System.Text.Encoding]::UTF8; }
            { $_ -in '.reg', '.vbs', '.js' } { [System.Text.UnicodeEncoding]::new( $false, $true ); }
            default { [System.Text.Encoding]::Default; }
        };
        [System.IO.File]::WriteAllBytes( $path, ( $encoding.GetPreamble() + $encoding.GetBytes( $content ) ) );
    }
}
        </ExtractScript>
        <File path="C:\Windows\Setup\Scripts\TaskbarIcons.ps1" transformation="Text">
Remove-ItemProperty -Path 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Explorer\Taskband' -Name '*';
        </File>
    </Extensions>
</unattend>

```

## 5. Progress and Testing Insights

The unattended.xml file is designed to simulate a Windows 11 installation process, including the installation of drivers, user accounts, and other settings. The script is designed to be executed in a virtual machine environment, and the results are simulated.

Test results using the previously written unattended.xml in a virtual machine: **Semi-automatic conditions can be passed.**

The script lacks **the following content**, so the simulation effect still has some defects:

• Does not simulate the built-in driver environment of pre-installed systems.

• The system disk on virtual machine is empty and requires manual partitioning.

• It is challenging to simulate the pre-installation process through the unattended script's specific environment.

**Studying PPKG or testing other commercial recovery systems** for further research.

