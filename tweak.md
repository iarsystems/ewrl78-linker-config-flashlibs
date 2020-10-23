## ICF Trio on IAR Embedded Workbench for RL78 v2.x

The __ICF Trio__ comes ready to use with IAR __Embedded Workbench for RL78 v3__ or later as it, by default. It uses the ICF __v2__ scheme. Nevertheless the __ICF Trio__ still can be used with the IAR Embedded Workbench __v2__, which relies on an earlier format for the linker configuration, the ICF __v1__ scheme. In order to do so, a minor manual switch has to be toggled, simply by commenting the exported symbol `__link_file_version_2` in the [common.icf](common.icf) file, as  below:

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
If you have questions regarding this repository contents, you can start by checking its [issue tracker][repo-old-issue-url] for the previously asked questions.
If it is a new question, feel free to post it [here][repo-new-issue-url].

[repo-new-issue-url]: https://github.com/IARSystems/ewrl78-linker-config-flashlibs/issues/new
[repo-old-issue-url]: https://github.com/IARSystems/ewrl78-linker-config-flashlibs/issues?q=is%3Aissue+is%3Aopen%7Cclosed
