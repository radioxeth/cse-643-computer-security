# Week 9 Access Control

## Directory
- [Home](/README.md#table-of-contents)
- [Week 8 Repackaging Attack and Rooting Attack](/week8/README.md#week-8-repackaging-attack-and-rooting-attack)
- **&rarr;[Week 9 Access Control](/week9/README.md#week-9-access-control)**
- [Week 10 80x86 Protection Mode](/week10/README#week-10-80x86-protection-mode)

## 9.2 Access Control
([top](#directory))

Protection more so than attacks

## 9.3 UID-Based Access Control and ACL
([top](#directory))

### Access Control: Introduction

Does the subject have the ability to act on the object?

- subject
- action
  - policy
- object

Access Control Mechanisms
- Access Control List (uid-based)
- Permission-bassed Access Control 
- Capability-based Access Control

### Access Control List (UID-Based)

```
$ ls -l
drwxrwxr-x 4 seed seed 4096 sep 30 19:54 sudio ## access control list
```

```
$ getfacl system.c
$ setfacl system.c
```

## 9.4 Access Control in Android
([top](#directory))

- each app is installed as a separate user
  - user1
  - user2
  - user3
  - UID-based access control
- system resources
  - restricted to privileged users

### Isolation Among Apps

### Isolation Between App and System

- uid 6009
- uid 0 or uid 1000 access
  - system resources
  - hardware
  - os kernel

### Granularity Problem and Proxy Approach

- user
  - cannot directly access protected resources
  - user has access control to Daemon service

- Protected resources
  - root and system
  - access control by Linux

- Daemon (service)
  - privileged
  - can access protected resources
  - access control: permission-based

### Android Permissions
- when app is installed
  - declare which permissions the app needs
  - OS asks user/grants permission

## 9.5 An Example
([top](#directory))

### UID-Based vs Capability-Based Access Control

- Alice wants to store A, B, C in bank boxes
  - each box has `read` and `write`



#### UID-Based
- subject check id and passport for id

- ACL
    - A
        - Alice: rw
        - Bob: r
        - Charlie : w
    - B
    - C

#### Capability-Based Access
- key
  - some keys have `read` some keys have `write`
- anonymous 

#### Difference

- Risk Managaement
  - ID is hard to hide
  - Key can guarded
- Delegation
  - no need to reconfigure access control system

## 9.6 Capability-Based Access Controls
([top](#directory))

### Capability for File System

```c
char buf[50];
int fd=open("/etc/passwd",O_RDONLY); //rw-r--r-- 644
read(fd,buf,10);
write(fd,buf,10);

getchar();//system pauses
// now the root changes the permission of the above file to rw------- 600

read(rd,buf,10); //will be able to read - not based on ACL
// capability allows to read
// fd is the key that allows us to read, key is based on ACL
```

```c
struct fdtable{
    struct file __rcu **fd;
}

struct files_struct{
    struct fdtable __rcu *fdt
}

struct file{
    struct path f_path;    //location
    struct inode *f_inode;

    fmode_t f_mode;  // access permissions are stored here
}
```

|fdtable||
|-|-|
|0|stdin|
|1|stdout|
|2|stderr|
|3|ptr to `files_struct`|

|`files_struct`||
|-|-|
|ID of the Object||
|Permission

- `open`: create a key
    - `fdtable` manages key in the kernel
- `read(fd,...)`
- `write(fd,...)`
- `close(fd)`: destroy key

### Capability Concepts

> A capability is a token, a ticket, or key that gives the possessor permission to acess an entity or object in a computer system.

object + permission

- ticket
  - obj: movie
  - permission: can watch the move

#### Discussion Questions

##### Question 1: Can you forge a capability? Why or why not?

No. fdtable is in the kernel, the memory is protected. All we are given is an index in the fd table, which points to the capability. If user tries to use a different fd, the os will not allow that because the user lacks the permissions.

##### Question 2: Where should we store capabilities?
 - kernel

if we have to share the key, we can use encryption to share the key in the user space.

## 9.7 Capability Leaking
([top](#directory))

### Case Study: Capability Leaking

- Privileged Program
  - Downgrade into non-privileged process
  - role change like `su`
  - changing role does not automatically revoke the capability key

(see previous example)


#### Capability Leaking is OS X 10.10 (2015)

Environment variable: `DYLS_PRINT_TO_FILE` - protected file

- set-uid program
- loader: open file (key created)
- change role
- run `/bin/sh` ( can use the key )

## 9.8 Exercise: Basic Functionalities
([top](#directory))

### Basic Functionalities of Capabilities
- create
- destroy
***
Risk management
- delegation
  - revocation
- disable/enable

### Discussion

Question: Describe how you can add the "disable" and "enable" functionalities to the file-descriptor's capability mechanism. Namely, if a process disables a file-descriptor capability, the process will not be able to use the capability to access the file until the process specially enables the capability again.

Permission:
[p][e][r][m][i][s][s][i][o][n][disable/enable] additional bit in permission

bit
0: ineffective
1: effective

## 9.9 Delegation
([top](#directory))

### Delegation
- sending a file descriptor from one process to another:
  - through inhertinace: `fork()`
  - using **Unix Domain Socket**

### Unix Domain Socket

p1&rarr;*data*&rarr;p2

- IPC
  - pipe
  - Unix domain socket
    - copies fd from p1 to a new index in p2

p1&rarr;*file descriptor*&rarr;p2


## 9.10 Applications
([top](#directory))

### Capability Applications

Virus Scanner
- scan all files
- can't have root
  - reduce risk

- virus scanner (normal user)
  - send filename to root process

- root process
  - open the file
  - send the file descriptor to virus scanner

|root process||virus scanner (normal user)|
|-|-|-|
||&larr;filename&larr;||
|`open()`|||
||&rarr;file descriptor&rarr;||
|*privileged*||*risky*|


#### ACL Approach
- group: scanner
- add group to ACL of all files
  - virus scanner user added to scanner group

#### New requirement
- 5:00pm-8:00am, don't scan /xyz
  - can't use ACL
  - capability allows for this
    - between 5-8, don't grant ticket/key to normal process

### Review Question

**Question: Which access control mechanism, ACL or capability, is better regarding privilege management (enabling, disabling, discarding)? Why?

- Risk
  - capability
    - managed by subject
    - more advantages to risk management
  - ACL 
    - managed by object
    - easier to implement

## 9.11 Summary
([top](#directory))

- UID-based access control and ACL
- Access contorl in Android
- Capability-based access control
- Applications
