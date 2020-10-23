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
- [ICF Trio on IAR Embedded Workbench for RL78 v2.x](#icf-trio-on-iar-embedded-workbench-for-rl78-v2x)


## What is the purpose of the ICF Trio Project?

The __ICF Trio__ Project was designed for helping __IAR Embedded Workbench for RL78__ users to quickly setup their linker configurations.

The project's purpose is to provide a group of 3 files which composes the _Trio_ and they will work together towards an optimal linker configuration, delivering an improved method towards quicker and easier setup specifically for the __ILinker Configuration__. 

The __ICF Trio__ also offers one semi-automated way to reconfigure any project according to the memory reservation requirements when there are __RL78 Flash Libraries__ in use.

> __Note__
> * The __ICF Trio__ requires at least  __IAR Embedded Workbench for RL78__ version 2.10 or newer, defaulting to version 3.10 or later. There are some [instructions](README.md#icf-trio-on-iar-embedded-workbench-for-rl78-v2x) on how to proceed when using it with version 2.10.

## Benefits 
* Provides simplified setup for new projects relying on any of the __RL78 Flash Libraries__.
* Reduces the developer efforts when linker reconfiguration is needed, specially when facing device migration in regards to the __RL78 Flash Libraries__.
* Offers contiguous __Code Flash__ which leads to improved flexibility when compiled objects placement is performed at the linking stage. This is achieved by taking advantage of the linker capability of placing all the **__near** constants from the __ending__ of the mirrorable area in the __Code Flash__ whereas the default linker configuration would start filling the **__near** constants section from the beginning of the __Mirrorable Area__ (normally starting from the address _**0x2000**_ or _**0x3000**_ depending on the device in use).
* Reduces the __Code Flash__ fragmentation effect which may be noticeable on target devices with smaller flash memory installed, also thanks to its better strategy for object placement.

## Linker Configuration Trio Layout Specification

![ICF Trio layout](/images/icf_trio_layout.png)

| __File__                     | __Description__ |
| :--------------------------- | --------------- |
| [common.icf](common.icf)     | The first file is used *internally* by the ICF Trio. It is the heart of the Trio, containing parametrized directives which could, in thesis, be applied on any supported RL78. |
| [self_ram.icf](self_ram.icf) | The second one is also used *internally* by the ICF Trio. It handles every Self-RAM reservation needs based on the same [symbol styles](README.md#rl78-flash-libraries-documentation-and-their-respective-required-linker-symbols-for-self-ram) used in the `IAR Embedded Workbench for RL78`. |
| __[trio_lnkR5F1`nn`X`n`.icf](trio_lnkR5F1---TEMPLATE.icf)__ | The third one is user-selectable. The selection is made based on the similarity in the memory map for distinct groups of RL78 parts. Each of these files hold the proper `Linker Configuration Override` parameters which can be set on the `Project options`. Notice that the `X` in the file name means that the number of pins can be ignored for the purposes of the trio. |


## Flash Library Flavors

The Renesas RL78 MCUs require specific set of library(/ies) to enable usage of its Flash Memories.

Renesas Electronics provides the __RL78 Flash Libraries__ in 3 different flavors:
- The **FSL** (Flash Self-Programming Library) does program the RL78's __Code Flash__. 
- The **FDL** (Flash Data Library) does program the RL78's __Data Flash__.
- The **EEL** (EEPROM Emulation Library) does emulate EEPROM behaviour on the RL78's __Data Flash__. It provides an extra layer of functionalities on top of its corresponding companion FDL Library Type (T01 or T02 - [explained below](README.md#flash-library-types)). It provides transparent [__Wear Leveling__](https://en.wikipedia.org/wiki/Wear_leveling) among the provisioned __Data Flash__ blocks which may, in practice, virtually raise the amount of possible rewrites.


## Flash Library Types 

The __RL78 Flash Libraries__ flavors may be provided on 3 different library types:
- The **T01** (Type01, also known as **Full**) are the fully fledged Flash Libraries.
- The **T02** (Type02, also known as **Tiny**) are the balanced ones, providing the main functionalities at expense of less resources when compared with the T01 Libraries.
- The **T04** (Type04, also known as **Pico**) is the one providing only the essential functionalities. This library type offers the lowest resource usage footprint. Usually this is the typical choice for the scenarios where the chosen RL78 MCU part does not come with plenty of memory.

> __Note__
> * For further information regarding the complete feature set provided in each of these flash libraries, refer to their respective [documentation](README.md#rl78-flash-libraries-documentation-and-their-respective-required-linker-symbols-for-self-ram).


## Self-RAM 

Typically for every combination of RL78 MCU and __RL78 Flash Library__, the programmer would need to refer to the *Renesas Electronics*'  **[Application Note 2944](https://www.renesas.com/doc/products/tool/doc/015/r20ut2944ej0302_rl78.pdf)** in order to know if the chosen combination will require some specific RAM range to be reserved, therefore the chosen combination can function properly.

Self-RAM refers to the aforementioned RAM area, which __must__ be reserved on some cases, when relying on the RL78 MCU's self-programming capabilities.

In order to tremendously simplify this process, the __ICF Trio__ mostly automates it, by taking advantage of every advanced linker configuration directive available to override the default linker configuration, while following the requirements defined in the aforementioned Application Note. 


## How to use the ICF Trio

The following sections are going to serve as a straightforward step-by-step guide containing what is considered to be near the minimal actions that should be taken from scratch to configure a project to use the __ICF Trio__.

As reference, the RL78/G14 MCU (PN# __[R5F104LEAFA](https://www.renesas.com/products/microcontrollers-microprocessors/rl78/rl78g1x/rl78g14/device/R5F104LEAFA.html)__) will be used alongside the most popular __RL78 Flash Libraries__ combinations. 

Based on those steps, any other RL78 part which can use these __RL78 Flash Libraries__ could be then be used in a quite similar way. 

## Required Software 

The following software components were successfuly usable alongside the __ICF Trio__:
- [IAR Embedded Workbench for RL78 version 4.20](https://www.iar.com/iar-embedded-workbench/#!?architecture=RL78) 
- [Git for Windows](https://git-scm.com/download/win) 
- [Applilet3 for RL78](https://www.renesas.com/software/D4000916.html) 
- A __RL78 Flash Library__ of your choice - The download requires pre-registration ([here](https://www2.renesas.eu/products/micro/download/index.html/auth/register)) or Sign-in ([here](https://www2.renesas.eu/products/micro/download/index.html/auth/login)) specific to the Renesas Europe "MyPages" site.

### RL78 Flash Libraries, documentation and their respective required linker symbols for Self-RAM

|  __RL78 Flash Library__         | __Documentation__                        | __Symbol__            | __Description__                                                     |
| :-----------------------------: | :--------------------------------------: | :-------------------: | :------------------------------------------------------------------ |
| [T01-FSL Download][t01-fsl-url] | [T01-FSL Documentation][t01-fsl-doc-url] | `__RESERVE_T01_FSL=1` | Reserves Self-RAM for use with Code Flash Library __T01-FSL__       |
| [T01-FDL Download][t0x-xxl-url] | [T01-FDL Documentation][t01-fdl-doc-url] | `__RESERVE_T01_FDL=1` | Reserves Self-RAM for use with Data Flash Library __T01-FDL__       |
| [T01-EEL Download][t0x-xxl-url] | [T01-EEL Documentation][t01-eel-doc-url] | `__RESERVE_T01_EEL=1` | Reserves Self-RAM for use with EEPROM Emulation Library __T01-EEL__ |
| [T02-FDL Download][t0x-xxl-url] | [T02-FDL Documentation][t02-fdl-doc-url] | `__RESERVE_T02_FDL=1` | Reserves Self-RAM for use with Tiny Data Flash Library __T02-FDL__  |
| [T02-EEL Download][t0x-xxl-url] | [T02-EEL Documentation][t02-eel-doc-url] | `__RESERVE_T02_EEL=1` | Reserves Self-RAM for use with EEPROM Emulation Library __T02-EEL__ |
| [T04-FDL Download][t0x-xxl-url] | [T04-FDL Documentation][t04-fdl-doc-url] | `__RESERVE_T04_FDL=1` | Reserves Self-RAM for use with Pico Flash Lbrary __T04-FDL__        |

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
> * When installing the IAR Embedded Workbench for RL78, the installer also will offer the possibility to install a code generator tool named [AP4](https://www.renesas.com/products/software-tools/tools/code-generator/ap4-applilet.html). The AP4 is a GUI-based code generator that can generate peripheral drivers in C source containing driver functions for some of the RL78 parts.

**2.** For the purposes of this general guide, the selected part which will be used is a general purpose RL78/G14 (PN# __[R5F104LEAFA](https://www.renesas.com/products/microcontrollers-microprocessors/rl78/rl78g1x/rl78g14/device/R5F104LEAFA.html)__). The part belongs to the *RL78/G14 Series*. The G14 Series uses a previous version of the AP4, the [Applilet3 for RL78](https://www.renesas.com/software/D4000916.html) configuration tool, which should then be installed.

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
> * The __$PROJ_DIR$__ is an argument variable which can be conveniently used refer to relative pathnames when accessing the project's resources in the storage media. With __IAR Embedded Workbench__, you can use a wide range of built-in argument variables, as well as create your own. For more information search for `Argument Variables` on the [__IAR Embedded Workbench IDE Project Management and Building Guide__][argvars-url].

[argvars-url]: https://netstorage.iar.com/SuppDB/Public/UPDINFO/014217/ew/doc/EWRL78_IDEGuide.ENU.pdf#page=82

**9.** Choose `Project` → `Create New Project...` and create an empty RL78 project. Save it on the project's __$PROJ_DIR$__ location.

**10.** Choose `Project` → `Add Project Connection...` and point to the __.ipcf__ which has been created when the __Applilet3__ code was generated.

![Workspace](/images/7workspace.png)

**11.** Invoke the context menu (right-click) on the __.ipcf__ file and choose `Open Containing Folder`. Invoke the context menu for the folder in the __Windows Explorer__ and choose `Git Bash Here`. This step will open a Git Bash Shell at `Place` (__$PROJ_DIR$__). From there perform:

 ```bash
 git clone https://github.com/IARSystems/ewrl78-linker-config-flashlibs.git
 ```
 
 ~~~
 devel@per-machine MING64 /c/FDL_Project
 $ git clone https://github.com/IARSystems/ewrl78-linker-config-flashlibs.git
 Cloning into 'ewrl78-linker-config-flashlibs'...
 remote: Counting objects: 58, done.
 remote: Compressing objects: 100% (33/33), done.
 remote: Total 58 (delta 36), reused 34 (delta 30), pack-reused 0
 Unpacking objects: 100% (58/58), done.
 
 devel@per-machine MING64 /c/FDL_Project
 $
 ~~~
 
**12.** Back to the __IAR Embedded Workbench for RL78__, go to the Project Options, `Linker` → `Config` → `Override default` and choose the right part number. For this example, __R5F104LEA__ will be used, hence the [trio_lnkR5F104xE.icf](trio_lnkR5F104xE.icf) should (and will) be selected:
```
$PROJ_DIR$\ewrl78-linker-config-flashlibs\trio_lnkR5F104xE.icf
```

From this point you can now choose one of the examples below, which contains further steps for creating simple programs which exercise the Flash Memories while using different combinations of the most popular __RL78 Flash Libraries__.

## Coding Examples

| __Example__                       | __Description__ |
| :-------------------------------- | :-------------- |
| [Example for T04-FDL](t04-fdl.md) | Creates a simple program which uses the T04-FDL Library in order to store some data in the RL78 __Data Flash__ | 
| [Example for T01-FSL](t01-fsl.md) | Creates a simple program which uses the T01-FSL Library in order to store some data in the RL78 __Code Flash__ |
| [Example for T02-EEL](t02-eel.md) | Creates a program which uses the T02-EEL Library and its corresponding dependency, the T02-FDL library. It exercises storing and retrieving data from the __EEL pool__ as well as from the __FDL pool__, both located into the RL78 __Data Flash__ |

## ICF Trio on IAR Embedded Workbench for RL78 v2.x

The __ICF Trio__ comes ready to use with IAR __Embedded Workbench for RL78 v3__ or later as it, by default. It uses the ICF __v2__ scheme. Nevertheless the __ICF Trio__ still can be used with the IAR Embedded Workbench __v2__, which relies on an earlier format for the linker configuration, the ICF __v1__ scheme. In order to do so, a minor manual switch has to be toggled, simply by commenting the exported symbol **__link_file_version_2** in the [Common.ICF](common.icf) file, as illustrated bellow:

```c
// Required symbol for the newer ICF format in IAR Embedded Workbench for RL78 v3.x and later to work
// (comment out the following symbol if using IAR Embedded Workbench for RL78 v2.x)
// define exported symbol __link_file_version_2 = 1;
```

By doing so, the [common.icf](common.icf) file will then switch between two different reset modes, in accordance to the following:

```c
if (isdefinedsymbol(__link_file_version_2))
{
    // Link file version 2 placement scheme is used in IAR Embedded Workbench for RL78 v3.x and later
  place at address mem:0x00000       { ro section .reset };
  place at address mem:0x00004       { ro section .intvec };
}
else
{
  // Link file version 1 placement scheme is used in IAR Embedded Workbench for RL78 v2.x
  place at address mem:0x00000       { ro section .intvec };
}
```

---
If you have questions regarding this repository contents, you can start by checking its [issue tracker][repo-old-issue-url] for the previously asked questions.
If it is a new question, feel free to post it [here][repo-new-issue-url].

[repo-new-issue-url]: https://github.com/IARSystems/ewrl78-linker-config-flashlibs/issues/new
[repo-old-issue-url]: https://github.com/IARSystems/ewrl78-linker-config-flashlibs/issues?q=is%3Aissue+is%3Aopen%7Cclosed
