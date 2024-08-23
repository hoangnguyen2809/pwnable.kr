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

3. Calculating what to pass in as argument
- First time approaching this challenge, I used python to convert `0x21DD09EC` to decimal `568134124`
   - ````568134124 / 5```` is `113626824.8`. So ```568134124 = 113626824 * 5 + 4```
   - Converting ```113626824 * 5 + 4``` to hex we have ```0x6c5cec8 *5 + 0x4```
   - Reverse the order for little endian, we have ```\xc8\xce\xc5\x06 * 5 + \x04```
   - Use that as input for `col`: ```./col `python - c "print '\xc8\xce\xc5\x06'*5+'\x04'"` ```
   
   - Unfortunately, I did not get the result I wanted:
        ```
        col@pwnable:~$ ./col `python - c "print '\xc8\xce\xc5\x06'*5+'\x04'"`
        passcode length should be 20 bytes
        ```
     - I realized that, I was passing in 21 bytes instead of 20 bytes. So this way did not work as intended.
    
- So, the right way to do this is ```568134124 = 113626824 * 4 + 113626828```
- We now have ```\xc8\xce\xc5\x06 * 5 + \xcc\xce\xc5\x06```
- Use that as input for `col`:  ```./col `python -c "print '\xc8\xce\xc5\x06'*4+'\xcc\xce\xc5\x06'"` ```

# Result 
```
daddy! I just managed to create a hash collision :)
```
