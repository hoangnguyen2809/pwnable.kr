# Toddler's Bottle -- collision

# Challenge description
```
Daddy told me about cool MD5 hash collision today.
I wanna do something like that too!

ssh col@pwnable.kr -p2222 (pw:guest)
```

# Steps
1. We first log into the remote machine using ```ssh col@pwnable.kr -p2222```
2. After connecting, use `ls` to list out files at target. There are 3 files: `col`, `col.c`, `flag`
- Learning from the first challenge `fd`, I immediately use ```cat col.c``` to see what's inside the c file instead of trying to explore the flag
```
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
        int* ip = (int*)p;
        int i;
        int res=0;
        for(i=0; i<5; i++){
                res += ip[i];
        }
        return res;
}

int main(int argc, char* argv[]){
        if(argc<2){
                printf("usage : %s [passcode]\n", argv[0]);
                return 0;
        }
        if(strlen(argv[1]) != 20){
                printf("passcode length should be 20 bytes\n");
                return 0;
        }

        if(hashcode == check_password( argv[1] )){
                system("/bin/cat flag");
                return 0;
        }
        else
                printf("wrong passcode.\n");
        return 0;
}
```

- The program takes 1 argument of 20 bytes, then check if it is similar to the hard coded hashcode `0x21DD09EC`. If it is, we can get to the flag.
- The program check our arg using `check_password` function. The loop processes 5 integers (each 4 bytes), so the function essentially takes the input string, 
interprets it as an array of 5 integers, and computes the sum of these integers.

3. First time approaching this challenge, I used python to divide `0x21DD09EC` into 5, and got 
