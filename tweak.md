## Tweaking the ICF Trio for IAR Embedded Workbench for Renesas RL78 v2.x

The __ICF Trio__ comes ready to use with the __IAR Embedded Workbench for Renesas RL78 v3.x__ or later. It uses the linker configuration scheme __v2__ format. 

Nevertheless, it is possible to use the __ICF Trio__ with the __IAR Embedded Workbench for Renesas RL78 v2.x__, which relies on the earlier linker configuration scheme, the __v1__ format.

To enable the tweak, comment the [`define exported symbol __link_file_version_2 = 1;`](common.icf#L12) in the [common.icf](common.icf#L12) file, as  below:

```c
// Required symbol for the newer ICF format in IAR Embedded Workbench for RL78 v3.x and later to work
// (comment out the following symbol if using IAR Embedded Workbench for RL78 v2.x)
// define exported symbol __link_file_version_2 = 1;
```

The [common.icf](common.icf) file will then switch between two different reset modes:
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

[Back to the main ICF Trio Documentation Page](README.md#coding-examples)
