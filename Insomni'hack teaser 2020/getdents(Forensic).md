# Question

### getdents by kevin
```
Oh shit! Data have been stolen from my computer... I looked for malicious activity but found nothing suspicious. Could ya give me a hand and find the malware and how it's hiding?
```

# Analysis

This question provides a memory dump and ubuntu profile for memory forensic. The ubuntu profile appeared in volatility after moving the given ubunto profile to "/usr/local/Cellar/volatility/2.6.1/libexec/lib/python2.7/site-packages/volatility/plugins/overlays/linux" in MacOS.

```
eloi@NULL@ROOT linux % vol.py --info
Volatility Foundation Volatility Framework 2.6.1

Profiles
--------
LinuxUbuntu_4_15_0-72-generic_profilex64 - A Profile for Linux Ubuntu_4.15.0-72-generic_profile x64
VistaSP0x64                              - A Profile for Windows Vista SP0 x64
VistaSP0x86                              - A Profile for Windows Vista SP0 x86
VistaSP1x64                              - A Profile for Windows Vista SP1 x64
VistaSP1x86                              - A Profile for Windows Vista SP1 x86
VistaSP2x64                              - A Profile for Windows Vista SP2 x64
VistaSP2x86                              - A Profile for Windows Vista SP2 x86
```

Meterpreter process (PID 1751) was found in bash command history and process list.

```
eloi@NULL@ROOT memory_a97a5b9a792b61131eb6193e09c69616df875bb43539359af0e421b1b0798ba7 % vol.py -f memory.vmem --profile=LinuxUbuntu_4_15_0-72-generic_profilex64 linux_bash
Volatility Foundation Volatility Framework 2.6.1
Pid      Name                 Command Time                   Command
-------- -------------------- ------------------------------ -------
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo apt upgrade
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo apt upgrade
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo apt update
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo apt update
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo reboot
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo reboot
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo apt upgrade
    1733 bash                 2020-01-16 14:00:36 UTC+0000   rub
    1733 bash                 2020-01-16 14:00:36 UTC+0000   uname -a
    1733 bash                 2020-01-16 14:00:36 UTC+0000   AWAVH??
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo apt update
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo apt autoclean 
    1733 bash                 2020-01-16 14:00:36 UTC+0000   uname -a
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo apt upgrade
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo reboot
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo apt upgrade
    1733 bash                 2020-01-16 14:00:36 UTC+0000   sudo reboot
    1733 bash                 2020-01-16 14:00:41 UTC+0000   chmod +x meterpreter
    1733 bash                 2020-01-16 14:00:42 UTC+0000   sudo ./meterpreter
```
```
eloi@NULL@ROOT memory_a97a5b9a792b61131eb6193e09c69616df875bb43539359af0e421b1b0798ba7 % vol.py -f memory.vmem --profile=LinuxUbuntu_4_15_0-72-generic_profilex64 linux_pslist  
Volatility Foundation Volatility Framework 2.6.1
Offset             Name                 Pid             PPid            Uid             Gid    DTB                Start Time
------------------ -------------------- --------------- --------------- --------------- ------ ------------------ ----------
...
0xffff8a9dcf5edd00 meterpreter          1751            1750            0               0      0x0000000014540000 2020-01-16 14:01:22 UTC+0000
...
0xffff8a9dc3c40000 sh                   2964            1751            0               0      0x0000000076aec000 2020-01-16 14:02:57 UTC+0000
```

However, meterpreter process did not have interesting information. 

```
eloi@NULL@ROOT memory_a97a5b9a792b61131eb6193e09c69616df875bb43539359af0e421b1b0798ba7 % vol.py -f memory.vmem --profile=LinuxUbuntu_4_15_0-72-generic_profilex64 linux_procdump -D ./ -p 1751    
Volatility Foundation Volatility Framework 2.6.1
Offset             Name                 Pid             Address            Output File
------------------ -------------------- --------------- ------------------ -----------
0xffff8a9dcf5edd00 meterpreter          1751            0x0000000000400000 ./meterpreter.1751.0x400000

eloi@NULL@ROOT memory_a97a5b9a792b61131eb6193e09c69616df875bb43539359af0e421b1b0798ba7 % strings meterpreter.1751.0x400000 
j"AZ
AYPj)X
Zj*X
Wj#Xj
YY_H
j<Xj
^j~Z
```

Based on process list above, sh process (PID 2964) was generated by the meterpreter. After extracting the process, found some interesting strings like "/home/julien/Downloads/rkit.ko", "hide=rJ/1g5PA5amy176A64akjuq/jryOug=="

```
eloi@NULL@ROOT memory_a97a5b9a792b61131eb6193e09c69616df875bb43539359af0e421b1b0798ba7 % vol.py -f memory.vmem --profile=LinuxUbuntu_4_15_0-72-generic_profilex64 linux_procdump -D ./ -p 2964
Volatility Foundation Volatility Framework 2.6.1
Offset             Name                 Pid             Address            Output File
------------------ -------------------- --------------- ------------------ -----------
0xffff8a9dc3c40000 sh                   2964            0x000055f26b1a8000 ./sh.2964.0x55f26b1a8000

eloi@NULL@ROOT memory_a97a5b9a792b61131eb6193e09c69616df875bb43539359af0e421b1b0798ba7 % strings sh.2964.0x55f26b1a8000 
...
insmod /home/julien/Downloads/rkit.ko hide=rJ/1g5PA5amy176A64akjuq/jryOug== hide_pid=1751
insmod
/home/julien/Downloads/rkit.ko
8{<k
hide=rJ/1g5PA5amy176A64akjuq/jryOug==
h{<k
hide_pid=1751
X{<k
insmod
/home/julien/Downloads/rkit.ko
hide=rJ/1g5PA5amy176A64akjuq/jryOug==
8|<k
hide_pid=1751
(|<k
(|<k
/sbin/insmod
smod
X{<k
```

To acquire the suspicious file "rkit.ko" from memory, used linux_enumerate_files and linux_find_file commands. This is the actual malware file that mentioned the question, and it contained another an interesting string "wC4jSbGOktXTIfdsHFuKoU7nZVvLq0eWl1mBQr9P5JEpMyN832AY6chRigDaxz". Guessed this string work with "rJ/1g5PA5amy176A64akjuq/jryOug==" to generate flag string.

```
eloi@NULL@ROOT memory_a97a5b9a792b61131eb6193e09c69616df875bb43539359af0e421b1b0798ba7 % vol.py -f memory.vmem --profile=LinuxUbuntu_4_15_0-72-generic_profilex64 linux_enumerate_files | grep rkit.ko
Volatility Foundation Volatility Framework 2.6.1
0xffff8a9dd42755e8                   2359303 /home/home/home/julien/Downloads/rkit.ko

eloi@NULL@ROOT memory_a97a5b9a792b61131eb6193e09c69616df875bb43539359af0e421b1b0798ba7 % vol.py -f memory.vmem --profile=LinuxUbuntu_4_15_0-72-generic_profilex64 linux_find_file -i 0xffff8a9dd42755e8 -O rkit.ko_
Volatility Foundation Volatility Framework 2.6.1

eloi@NULL@ROOT memory_a97a5b9a792b61131eb6193e09c69616df875bb43539359af0e421b1b0798ba7 % strings rkit.ko_ 
AWAVAUATSH
0[A\A]A^A_]
APH=
APH=
[A\A]]
tIUH
ens33
hide_
license=GPL
description=Nice and clean kernel module.. ;-)
author=Kevin1337
parmtype=hide_pid:uint
parmtype=hide:charp
srcversion=BE3F739755DCEB7B9B84E7C
depends=
retpoline=Y
name=rkit
vermagic=4.15.0-72-generic SMP mod_unload 
ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
hide_pid
hide
wC4jSbGOktXTIfdsHFuKoU7nZVvLq0eWl1mBQr9P5JEpMyN832AY6chRigDaxz
rkit
GCC: (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0
GCC: (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0
__check_object_size
param_ops_uint
_copy_from_user
__this_module
cleanup_module
sub_8000288
kfree
__fentry__
init_module
pv_cpu_ops
__x86_indirect_thunk_rax
__stack_chk_fail
sub_8000266
init_net
strncmp
sub_80005EE
memset
param_ops_charp
_copy_to_user
syscall_table
sub_80005F9
original_getdents
strlen
sub_8000AB8
__kmalloc
.symtab
.strtab
.shstrtab
.note.gnu.build-id
.rela.text
.rodata.str1.1
.rela.parainstructions
.modinfo
.rodata
.rela__param
.rela__mcount_loc
.rela.data
.rela.gnu.linkonce.this_module
.bss
.comment
.note.GNU-stack
```

Based on the binary analysis with IDA, found the variable names "pk" and "sk". The "pk" had string value in data segment and the "sk" was in bss segment, but it had unknown value in this stage.

```
.data:0000000000000900 pk              db 'wC4jSbGOktXTIfdsHFuKoU7nZVvLq0eWl1mBQr9P5JEpMyN832AY6chRigDaxz',0"
```
```
.text:0000000000000361                 mov     r11, offset sk
.text:0000000000000368                 mov     rdi, offset pk  ; "wC4jSbGOktXTIfdsHFuKoU7nZVvLq0eWl1mBQr9"...
```
```
.bss:0000000000000CE0 sk              db    ? ;               ; DATA XREF: sub_70+7D↑o
```

To get more information of the binary, analysed it with GHIDRA as well and found below code that mentioned some calculation between varable "pk" and "sk". Suspected varible "sk" is an encrypted string of "pk" string value.  

```
      do {
        sk[lVar3] = *(byte *)(lVar2 + (long)(int)(uVar4 % 6)) ^ pk[lVar3];
        lVar3 = lVar3 + 1;
        uVar4 = uVar4 + 0x2a;
      } while (lVar3 != 0x3e);
```

Also, discovered another variable name "ep" in GHIDRA. Suspected that the base64 string "rJ/1g5PA5amy176A64akjuq/jryOug==" was the value of this variable cause the function sub_80005EE had base64 logic. The "ep" had some calculation logic with "sk" which is an encrypted string of "pk" variable

```
ep = sub_80005EE(puVar1,sVar2,&epl);
```
```
    do {
      iVar3 = iVar3 + 1;
      pbVar4 = (byte *)(ep + uVar2);
      lVar1 = uVar2 + ((uVar2 >> 1) / 0x1f) * -0x3e;
      uVar2 = SEXT48(iVar3);
      *pbVar4 = *pbVar4 ^ sk[lVar1];
    } while (uVar2 < epl);
```

Assumed the value of "sk" also captured in the memory image, so used "linux_check_modules" and "linux_moddump" to dump kernel module "rkit" that related to the binary "rkit.ko". With IDA, found some HEX values "E5D1A6F8C1F0D5DDF9E6CAC6DBF4F6E1DAD4E7D9FDC7A5FCC8C4E4DEE3A2F7C5FEA3FFD0C3E0ABC2A7D8D7E2DFEBDCAAA1A0D3CBA4F1FAC0FBF5D6F3EAE8" in the bss segment as the variable "sk" found in bss segment of "rkit.ko".

```
eloi@NULL@ROOT memory_a97a5b9a792b61131eb6193e09c69616df875bb43539359af0e421b1b0798ba7 % vol.py -f memory.vmem --profile=LinuxUbuntu_4_15_0-72-generic_profilex64 linux_check_modules                      
Volatility Foundation Volatility Framework 2.6.1
    Module Address       Core Address       Init Address Module Name             
------------------ ------------------ ------------------ ------------------------
0xffffffffc0943080 0xffffffffc0941000                0x0 rkit

eloi@NULL@ROOT memory_a97a5b9a792b61131eb6193e09c69616df875bb43539359af0e421b1b0798ba7 % vol.py -f memory.vmem --profile=LinuxUbuntu_4_15_0-72-generic_profilex64 linux_moddump -D ./ -b 0xffffffffc0943080
Volatility Foundation Volatility Framework 2.6.1
Wrote 4146551 bytes to rkit.0xffffffffc0943080.lkm

00000000080023B0  00 00 00 00 00 00 00 00  00 00 00 00 E5 D1 A6 F8  ................
00000000080023C0  C1 F0 D5 DD F9 E6 CA C6  DB F4 F6 E1 DA D4 E7 D9  ................
00000000080023D0  FD C7 A5 FC C8 C4 E4 DE  E3 A2 F7 C5 FE A3 FF D0  .ǥ .............
00000000080023E0  C3 E0 AB C2 A7 D8 D7 E2  DF EB DC AA A1 A0 D3 CB  ...§ .....ܪ ....
00000000080023F0  A4 F1 FA C0 FB F5 D6 F3  EA E8 00 00 16 00 00 00  ................
```

Got flag with below python script that calculate XOR opeation with the values of "ep" and "sk"" variables.

```
import base64
import binascii

b64param = base64.b64decode("rJ/1g5PA5amy176A64akjuq/jryOug==")
enc_sk = binascii.unhexlify("E5D1A6F8C1F0D5DDF9E6CAC6DBF4F6E1DAD4E7D9FDC7A5FCC8C4E4DEE3A2F7C5FEA3FFD0C3E0ABC2A7D8D7E2DFEBDCAAA1A0D3CBA4F1FAC0FBF5D6F3EAE8")

flag = ""

for i in range(len(b64param)):
    flag += chr(ord(b64param[i]) ^ ord(enc_sk[i]))

print flag
```

# Flag

INS{R00tK1tF0rRo0kies}
