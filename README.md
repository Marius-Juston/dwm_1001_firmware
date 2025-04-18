# Compiled file changed Steps

We want to have more anchors being outputted from the dwm_loc_get

- Install [Ghidra](https://github.com/NationalSecurityAgency/ghidra)
- We first need to change the the hard coded limit of the number of anchors that will be stored 
  - Open the libdwm.a/dwm-api-cmd-infra.o file
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
