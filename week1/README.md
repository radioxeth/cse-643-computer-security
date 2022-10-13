# Week 1 Set-UID Programs

## Directory
- [Home](/README.md#table-of-contents)
- **&rarr;[Week 1 Set-UID Programs](/week1/README.md#Week-1-set-uid-programs)**
- [Week 2 Buffer-Overflow Attacks and Countermeasures](/week2/README.md#Week-2-buffer-overflow-attacks-and-countermeasures)

## 1.2
([top](#directory))

## 1.3 Unix Security Basics
([top](#directory))

### Unix Security Basics
- user
- group
- permissions
- access conrol 

### User and Group

#### User
- login
  - *seed*
    - converted to number 2000
    - user id (UID)
  - root
    - UID=0

#### Group
- xyz group
  - users seed, alice, bob

### User and Group Files

`/etc/passwd`

`/etc/group`

#### Permissions

```
$ ls -l
-rwxrwxr-x  # file starts with -
drwxr-xr-x  # directory starts with d
```

r: read
w: write
x: executable

chmod [775] file

|r|-|x|
|-|-|-|
|1|0|1|
binary=5

-rwx|rwx|r-x (775)
- -rwx
  - owner
- rwx
  - primary group
- r-x
  - others

### The Sudo Command
#### Run the sudo command

```
seed@ubuntu:$ head /etc/shadow
head: cannot open `/etc/shadow` for reading: Permission denied
seed@ubuntu:$ sudo head /etc/shadow
[sudo] password for seed:
...
```

#### The /etc/sudoer file


## 1.4 The Need for Privileged Programs
([top](#directory))

### Password Dilema: How to Change Password?

```
seed@ubuntu:~$ ls -l /etc/shadow
-rw-r----- l root shadow 1320 Jan 9 2014 /etc/shadow password
```
how do we change the password?

- solution 1
  - user becomes root
    - bad idea, can change anyone's password
- solution 2
  - sudo
    - bad idea, same as granting root privilege
- solution 3
  - call the root
    - background program
- solution 4
  - set-uid program
    - passwd: privileged

## 1.5 Exercise: Set-UID Privileged Programs
([top](#directory))

key point: restricted behavior for sudo users

### How Set-UID Programs Work

user:
normal &rarr; root

Program &rarr; OS creates a process
- process
  - uid (seed)
- set-uid
  - effective uid: owner of the program
    - owner is root usually
    - restricted behavior
  - real uid: seed

### Turn a Program Into a Set-UID Program

| 1 _ _ | rwx | r-x | r-x |
|-|-|-|-|
| set-uid bit, set-gid bit| | |

|1|-|-|
|-|-|-|
|set-uid|set-gid| |

`chmod 4755` **root user**

### Exercise

> Somebody gives you a chance to use their Unix account, and you have your own account on the same system. Can you take over this person's account in 10 seconds?

create a cloak shell program
behavior of shell is unrestricted
```
$ cp /bin/sh /tmp/mysh
$ chmod 4777 /tmp/mysh
```

## 1.6 What Can Go Wrong in a Program?
([top](#directory))

## 1.7 Attack Surfaces
([top](#directory))

### Risk Analysis: Attack Surface

- set-uid program attack surface
  - user input
  - environment variables (controlable by user)
  - system input (controlable by user)

## 1.8 Attacks via Environment Variables Part 1
([top](#directory))

### PATH Environment Variables

```C
#include <stdlib.h>

int main(){
    system("cal")
}
```
- setuid root program
- system runs a shell program with root privilege
  - `/bin/sh cal`
  - `$ cal`
  - uses the PATH to find the program
    - PATH = list of directories
- change PATH env variable
  - create own program `cal`
  - `PATH=[dir]:$PATH`
    - find your `cal` progam first and execute

### IFS Attacks

```C
#include <stdlib.h>

int main(){
    system("/bin/cal")
}
```
- still run shell
  - provide direct path

```
IFS="/a"
```
```
_bin_c_l
```
now we get back to path because bincl does not exist.

## 1.9 Attacks via Environment Variables Part 2
([top](#directory))

### What is a Dynamic-Link library?

- static-link or dynamic-link
  - Compiler defaults to dynamic-link

### Shared Library

#### The idd command

name
  - ldd - print shared library dependencies
synopsis
  - ldd prints the shared libraries requred by each program or shared library speicified on the command line

#### Run Idd on a binary

```c
void main(){
    printf("Hello World\n");
}
```

```
$ ldd a.out
```

### LD_PRELOAD

#### How LD_PRELOAD affects Dynamic-Linked library

```c
void main(){
    printf("Hello World\n");
    sleep(2);
}
```
sleep.c
```c
#include <stdio.h>
void sleep(int s){
    printf("I am not sleeping!\n");
}
```

```
$ unset LD_PRELOAD
$ ldd a.out
```

### How LD_PRELOAD Affects Set-UID programs

#### Experiment
```
$ cp /usr/bin/env ./myenv
$ sudo chown root myenv
$ sudo chmod 4755 my env #root owned set-uid
$ ls -l my env
-rwsr-xr-x 1 root seed 22060 Date myenv
```

#### Difference
```
$ export LD_PRELOAD=./libmylib.so.1.0.1
$ export LD_LIBRARY_PATH=.
$ export LD_MYOWN="my own value"
$ env | grep LD
LD_PRELOAD=./libmylib.so.1.0.1
LD_LIBRARY_PATH=.
LD_MYOWN=my own value
$ myenv | grep LD_
LD_MYOWN=my own value
```

**OS removes LD_\* env variables**

## 1.10 Attacks via Explicit Inputs
([top](#directory))

### Attacks via Explicit User Inputs

catall.c
```C
#include <string.h>
#include <studio.h>
#include <stdlib.h>

int main(int argc, char *argv[]){
    char *cat = "/bin/cat";

    if(argc < 2){
        printf("please type a file name \n");
        return 1;
    }
    char *command = malloc(strlen(cat)+strlen(argv[1])+2);
    sprintf(command, "%s %s", cat, argv[1]);
    system(command);
    return 0;
}
```


```
$ catall "aa;/bin/sh"
/bin/cat: aa: No such file or directory
# id
uid=1000(seed), gid=1000(seed), euid=0(root)
```

- `;` denotes the end of a command and the begining on of another

### Secure way to Invoke External Programs

safecatall.c
```C
#include <string.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char*argv[]){
    char *v[3];

    if(argc < 2){
        printf("please type a file name\n");
        return 1;
    }
    v[0] = "/bin/cat"; v=[1]=argv[1]; v[2]=0;
    execve(v[0],v,0);

    return 0;
}
```

```
$ safecatall "aa;/bin/sh"
/bin/cat: aa: No such file or directory
$
```

## 1.11 Capability Leaking

([top](#directory))

```C
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>

void main(){
  int fd;
  char *v[2];


// assume that the file is an important system file and it is owned by root with permission 0644
  fd = open("/etc/zzz",O_RDWR|APPEND);

  if(fd==1){
    printf("Cannot open /etc/zzz\n");
    exit(0);
  }

  printf("fdis %d\n",fd);
//permanently disable the privilege yb making the effective uid the same as the real uid
  setuid(getuid());

  //execute shell
  v[0]="/bin/sh";v[1]=0;
  execve(v[0],v,0);
}

```

we never closed the file! Now we have the key of the root owner

### Capbility Leaking in OS X 10.10 (2015)

```
$ DYLD_PRINT_TO_FILE=/this_system_is_vulnerable su <some_username>

```

## 1.12 Server Approach vs Set-UID

([top](#directory))

### Comparisons

- Discussion: Compare the Set-UID approach with the server approach
  - Daemom/Server (root)
    - user sends request to running process
  - Set-uid
    - program (no idle processes)

- which one is safer?
- attack surfaces
  - user input
    - both are the same
  - environment variables
    - daemon
      - parent process (root system, trusted)
      - child process (daemon), env variables trusted
    - set-uid
      - parent process (user, not trusted)
      - child process (set-uid), env variables not trusted
  - system input

What is my attack surface?

## 1.13 Summary

([top](#directory))

- why we need privileged programs
- how set-uid programs work
- how can privileged program go wrong

# Week 1 Live Session

## Malicious Software
