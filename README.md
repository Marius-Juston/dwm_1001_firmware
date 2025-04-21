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

# Event listneing

- `dwm_evet_listener_register` is in `libdwm.a/dwm.o`
- `uwbmac_listener_register` is in `libdwm.a/uwbmac.o`
- Once it has a location it then calls the `uwb_loc_cb_call` function in `libdwm.a/uwbmac.o`
- Where the function calls `uwbmac_cb` which is in `libdwm.a/dwm-api.o`

