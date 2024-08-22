# Toddler's Bottle -- fd

# Challenge description
```
Mommy! what is a file descriptor in Linux?

* try to play the wargame your self but if you are ABSOLUTE beginner, follow this tutorial link:
https://youtu.be/971eZhMHQQw

ssh fd@pwnable.kr -p2222 (pw:guest)
```

# Steps

1. We first log into the remote machine using ```ssh fd@pwnable.kr -p2222```
2. After connecting, use `ls` to list out files at target. There are 3 files: `fd`, `fd.c`, `flag`
- Capturing the flag is my target, so I immediately try using `cat flag` to see what is inside that file, but it was not that easy :). I did not have the permission to open that file.
- There are two other files, `fd` and `fd.c`. I used `cat fd.c` to see the code:

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
    if(argc<2){
        printf("pass argv[1] a number\n");
        return 0;
    }
    int fd = atoi( argv[1] ) - 0x1234;
    int len = 0;
    len = read(fd, buf, 32);
    if(!strcmp("LETMEWIN\n", buf)){
        printf("good job :)\n");
        system("/bin/cat flag");
        exit(0);
    }
    printf("learn about Linux file IO\n");
    return 0;
}
```

- The program takes in one number argument `argv[1]`. That argument will later be used to decide `file descriptor fd` such that ```fd = atoi( argv[1] ) - 0x1234;```
- The program then use `read` to copy 32 bytes from `file descriptor` to buf.
- If we can fill up `buf` with `"LETMEWIN\n"`, we can capture the flag with line `system("/bin/cat flag");` being executed.
- In Linux, file descriptor 0 is standard input, so if we can set the file descriptor to 0, we can fill `buf` with `"LETMEWIN\n"`.

3. What number do we use to run the program?
- `0x1234` converted to decimal is `4660`. So we pass in `4660` in order to make `fd` 0.
- Run ```./fd 4660``` gives a line for user to put whatever input they want. We type in `LETMEWIN` and the program prints out our flag

# Result

```
good job :)
mommy! I think I know what a file descriptor is!!
```


