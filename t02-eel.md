## Project Setup Example for T02-EEL (tiny EEL) 
You can refer to the previous setup steps [here](README.md#how-to-use-the-icf-trio).

- Install the [RL78 T02-EEL Library](https://www2.renesas.eu/products/micro/download/?oc=EEPROM_EMULATION_RL78) you have [previously downloaded](README.md#pre-requisites). During the installation, select to install for the __IAR Compiler v2.10__ (or later) on the project folder (`$PROJ_DIR$`). The installer will create a folder named `EEL\IAR_210` (or similar).

### Linker configuration
- In the IDE, go to the __Project__ → __Options__ → __Linker__ → __Config__ → __Linker configuration file__.

- Tick __Override default__ and select the corresponding __trio_lnk\<device\>.icf__ for the target device in use. In this example, we are using the R5F104LEA target, so the [trio_lnkR5F104xE.icf](trio_lnkR5F104xE.icf) will be selected: `$PROJ_DIR$\ewrl78-linker-config-flashlibs\trio_lnkR5F104xE.icf`

- In the same configuration tab, append `__RESERVE_T02_EEL=1` to the __Configuration file symbol definition__, as below: 

![T02 Linker Configuration](images/reserve_t02_eel.png)

- In __Linker__ → __Library__ → __Additional libraries__, add the following line:
```
$PROJ_DIR$\EEL\IAR_210\FDL\lib\fdl.a
$PROJ_DIR$\EEL\IAR_210\EEL\lib\eel.a
```

- In __C/C++ Compiler__ → __Preprocessor__ → __Additional include directories__, add the following 2 lines:
```
$PROJ_DIR$\applilet3_src
$PROJ_DIR$\EEL\IAR_210\FDL\lib
$PROJ_DIR$\EEL\IAR_210\EEL\lib
```

> __Note__ Each target device has its own memory reservation requirements. In order to get the best out of the trio configurations, please choose the appropriate `trio_lnk<device>.icf` linker configuration for the actual device you are using in your project, directly from the repository you cloned inside the project's directory (`$PROJ_DIR$\ewrl78-linker-config-flashlibs`).

### Example source code
Now it is time to update the application with this example source code below that exercises some of the library's capabilities. The execution flow exercises some basic flash operations using both __EEL__ and __FDL__ layers.

- Open the `Renesas_AP\cg_src\r_main.c` and insert the FDL and EEL headers between the generated comment guards for the "user code for include", as below:
```c
...
/* Start user code for include. Do not edit comment generated here */
#include "fdl.h"
#include "fdl_types.h"
#include "eel.h"
#include "eel_types.h"
/* End user code. Do not edit comment generated here */
#include "r_cg_userdefine.h"
...
```

- Add the following snippet of code in the _Global variables and functions_, as below:
```c
/********************************...**
Global variables and functions
*********************************...**/
/* Start user code for global. Do not edit comment generated here */

/* FDL and EEL macros */
#define FDL_BLOCK_SIZE  (0x400U)
#define FDL_FREQ_MHZ    (32)
#define FDL_DELAY       ((fdl_u16)(10*FDL_FREQ_MHZ/6))

#define FDL_POOL_BLOCKS (1)
#define FDL_POOL_BYTES  (FDL_POOL_BLOCKS*FDL_BLOCK_SIZE)

#define EEL_POOL_BLOCKS (3)
#define EEL_POOL_BYTES  (EEL_POOL_BLOCKS*FDL_BLOCK_SIZE)

/* Global data variables */
#define STRING_SZ (16)

typedef fdl_u08 fdl_configdata_t[STRING_SZ];

typedef eel_u08 eel_record01_t;
typedef eel_u08 eel_record04_t[STRING_SZ/4];
typedef eel_u08 eel_record16_t[STRING_SZ];
typedef eel_u08 eel_record64_t[4*STRING_SZ];
enum {EEL_RECZZ_NULL,
      EEL_REC01_ID,
      EEL_REC04_ID,
      EEL_REC16_ID,
      EEL_REC64_ID,
      EEL_REC_TERMINATOR,
      EEL_DESCRIPTOR_SZ,};

#pragma location="EEL_CNST"
#pragma data_alignment=2
__far const eel_u08 eel_descriptor[EEL_DESCRIPTOR_SZ] =
{
  (eel_u08)(EEL_REC_TERMINATOR-1),    /* variable count   */  \
  (eel_u08)(sizeof(eel_record01_t)),  /* id=1             */  \
  (eel_u08)(sizeof(eel_record04_t)),  /* id=2             */  \
  (eel_u08)(sizeof(eel_record16_t)),  /* id=3             */  \
  (eel_u08)(sizeof(eel_record64_t)),  /* id=4             */  \
  (eel_u08)(0x00),                    /* zero terminator  */  \
};

#pragma location="EEL_CNST"
#pragma data_alignment=2
__far const eel_u08  eel_internal_cfg_cau08[] = { 0x40 };

/* Populate the EEL "wr_recordXX" records */ 
eel_record01_t wr_record01 = 'S';
eel_record04_t wr_record04 = "XYWZ";
eel_record16_t wr_record16 = "0123456789ABCDEF";
eel_record64_t wr_record64 = "0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF";
eel_record64_t w2_record64 = "A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5A5";

/* Declare the EEL "rd_recordXX" records for read-back  */
eel_record01_t rd_record01;
eel_record04_t rd_record04;
eel_record16_t rd_record16;
eel_record64_t rd_record64;

/* Populates the FDL data variable */
fdl_configdata_t wr_configdata = "#012.456.89A.CDE";

/* Declares the FDL data variable to read-back*/
fdl_configdata_t rd_configdata;

/* End user code. Do not edit comment generated here */
```

- Replace the original `main()` function with the following code snippet.
```c
void main(void)
{
    R_MAIN_UserInit();
    /* Start user code. Do not edit comment generated here */
    
    /* FDL and EEL local vars */
    fdl_request_t    FDL_request;
    fdl_descriptor_t FDL_descriptor;
    eel_request_t    EEL_request;
    uint8_t          xxL_status;

    /* Initialize T02-FDL */
    FDL_descriptor.eel_pool_bytes_u16    = EEL_POOL_BYTES;
    FDL_descriptor.fdl_pool_bytes_u16    = FDL_POOL_BYTES;
    FDL_descriptor.fdl_delay_u16         = FDL_DELAY;
    FDL_descriptor.eel_pool_blocks_u08   = EEL_POOL_BLOCKS;
    FDL_descriptor.fdl_pool_blocks_u08   = FDL_POOL_BLOCKS;
    FDL_descriptor.fx_MHz_u08            = FDL_FREQ_MHZ;
    FDL_descriptor.wide_voltage_mode_u08 = 0x00U;
    xxL_status = FDL_Init(&FDL_descriptor);
    if (FDL_OK != xxL_status)
    {
      while(1);
    }
    FDL_Open();

    /* Initialize T02-EEL */
    xxL_status = EEL_Init();
    if (EEL_OK != xxL_status)
    {
      while(1);
    }
    EEL_Open();

    /* Format the EEL pool before it can be used */
    EEL_request.command_enu = EEL_CMD_FORMAT;
    EEL_Execute(&EEL_request);
    do { EEL_Handler(); } while (EEL_BUSY == EEL_request.status_enu);

    /* Start the EEL pool */
    EEL_request.command_enu = EEL_CMD_STARTUP;
    EEL_Execute(&EEL_request);
    do { EEL_Handler(); } while (EEL_BUSY == EEL_request.status_enu);

    /* Write on different EEL variables */
    EEL_request.command_enu = EEL_CMD_WRITE;
    EEL_request.address_pu08 = &wr_record01;
    EEL_request.identifier_u08 = EEL_REC01_ID;
    EEL_Execute(&EEL_request);
    do { EEL_Handler(); } while (EEL_BUSY == EEL_request.status_enu);
    EEL_request.address_pu08 = wr_record04;
    EEL_request.identifier_u08 = EEL_REC04_ID;
    EEL_Execute(&EEL_request);
    do { EEL_Handler(); } while (EEL_BUSY == EEL_request.status_enu);
    EEL_request.address_pu08 = wr_record16;
    EEL_request.identifier_u08 = EEL_REC16_ID;
    EEL_Execute(&EEL_request);
    do { EEL_Handler(); } while (EEL_BUSY == EEL_request.status_enu);
    EEL_request.address_pu08 = wr_record64;
    EEL_request.identifier_u08 = EEL_REC64_ID;
    EEL_Execute(&EEL_request);
    do { EEL_Handler(); } while (EEL_BUSY == EEL_request.status_enu);
    
    /*
     *  T02-EEL allows record updates to be performed 
     *  without requiring full block erasure upfront 
     */
    EEL_request.address_pu08 = w2_record64;
    EEL_request.identifier_u08 = EEL_REC64_ID;
    EEL_Execute(&EEL_request);
    do { EEL_Handler(); } while (EEL_BUSY == EEL_request.status_enu);
    
    /* Read the previously EEL variables */
    EEL_request.command_enu = EEL_CMD_READ;
    EEL_request.address_pu08 = &rd_record01;
    EEL_request.identifier_u08 = EEL_REC01_ID;
    EEL_Execute(&EEL_request);
    do { EEL_Handler(); } while (EEL_BUSY == EEL_request.status_enu);
    EEL_request.address_pu08 = rd_record04;
    EEL_request.identifier_u08 = EEL_REC04_ID;
    EEL_Execute(&EEL_request);
    do { EEL_Handler(); } while (EEL_BUSY == EEL_request.status_enu);
    EEL_request.address_pu08 = rd_record16;
    EEL_request.identifier_u08 = EEL_REC16_ID;
    EEL_Execute(&EEL_request);
    do { EEL_Handler(); } while (EEL_BUSY == EEL_request.status_enu);
    EEL_request.address_pu08 = rd_record64;
    EEL_request.identifier_u08 = EEL_REC64_ID;
    EEL_Execute(&EEL_request);
    do { EEL_Handler(); } while (EEL_BUSY == EEL_request.status_enu);
                               
    /* Erase FDL config data block */
    FDL_request.command_enu = FDL_CMD_ERASE_BLOCK;
    FDL_request.index_u16 = 0x0000U;   // Virtual address for the FDL Pool
    FDL_Execute(&FDL_request);
    do { FDL_Handler(); } while (FDL_BUSY == FDL_request.status_enu);
    
    /* Write FDL config data */
    FDL_request.command_enu = FDL_CMD_WRITE_BYTES;
    FDL_request.bytecount_u16 = sizeof(fdl_configdata_t);
    FDL_request.data_pu08 = wr_configdata;
    FDL_Execute(&FDL_request);
    do { FDL_Handler(); } while (FDL_BUSY == FDL_request.status_enu);
  
    /* Read FDL config data */
    FDL_request.command_enu = FDL_CMD_READ_BYTES;
    FDL_request.data_pu08 = rd_configdata;
    FDL_Execute(&FDL_request);
    do { FDL_Handler(); } while (FDL_BUSY == FDL_request.status_enu);

    /* EEL and FDL shutdown */
    EEL_request.command_enu = EEL_CMD_SHUTDOWN;
    EEL_Execute(&EEL_request);
    EEL_Close();
    FDL_Close();
    
    while (1U)
    {
        ;
    }
    /* End user code. Do not edit comment generated here */
}
```

- By default, the ICF Trio configuration reserves the memory to work with the maximum amount of variables allowed by the library. For this case, open the [trio_lnkrR5F104xE.icf](trio_lnkR5F104xE.icf) or corresponding file and then reduce the amount of `T02_EEL_NUM_VARS` to `4`, in order to significantly save on resource usage. The Self-RAM usage for EEL variables goes from `384` bytes to `264` bytes (a 31.25% reduction), as follows:
```c
// Maximum number of variables for EEPROM Emulation Layer Libraries
define symbol _T01_EEL_NUM_VARS = 255; // [1~255 variables]
define symbol _T02_EEL_NUM_VARS = 4;   // [1~64  variables] 
```

## Configuring and debugging the project

- Go to the Project Options, __General Options__ → __Target__ → __Device__ and choose the desired part number.

- In the project options, __Debugger__ → __Setup__ → __Driver__ and choose the emulator you have. Typically _TK_, _E1_ or _E2 Lite_ depending on the emulator in use.

- Start a new C-SPY debugging session by choosing __Project__ → __Download and Debug__. If necessary, choose the right _Power supply_ voltage for the _Target system_ in the _Emulator Hardware Setup window_. In this case, as the __LVD__ was set to `3.63V`, the choosen voltage was `5V`.

- Tick the __Erase flash before next ID check__ checkbox.

- Click _OK_ to close the _Hardware Setup window_.

- By default, C-SPY will execute the application until it reaches a breakpoint in the beginning of the `main()` function. Insert a breakpoint near the `FDL_Close()` call in the end of the `main()` function.

- Activate the _Watch1 window_ by selecting __View__ → __Watch__ → __Watch1__. This window will allow you to add expressions to watch the contents of global variables used to hold different record types used to test the _EEL pool_.

- __\<Click to add\>__ all the `rd_recordNN` and `wr_recordNN` variables (`NN` can be `01`|`04`|`16`|`64`).

- Activate the _Memory1 window_ by selecting  __View__ → __Memory__ → __Memory1__. _Go to_ the beginning of the _FDL pool_ in this example, at address `0xF1C00`.

- Hit __Go__ (<kbd>F5</kbd>) on the _Debug toolbar_ to execute the program until it reaches the breakpoint.

- In the _Watch1 window_, verify if the contents of the variables `rd_recordNN` and `wr_recordNN` do match.

- In the _Memory1 window_, verify the data written to the _FDL pool_ @ `0xF1C00`.

![T02-FDL Memory View](/images/t02_fdl_memory1.png)

[< ICF Trio documentation page](README.md#examples)
