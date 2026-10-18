answer : Ea5yR3versing

해당하는 c 코드

```
void __cdecl FUN_00401080(HWND param_1)

{
  byte bVar1;
  byte *pbVar2;
  int iVar3;
  char *pcVar4;
  bool bVar5;
  CHAR local_64;
  char local_63;
  char local_62;
  char acStack_61 [97];

  local_64 = '\0';
  pcVar4 = &local_63;
  for (iVar3 = 0x18; iVar3 != 0; iVar3 = iVar3 + -1) {
    pcVar4[0] = '\0';
    pcVar4[1] = '\0';
    pcVar4[2] = '\0';
    pcVar4[3] = '\0';
    pcVar4 = pcVar4 + 4;
  }
  pcVar4[0] = '\0';
  pcVar4[1] = '\0';
  pcVar4[2] = '\0';
  GetDlgItemTextA(param_1,1000,&local_64,100);
  if (local_63 == 'a') {
    iVar3 = _strncmp(&local_62,&DAT_00406078,2);
    if (iVar3 == 0) {
      pcVar4 = s_AGR3versing_0040606a;
      pbVar2 = (byte *)(acStack_61 + 1);
      do {
        pcVar4 = pcVar4 + 2;
        bVar1 = *pbVar2;
        bVar5 = bVar1 < (byte)*pcVar4;
        if (bVar1 != *pcVar4) {
LAB_00401102:
          iVar3 = (1 - (uint)bVar5) - (uint)(bVar5 != 0);
          goto LAB_00401107;
        }
        if (bVar1 == 0) break;
        bVar1 = pbVar2[1];
        bVar5 = bVar1 < (byte)pcVar4[1];
        if (bVar1 != pcVar4[1]) goto LAB_00401102;
        pbVar2 = pbVar2 + 2;
      } while (bVar1 != 0);
      iVar3 = 0;
LAB_00401107:
      if ((iVar3 == 0) && (local_64 == 'E')) {
        MessageBoxA(param_1,s_Congratulation_!!_00406044,s_EasyCrackMe_00406058,0x40);
        EndDialog(param_1,0);
        return;
      }
    }
  }
  MessageBoxA(param_1,s_Incorrect_Password_00406030,s_EasyCrackMe_00406058,0x10);
  return;
}
```
