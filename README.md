# Compiled file changed Steps

We want to have more anchors being outputted from the dwm_loc_get

- Install [Ghidra](https://github.com/NationalSecurityAgency/ghidra)
- We first need to change the the hard coded limit of the number of anchors that will be stored 
  - Open the `libdwm.a/dwm-api-cmd-infra.o` file
  - Search for the `undefined4 dwm_api_cmd_loc_get(undefined1 *param_1)` function
  - ```C
    if (iVar2 < 5) {
      *pbVar7 = (byte)iVar2;
    }
    else {
      *pbVar7 = 4;
    }
    ```
    where we want to change the 5 and 4 to 6 and 5, respectively, as such we have to change:
    ```
    00010114 05 2f           cmp        r7,#0x4
    00010116 ca bf           itet       gt
    00010118 05 23           mov.gt     r3,#0x4
    ```
    to
    ```
    00010114 05 2f           cmp        r7,#0x5
    00010116 ca bf           itet       gt
    00010118 05 23           mov.gt     r3,#0x5
    ```
    which thus results to:
    ```C
    if (iVar2 < 6) {
      *pbVar7 = (byte)iVar2;
    }
    else {
      *pbVar7 = 5;
    }
    ```
  - Other important functions
    - `uwbmac_loc_get` is at `libdwm.a/uwbmac.o`
    - `le_loc_get` is at `libdwm.a/process-le.o`

- Then we need to File -> Export Program -> Orginal File
- Then we need to relink the static library
  - `ar rcs libdwm.a dwm-api-cmd-infra.o`


- We also need to change inside the `uwbmac_twr_fsm_inr_entry` in `libdwm.a/uwbmac-sstwr.o`
 - ```C
    do {
      iVar12 = iVar4 + 1;
      if (iVar4 < (int)(uint)*(byte *)(iVar11 + 0x474)) {
        uVar5 = *puVar9;
      }
      else {
        uVar5 = 0xffff;
      }
      *(undefined2 *)(iVar3 + iVar4 * 2 + 0x10) = uVar5;
      puVar9 = puVar9 + 0x28;
      iVar4 = iVar12;
    } while (iVar12 != 4);
    ```
    To instead be `while (iVar12 != 5)`
    which requests changing 
    ```
    000104da 05 2e           cmp        r6,#0x4
    ```
    to 
    ```
    000104da 05 2e           cmp        r6,#0x5
    ```
- We also need to change inside the `uwbmac_twr_fsm_result_process` in `libdwm.a/uwbmac-tn.o`
 - ```C
    if (uVar11 == 5) {
      uVar5 = *(uint *)(iVar10 + 0xe4) & 0xffffffbf;
    }
    if (uVar11 == 5) {
      *(uint *)(iVar10 + 0xe4) = uVar5;
      uVar15 = CONCAT44(DAT_0001256c,DAT_00012568);
    }
    ```
    To instead be `while (iVar12 != 5)`
    which requests changing 
    ```
    000110ca 05 2d           cmp        r5,#0x4

    ```
    to 
    ```
    000110ca 05 2d           cmp        r5,#0x5
    ```
  - With also needing to increase the array of inputs datatype, from address `0x2000A048` (possible to change location but is the datatype that has the information about the anchors)
    ```
    000119dc 4f f4 c8 72     mov.w      r2,#0x140
    ```
    to
    ```
    000119dc 4f f4 c8 72     mov.w      r2,#0x190
    ```
    (80 bytes for each piece of information)

  - Need to modify the definition of `.anc_pos` as well so to do this:
      ```
      s::00000820 3b 03 00 00 08  Elf32_Shdr                        [52]
              00 00 00 03 00 
              00 00 00 00 00
      s::00000820 3b 03 00 00     ddw       33Bh                    sh_name       .bss.an_pos - SHT_
      s::00000824 08 00 00 00     Elf_Sect  SHT_NOBITS              sh_type
      s::00000828 03 00 00 00     ddw       3h                      sh_flags
      s::0000082c 00 00 00 00     ddw       an_pos                  sh_addr       = ??
      s::00000830 20 25 00 00     ddw       2520h                   sh_offset
      s::00000834 60 00 00 00     ddw       60h                     sh_size
      s::00000838 00 00 00 00     ddw       0h                      sh_link
      s::0000083c 00 00 00 00     ddw       0h                      sh_info
      s::00000840 08 00 00 00     ddw       8h                      sh_addralign
      s::00000844 00 00 00 00     ddw       0h                      sh_entsize
    ```
    and we want to be changing the `s::00000834 60 00 00 00     ddw       60h                     sh_size` to instead be `0x60` = 96 bytes = 4 anchors × 3 coordinates × 4 bytes to `0x78` = 120 bytes = 5 anchors × 3 coordinates × 4 bytes as such
    ```
      s::00000820 3b 03 00 00 08  Elf32_Shdr                        [52]
              00 00 00 03 00 
              00 00 00 00 00
      s::00000820 3b 03 00 00     ddw       33Bh                    sh_name       .bss.an_pos - SHT_
      s::00000824 08 00 00 00     Elf_Sect  SHT_NOBITS              sh_type
      s::00000828 03 00 00 00     ddw       3h                      sh_flags
      s::0000082c 00 00 00 00     ddw       an_pos                  sh_addr       = ??
      s::00000830 20 25 00 00     ddw       2520h                   sh_offset
      s::00000834 78 00 00 00     ddw       78h                     sh_size
      s::00000838 00 00 00 00     ddw       0h                      sh_link
      s::0000083c 00 00 00 00     ddw       0h                      sh_info
      s::00000840 08 00 00 00     ddw       8h                      sh_addralign
      s::00000844 00 00 00 00     ddw       0h                      sh_entsize
    ```

# Event listneing

- `dwm_evet_listener_register` is in `libdwm.a/dwm.o`
- `uwbmac_listener_register` is in `libdwm.a/uwbmac.o`
- Once it has a location it then calls the `uwb_loc_cb_call` function in `libdwm.a/uwbmac.o`
- Where the function calls `uwbmac_cb` which is in `libdwm.a/dwm-api.o`

