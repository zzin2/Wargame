# ELF x86 - Race condition

- Problem: https://www.root-me.org/en/Challenges/App-System/ELF-x86-Race-condition

We can see the source code as follows.
```
#include <string.h>
#include <sys/ptrace.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdlib.h>

#define PASSWORD "/challenge/app-systeme/ch12/.passwd"
#define TMP_FILE "/tmp/tmp_file.txt"

int main(void)
{
  int fd_tmp, fd_rd;
  char ch;


  if (ptrace(PTRACE_TRACEME, 0, 1, 0) < 0)
    {
      printf("[-] Don't use a debugguer !\n");
      abort();
    }
  if((fd_tmp = open(TMP_FILE, O_WRONLY | O_CREAT, 0444)) == -1)
    {
      perror("[-] Can't create tmp file ");
      goto end;
    }

  if((fd_rd = open(PASSWORD, O_RDONLY)) == -1)
    {
      perror("[-] Can't open file ");
      goto end;
    }

  while(read(fd_rd, &ch, 1) == 1)
    {
      write(fd_tmp, &ch, 1);
    }
  close(fd_rd);
  close(fd_tmp);
  usleep(250000);
end:
  unlink(TMP_FILE);

  return 0;
}
```

We can read the /tmp/tmp_file.txt when ch5 binary is running in the background.
```
app-systeme-ch12@challenge02:~$ ./ch12 & cat /tmp/tmp_temp.txt
[1] 4570
```
