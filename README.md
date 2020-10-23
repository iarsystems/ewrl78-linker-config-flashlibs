# ICF Trio

## Table of Contents
- [What is the purpose of the ICF Trio Project?](#what-is-the-purpose-of-the-icf-trio-project)
- [Benefits](#benefits)
- [Linker Configuration Trio Layout Specification](#linker-configuration-trio-layout-specification)
- [Flash Library Flavors](#flash-library-flavors)
- [Flash Library Types](#flash-library-types)
- [Self-RAM](#self-ram)
- [How to use the ICF Trio](#how-to-use-the-icf-trio)
- [Required Software](#required-software)
- [RL78 Flash Libraries, documentation and Symbols](#rl78-flash-libraries-documentation-and-their-respective-required-linker-symbols-for-self-ram)
- [Usage Guidelines](#usage-guidelines)
    - [Install the tools](#install-the-tools)
    - [Applilet3 base project creation](#applilet3-base-project-creation)
- [Coding Examples](#coding-examples)


## What is the purpose of the ICF Trio Project?

The __ICF Trio__ project was developed for __IAR Embedded Workbench for Renesas RL78__ version __3.10 or later__. It uses a set of 3 files which together will lead to a proper linker configuration for the memory reservation requirements when there are __RL78 Flash Libraries__ in use.

> __Notes__
> * Although it is highly recommended staying with the latest release of the IDE when starting new projects to take advantage of bug fixes and improved experience, we kept a reference for a simple [tweak](tweak.md) for our long standing EWRL78 v2.x users.
> * The __ICF Trio__ cannot be used with the EWRL78 v1.x series.

## Benefits 
* Provides simplified setup for new projects relying on any of the __RL78 Flash Libraries__.
* Reduces the developer efforts when linker reconfiguration is needed, specially when facing target migration in regards to the __RL78 Flash Libraries__.
* Offers contiguous __Code Flash__ which leads to improved flexibility when compiled objects placement is performed at the linking stage. This is achieved by taking advantage of the linker capability of placing all the **__near** constants from the __ending__ of the mirrorable area in the __Code Flash__ whereas the default linker configuration would start filling the **__near** constants section from the beginning of the __Mirrorable Area__ (normally starting from the address _**0x2000**_ or _**0x3000**_ depending on the target in use).
* Reduces the __Code Flash__ fragmentation effect which may be noticeable on targets with smaller flash memory installed, also thanks to its better strategy for object placement.

## Linker Configuration Trio Layout Specification

In this section you can understand how the __ICF Trio__ components fit together.

![ICF Trio layout](/images/icf_trio_layout.png)

| __File__                     | __Description__ |
| :--------------------------- | --------------- |
| __[trio_lnkR5F1`nn`X`n`.icf](trio_lnkR5F1---TEMPLATE.icf)__ | The first one is __user-selectable__. The selection is made based on the similarity in the memory map for distinct groups of RL78 targets. Each of these files hold the proper `Linker Configuration Override` parameters which can be set on the `Project options`.<br><br>The `X` within the part number means that the linker configuration is offered regardless the target's pin count. For example, __trio_lnkR5F100xE.icf__ should be selected for a __R5F100LE__ target. |
| [common.icf](common.icf)     | The second is automatically included from the user-selected configuration. This is the heart of the Trio, containing parametrized directives which can be applied to any of the supported RL78 targets. |
| [self_ram.icf](self_ram.icf) | The third is automatically included by __common.icf__ to evaluate the same [symbol styles](README.md#rl78-flash-libraries-documentation-and-their-respective-required-linker-symbols-for-self-ram) used in the __IAR Embedded Workbench for RL78__ to handle every __Self-RAM__ reservation needs for the Trio. |


## Flash Library Flavors

The Renesas RL78 MCUs require specific set of library(/ies) to enable usage of their Flash Memories.

Renesas Electronics provides the __RL78 Flash Libraries__ in 3 different flavors:
- The **FSL** (Flash Self-Programming Library) does program the RL78's __Code Flash__. 
- The **FDL** (Flash Data Library) does program the RL78's __Data Flash__.
- The **EEL** (EEPROM Emulation Library) does emulate EEPROM behaviour on the RL78's __Data Flash__. It provides an extra layer of functionalities on top of its corresponding companion FDL Library Type (T01 or T02 - [explained below](README.md#flash-library-types)). It provides transparent [__Wear Leveling__](https://en.wikipedia.org/wiki/Wear_leveling) among the provisioned __Data Flash__ blocks which may, in practice, virtually raise the amount of possible rewrites.


## Flash Library Types 

The __RL78 Flash Libraries__ flavors may be provided as one of the following library types:
- The **T01** (Type01, also known as **Full**) are the fully fledged Flash Libraries.
- The **T02** (Type02, also known as **Tiny**) are the balanced ones, providing the main functionalities at expense of less resources when compared with the T01 Libraries.
- The **T04** (Type04, also known as **Pico**) is the one providing only the bare essential functionalities. This library type offers the lowest resource usage footprint. Usually this is the suitable choice for the scenarios where the chosen RL78 target comes with constrained memory resources.

> __Note__
> * For further information regarding the complete feature set provided in each of these flash libraries, refer to their respective [documentation](README.md#rl78-flash-libraries-documentation-and-their-respective-required-linker-symbols-for-self-ram).


## Self-RAM 

Typically for every combination of RL78 MCU and __RL78 Flash Library__, the programmer would need to refer to the *Renesas Electronics*'  **[Application Note 2944](https://www.renesas.com/doc/products/tool/doc/015/r20ut2944ej0302_rl78.pdf)** in order to know if the chosen combination will require some specific RAM range to be reserved, therefore the chosen combination can function properly.

Self-RAM refers to the aforementioned RAM area, which __must__ be reserved on some cases, when relying on the RL78 MCU's self-programming capabilities.

In order to tremendously simplify this process, the __ICF Trio__ mostly automates it, by taking advantage of every advanced linker configuration directive available to override the default linker configuration, while following the requirements defined in the aforementioned Application Note. 


## How to use the ICF Trio

The following sections are going to serve as a straightforward step-by-step guide containing what is considered to be near the minimal actions that should be taken from scratch to configure a project to use the __ICF Trio__.

As reference, the RL78/G14 MCU (PN# __[R5F104LEAFA][R5F104LEAFA-url]__) will be used alongside the most popular __RL78 Flash Libraries__ combinations. Even then, the following steps should be similarly applicable to other RL78 targets.

[R5F104LEAFA-url]: https://www.renesas.com/products/microcontrollers-microprocessors/rl78/rl78g1x/rl78g14/device/R5F104LEAFA.html

## Required Software 

The following software components were successfuly usable alongside the __ICF Trio__:
- [IAR Embedded Workbench for RL78 version 4.20](https://www.iar.com/iar-embedded-workbench/#!?architecture=RL78) 
- [Git for Windows](https://git-scm.com/download/win) 
- [Applilet3 for RL78](https://www.renesas.com/software/D4000916.html) 
- A __RL78 Flash Library__ of your choice, downloadable from the Renesas Electronics Europe "MyPages": 
   - The download requires [pre-registration](https://www2.renesas.eu/products/micro/download/index.html/auth/register)
   - or [sign-in](https://www2.renesas.eu/products/micro/download/index.html/auth/login) for registered users

### RL78 Flash Libraries, documentation and their respective required linker symbols for Self-RAM

| __RL78 Flash Library__ | __Documentation__          | __Symbol__            | __Description__                                                 |
| :--------------------: | :------------------------: | :-------------------: | :-------------------------------------------------------------- |
| [T01-FSL][t01-fsl-url] | [T01-FSL][t01-fsl-doc-url] | `__RESERVE_T01_FSL=1` | Reserves Self-RAM for the __T01-FSL__ Code Flash Library        |
| [T01-FDL][t0x-xxl-url] | [T01-FDL][t01-fdl-doc-url] | `__RESERVE_T01_FDL=1` | Reserves Self-RAM for the __T01-FDL__ Data Flash Library        |
| [T01-EEL][t0x-xxl-url] | [T01-EEL][t01-eel-doc-url] | `__RESERVE_T01_EEL=1` | Reserves Self-RAM for the __T01-EEL__ EEPROM Emulation Library  |
| [T02-FDL][t0x-xxl-url] | [T02-FDL][t02-fdl-doc-url] | `__RESERVE_T02_FDL=1` | Reserves Self-RAM for the __T02-FDL__ Tiny Data Flash Library   |
| [T02-EEL][t0x-xxl-url] | [T02-EEL][t02-eel-doc-url] | `__RESERVE_T02_EEL=1` | Reserves Self-RAM for the __T02-EEL__ EEPROM Emulation Library  |
| [T04-FDL][t0x-xxl-url] | [T04-FDL][t04-fdl-doc-url] | `__RESERVE_T04_FDL=1` | Reserves Self-RAM for the __T04-FDL__ Pico Flash Lbrary         |

[t01-fsl-url]:     https://www2.renesas.eu/products/micro/download/?oc=SelfLib_RL78
[t0x-xxl-url]:     https://www2.renesas.eu/products/micro/download/?oc=EEPROM_EMULATION_RL78
[t01-fsl-doc-url]: https://www.renesas.com/eu/en/doc/DocumentServer/011/R01US0016ED0105_RL78.pdf
[t01-fdl-doc-url]: https://www.renesas.com/eu/en/doc/DocumentServer/011/R01US0034ED0101.pdf
[t01-eel-doc-url]: https://www.renesas.com/eu/en/doc/DocumentServer/011/R01US0128ED0101_RL78.pdf
[t02-fdl-doc-url]: https://www.renesas.com/eu/en/doc/DocumentServer/011/R01US0061ED0120_RL78.pdf
[t02-eel-doc-url]: https://www.renesas.com/eu/en/doc/DocumentServer/011/R01US0070ED0105_RL78.pdf
[t04-fdl-doc-url]: https://www.renesas.com/eu/en/doc/DocumentServer/011/R01US0055ED0112.pdf


## Usage Guidelines 

### Install the tools  

**1.** Install the [IAR Embedded Workbench for RL78](https://www.iar.com/iar-embedded-workbench/#!?architecture=RL78).

> __Note__
> * When installing the IAR Embedded Workbench for RL78, the installer also will offer the possibility to install a code generator tool named [AP4](https://www.renesas.com/products/software-tools/tools/code-generator/ap4-applilet.html). The AP4 is a GUI-based code generator that can generate peripheral drivers in C source containing driver functions for some of the RL78 targets.

**2.** For the purposes of this general guide, the selected target will be a general purpose RL78/G14 (PN# __[R5F104LEAFA](https://www.renesas.com/products/microcontrollers-microprocessors/rl78/rl78g1x/rl78g14/device/R5F104LEAFA.html)__). The target belongs to the *RL78/G14 Series*. The G14 Series uses a previous version of the AP4, the [Applilet3 for RL78](https://www.renesas.com/software/D4000916.html) configuration tool, which should then be installed.

**3.** Install [Git for Windows](https://git-scm.com/download/win) on its defaults.

### Applilet3 base project creation

**4.** Start the __Applilet3__ tool, creating a new project which targets a __R5F104LEA__ (RL78/G14(ROM:64KB)) using the `IAR Compiler` build tool, then choose a `Project Name` and save it under the project's chosen `Place`. At this point, a new folder with the `Project Name` is going to be created at the chosen `Place`.

![New Project](/images/3newproject.png)

**5.** On `Pin assignment` tab, click on `Fix settings`.

![Pins Assignment](/images/pin_assignment.png)

**6.** On the `On-Chip debug setting` tab, enable the `On-chip debug operation setting`.

![OCD](/images/5ocd.png)

**7.** Enable the __Low Voltage Detection__, selecting any coherent value for `VLVD`, such as *3.63V*. Save the Project. Finally, click on `Generate code`.

![CG LVD](/images/cg-lvd.png)

### IAR Embedded Workbench, project connection and setup

**8.** Start the __IAR Embedded Workbench for RL78__, save the the Workspace (_.eww_) on the same project folder which was created at the chosen `Place`. This folder can (and will) be referred by __IAR Embedded Workbench__ through its built-in environment variable __$PROJ_DIR$__.

> __Note__
> * The __$PROJ_DIR$__ is an argument variable which can be conveniently used refer to relative pathnames when accessing the project's resources in the storage media. With __IAR Embedded Workbench__, you can use a wide range of built-in argument variables, as well as create your own. For more information search for __[Argument Variables][argvars-url]__ on the [__IAR Embedded Workbench IDE Project Management and Building Guide__][argvars-url].

[argvars-url]: https://netstorage.iar.com/SuppDB/Public/UPDINFO/014217/ew/doc/EWRL78_IDEGuide.ENU.pdf#page=82

**9.** Choose `Project` → `Create New Project...` and create an empty RL78 project. Save it on the project's __$PROJ_DIR$__ location.

**10.** Choose `Project` → `Add Project Connection...` and point to the __.ipcf__ which has been created when the __Applilet3__ code was generated.

![Workspace](/images/7workspace.png)

**11.** Invoke the context menu (right-click) on the __.ipcf__ file and choose `Open Containing Folder`. Invoke the context menu for the folder in the __Windows Explorer__ and choose `Git Bash Here`. This step will open a Git Bash Shell at `Place` (__$PROJ_DIR$__). From there perform:

 ```bash
 $ git clone https://github.com/IARSystems/ewrl78-linker-config-flashlibs.git
 ```
 >~~~
 > Cloning into 'ewrl78-linker-config-flashlibs'...
 > remote: Counting objects: 58, done.
 > remote: Compressing objects: 100% (33/33), done.
 > remote: Total 58 (delta 36), reused 34 (delta 30), pack-reused 0
 > Unpacking objects: 100% (58/58), done.
 >~~~
 
**12.** Back to the __IAR Embedded Workbench for RL78__, go to the Project Options, `Linker` → `Config` → `Override default` and select the appropriate __.icf__ for the target. In this example, the __R5F104LEA__ target will be used, so the [trio_lnkR5F104xE.icf](trio_lnkR5F104xE.icf) will be selected:
```
$PROJ_DIR$\ewrl78-linker-config-flashlibs\trio_lnkR5F104xE.icf
```

From this point you can now choose one of the examples below, which contains further steps for creating simple programs which exercise the Flash Memories while using different combinations of the most popular __RL78 Flash Libraries__.

## Coding Examples

| __Example__                       | __Description__ |
| :-------------------------------- | :-------------- |
| [T04-FDL](t04-fdl.md) | Creates a simple program which uses the T04-FDL Library in order to store some data in the RL78 __Data Flash__ | 
| [T01-FSL](t01-fsl.md) | Creates a simple program which uses the T01-FSL Library in order to store some data in the RL78 __Code Flash__ |
| [T02-EEL](t02-eel.md) | Creates a program which uses the T02-EEL Library and its corresponding dependency, the T02-FDL library. It exercises storing and retrieving data from the __EEL pool__ as well as from the __FDL pool__, both located into the RL78 __Data Flash__ |

---
If you have questions regarding this repository contents, you can start by checking its [issue tracker][repo-old-issue-url] for the previously asked questions.
If it is a new question, feel free to post it [here][repo-new-issue-url].

[repo-new-issue-url]: https://github.com/IARSystems/ewrl78-linker-config-flashlibs/issues/new
[repo-old-issue-url]: https://github.com/IARSystems/ewrl78-linker-config-flashlibs/issues?q=is%3Aissue+is%3Aopen%7Cclosed
