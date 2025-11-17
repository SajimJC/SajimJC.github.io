---
title: "Silent Installation Progress 202511"
date: "2025-11-17"  
permalink: "/posts/2025/11/silent_install_guide_windows_progress-202511/"  
tags:
  - progress
  - silent_installation
---
Silent Installation Progress 202511 

## Progress

**Because of highest pressure on Ph D project. The progress is slow and delayed.*

In some cases, I have attempted to automatically download the installer to the %TEMP% directory from internet and perform the installation using an autosetup script. I plan to extend the code in future versions to improve this process. Some programs lack proper firewall configurations, which should be addressed during setup.

The previous ISO image of Class 1/2/3 has been packaged. It is now updated.

Some optional or open-source software for replacing commercial software are also packed, but not listed in this post.

## Problem

Some commercial software packaged with **InstallShield** can be challenging to deploy silently using conventional methods. These methods include searching for silent installation switches, recording a setup.iss file, extracting the EXE package, or monitoring the `%temp%` directory for installation files. In such cases, alternative approaches may need to be explored for successful silent deployment.

## Overview(Updated)

The progress details are outlined below:

1. Done: Already completed. **Done(Online)**: Code will be exececuted as *downloading and installing for web automatically*, used in **Installation Column**. 

2. Semi-automation: Silent installation with some required **few manual steps**. The major process is undergoing automation.

3. N/A: Not applicable to this process.

4. Partial: Additional settings are required for enhancement **for further stage**.

5. No: Configuration settings must be completed **by human operations** to enable functionality.

6. Default: Configuration settings are adhered to the software **default parameters**.

7. Code Ongoing: The silent installation **code editing** is in progress.

8. Test Ongoing: The silent installation **code testing** is in progress.

9. Note As Microsoft Store or msstore, meaning as download via MSStore. Note as UWP, meaning the app is an UWP application on Microsoft Store only.


## Class1_Basic_Software


|   Software Name         |   Installation   |   Cracking   | Configuration |
|:------------------------:|:----------------:|:------------:|:-------------:|
|  7z                     |       Done       |      N/A     |      Done     |
|  UltraISO               |       Done       |     Done     |      N/A      |
|  Fonts                  |       Done       |      N/A     |      Done     |
|  QQ Pinyin              |       Done       |      N/A     |      Done     |
|  VLC                    |       Done       |      N/A     |      Done     |
|  Potplayer              |       Done       |      N/A     |      Done     |
|  TTplayer Lite          |       Done       |      N/A     |      Done     |
|  Motrix                 |       Done       |      N/A     |    Partial    |
|  Free Download Manager  |       Done       |      N/A     |      N/A      |
|  CCleaner               |       Done       |      N/A     |      Done     |
|  CAJViewer              |       Done       |      N/A     |    Partial    |
|  PDF-XChange Editor Plus|       Done       |     Done     |      Done     |
|  Acrobat                |       Done       |     Done     |    Partial    |
|  FoxitReaderStandalone  |       Done       |     N/A      |      N/A      |
|  Chrome Enterprise      | **Done(Online)** |     N/A      | Code Ongoing  |
|  Edge   Enterprise      |       Done       |     N/A      | Test Ongoing  |
|  WeChat Microsoft Store | **Done(Online)** |     N/A      | Test Ongoing  |
|  QQ Microsoft Store     | **Done(Online)** |     N/A      | Test Ongoing  |
|Tencent Meeting(msstore) | **Done(Online)** |     N/A      | Test Ongoing  |

*Many software applications are under consideration for inclusion in the silent installation library classes as appropriate.*


## Class2_Optional_Software

|      Software Name      |   Installation   |   Cracking   | Configuration |
|:-----------------------:|:----------------:|:------------:|:-------------:|
|  Adobe Illustrator      |       Done       |      N/A     |      Done     |
|  Adobe Photoshop        |       Done       |      N/A     |      Done     |
|  Only Office            |   Code Ongoing   |      N/A     |  Code Ongoing |  
|  LibreOffice            |   Code Ongoing   |      N/A     |  Code Ongoing |
|  Inkscape               |       Done       |      N/A     |      Done     |


## Class3_LabSoftwareInstallation


|         Software Name         |   Installation    |   Cracking    | Configuration |
|:-----------------------------:|:-----------------:|:-------------:|:-------------:|
|         ChemOffice            |        Done       |      Done     |    Partial    |
|         ChemCraft             |        Done       |      Done     |      N/A      |
|         GraphPad              |        Done       |      Done     |      Done     |
|         OriginLab             |        Done       |      Done     |    Partial    |
|         HITACHI-FL Solutions  |        Done       |      Done     |      Done     |
|         MestReNova            |        Done       |      Done     |    Partial    |
|         Fiji                  |        Done       |      N/A      |      N/A      |
|         Agilent MassHunter    |        Done       |      N/A      |    Partial    |
|         MS-DIAL               |        Done       |      N/A      |      N/A      |
|         Mzmine                |        Done       |      N/A      |      N/A      |
|         MSFINDER              |        Done       |      N/A      |      N/A      |
|         Discovery Studio      |        Done       |      N/A      |    Partial    |
|         MOE2019               |  Semi-automation  |      Done     |      Done     |
|         MOE2022               |        Done       |      Done     |      Done     |
|         Schrodinger           |        Done       |      Done     |      Done     |
|         Chimera               |        Done       |      N/A      |      Done     |
|         Pymol-Open Source     |        Done       |      N/A      |      Done     |
|         Pymol-Schrodinger     |        Done       |      **No**   |     **No**    |
|         Scripts Autodock      |        Done       |      N/A      |      Done     |
|         icmbrowser            |        Done       |      N/A      |    Partial    |
|         datawarrior           |        Done       |      N/A      |    Partial    |
|         Zotero                |        Done       |      N/A      |    Partial    |
|         R Studio              |        Done       |      N/A      |    Partial    |
|         Python                | **Done(Online)**  |      N/A      | Code Ongoing  |
|         Anaconda              |        Done       |      N/A      |      Done     |
|         Digital Micrograph    |        Done       |      Done     |      Done     |
|         Jade                  |        Done       |      Done     |      Done     |
|         Diamond               |        Done       |      N/A      |      Done     |
|         Ortep3                |        Done       |      N/A      |      Done     |
|         DrawIo                |        Done       |      N/A      |    Default    |
|         SnapGene              |        Done       |      Done     |      Done     |
|         PerseeUV              |        Done       |      N/A      |    Partial    |
|         Shimadzu UV Solution  |       **No**      |      N/A      |     **No**    |
|         Omnic                 |  Semi-automation  |      N/A      |     **No**    |
|         Xcalibur              |       **No**      |      N/A      |     **No**    |
|         Zhiyun                |        Done       |      N/A      |    Partial    |
|         MassLynx              |       **No**      |      N/A      |     **No**    |
|         NIS Element Viewer    |       **No**      |      N/A      |     **No**    |
|         Gaussian/GaussView    |        Done       |      Done     |    Default    |
|         Mathtype              |       **No**      |      N/A      |     **No**    |
|         FlowJo                |       **No**      |     **No**    |     **No**    |
|         MatLab                |       **No**      |     **No**    |     **No**    |
|         Multiwfn              |   Code Ongoing    |      N/A      |     Default   |
|         Shermo                |   Code Ongoing    |      N/A      |     Default   |
|         SPSS                  |   Code Ongoing    |      N/A      |     Default   | 
|         VMD                   |   Code Ongoing    |      N/A      |     Default   |


*Many software applications are under consideration for inclusion in the silent installation library classes as appropriate.*

## Class4_ForAdvancedUserOnly


|   Software Name   |   Installation   |   Cracking   | Configuration |
|:-----------------:|:----------------:|:------------:|:-------------:|
|       AIDA        |       Done       |     Done     |      N/A      |
|    Everything     |       Done       |      N/A     |    Partial    |
|    MobaXterm      |       Done       |      N/A     |    Partial    |
|      Winscp       |       Done       |      N/A     |    Partial    |
|    BulkRename     |       Done       |      N/A     |    Partial    |
|    EV Recorder    |       Done       |      N/A     |    Partial    |
| Sysinternal Series|       Done       |      N/A     |      N/A      |
| VMware Workstation|       Done       |      N/A     |    Partial    |
|      VSCode       | **Done(Online)** |      N/A     | Code Ongoing  |
| Adobe After Effect|       Done       |      N/A     |      Done     |
|  Adobe Animate    |       Done       |      N/A     |      Done     |
|  Adobe Audition   |       Done       |      N/A     |      Done     |
|Adobe Premiere Pro |       Done       |      N/A     |      Done     |
|      FFmpeg       |       Done       |      N/A     |      Done     |
|     Goldwave      |       Done       |     Done     |    Partial    |
|  RealVNC Server   |       Done       |      N/A     |      Done     |
|  RealVNC Viewer   |       Done       |      N/A     |      Done     |
|    TeamViewer     |       Done       |      N/A     |    Partial    |


## Class5_ForProgrammerOnly


|   Software Name   |   Installation   |   Cracking   | Configuration |
|:-----------------:|:----------------:|:------------:|:-------------:|
|       AIDA        |       Done       |     Done     |      N/A      |
|  Beyond Compare   |       Done       |     N/A      |      N/A      |
|      dbeaver      |       Done       |     N/A      |      N/A      |
|        Git        | **Done(Online)** |     N/A      |      N/A      |
|       Java        |       Done       |     N/A      |      N/A      |
|     Fiddler       |       Done       |     N/A      |      N/A      |
|    Wireshark      |       Done       |     N/A      |      N/A      |
|       IDA         |       Done       |     N/A      |      N/A      |
|      Pycharm      |       Done       |     N/A      |      N/A      |
|  RegistryWorkshop |       Done       |     N/A      |      N/A      |
|      ThinApp      |       Done       |     N/A      |      N/A      |
|       Tuba        |       Done       |     N/A      |      N/A      |    
|       Trae        |       Done       |     N/A      |      N/A      |  


## Other Silent Installation Software Coding--HARD and CHALLENGE 

Other software for silent installation is currently being tested. 

Configuration is just for mentioning. Not ready to write code script. So the configuration set to be N/A.


|       Software Name     |   Installation   |   Cracking   | Configuration |
|:-----------------------:|:----------------:|:------------:|:-------------:|
|  MS Office 2007         |  Code Ongoing    | N/A(KeyLeak) | Code Ongoing  |
|  WPS Education          | **Done(Online)** |     N/A      | Code Ongoing  |
|  WPS  (msstore)         | **Done(Online)** |     N/A      | Code Ongoing  | 
|  QQ Music (msstore)     | **Done(Online)** |     N/A      |      N/A      | 
|  KuGou Music (msstore)  | **Done(Online)** |     N/A      |      N/A      | 
|  Kuwo Music (msstore)   | **Done(Online)** |     N/A      |      N/A      | 
|  Sysdiag  (msstore)     | **Done(Online)** |     N/A      |      N/A      |
|  Bibilibi (msstore)     | **Done(Online)** |     N/A      |      N/A      |  
|  Douyin   (msstore)     | **Done(Online)** |     N/A      |      N/A      |
|  Jianying (msstore)     | **Done(Online)** |     N/A      |      N/A      |
|  Capcut (msstore)       | **Done(Online)** |     N/A      |      N/A      |

--