# bof - pwnable.kr
- Challenge Name: __bof__
- Challenge category: __pwn__
- Challenge Difficulty: __easy__
- CTF Year and Date: __2025__
- Date: __2025-05-12__

## Challenge Description
Nana told me that buffer overflow is one of the most common software vulnerability. 
Is that true?

# Information Gathering
The challenge is hosted on an server, that may be accessed by ssh:
`ssh bof@pwnable.kr -p2222 (pw: guest)`

```shell
bof@ubuntu:~$ ls -la
total 44
drwxr-x---   2 root bof   4096 Apr  3 16:04 .
drwxr-xr-x 129 root root  4096 May  6 09:27 ..
-rw-r--r--   1 root root   220 Feb 14 11:22 .bash_logout
-rw-r--r--   1 root root  3771 Feb 14 11:22 .bashrc
-rw-r--r--   1 root root   811 Apr  3 16:04 .profile
-rwxr-xr-x   1 root bof  15300 Mar 26 13:03 bof
-rw-r--r--   1 root root   342 Mar 26 13:09 bof.c
-rw-r--r--   1 root root    86 Apr  3 16:03 readme
```
Get the file information
```shell
file bof
bof: ELF 32-bit LSB pie executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=1cabd158f67491e9edb3df0219ac3a4ef165dc76, for GNU/Linux 3.2.0, not stripped
````

```shell
bof@ubuntu:~$ checksec --file bof
[!] Could not populate PLT: [Errno 12] Cannot allocate memory
[*] '/home/bof/bof'
    Arch:       i386-32-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    Stripped:   No

````

The binary has stack protection enabled (canary) has position-independent code (PIC) enabled. The binary runs properly regardless of the memory location it’s loaded at, i.e., the memory location doesn’t need to be a fixed address.

--> Run the binary in ctf-pwn container (tools/ctf/pwn)
The source code of the binary is provided in `bof.c`

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		setregid(getegid(), getegid());
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
````

The readme contains insructions on how to get the flag: 
```shell
cat readme
bof binary is running at "nc 0 9000" under bof_pwn privilege. get shell and read flag
```
## Vulnerability
The binary has a buffer overflow vulnerabilty because the gets() function performs no check on the size of the input. It will keep on accepting input until a newline is encountered irrespective of the size of the variable in which the input is being stored. This property is described as a lack of ‘bounds checking’.

## Summary of recon
- A binary with a buffer overflow vunerabilty
- The binary has stack protection enabled (canary)

# Solution/Exploit
## Task
To get the shell, we must overwrite the function local variable with the value `0xcafebabe`. This can be done with a payload that gets read by the gets() function and will overflow the `overflowne` buffer. However, we have to also takecare of the stack canary values. 

## Strategy

1. Because there is no debugger on the target system, we will compile the source code locally to be able to test the payload with the proper compile flags. We wil use the `pwn` docker container [1](#1)
2. Create `exploit.py` that interacts with the bynary and sends the payload using pwntools
3. Test the exploit

## Compile the source
We will 

# Conclusion
# References
1. <a name="1"></a> [pwn](/tools/pwn) Docker image to run the binary locally 
2. <a name="2"></a> [pwntools]() 
