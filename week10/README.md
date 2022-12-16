# Week 10 80x86 Protection Mode

## Directory
- [Home](/README.md#table-of-contents)
- [Week 9 Access Control](/week9/README.md#week-9-access-control)
- **&rarr;[Week 10 80x86 Protection Mode](/week10/README#week-10-80x86-protection-mode)**

## 10.2 80x86 Protection Mode
([top])(#directory)

## 10.3 Why We Need Access Control in CPU
([top])(#directory)

Why do we need access control in CPU?
- password (euid=0)
  - c1
  - ...
  - c99 cpu allow
  - cn

- copy password (euid=5000)
  - c1
  - ...
  - c99 cpu failure
  - cn

in 8086~80286
  - real mode
in 80286 ~
  - protection mode
    - cpu-level/hardware security
  - TCP: Trust Computing Base

These days
  - have further protection in hardware
  - Hardware Security
    - Intel SGX
    - ARM TrustZone

### Access Control Overview

|subject|&rarr;|object|
|-|-|-|
||CPU level Access Control||
|||kernel memory|
|||register|
|||I/O Devices|
||Instruction||

## 10.4 Exercise: Memory Isolation
([top](#directory))

### Virtual Memeory


#### Older Generations
- p1
  - `mov 0x8000 eax`
  - use the real, physical address
- where do we store the access control??
- access control hard

|physical memeory|
|-|
|0x8000|


#### Newer Generations

|p1||
|-|-|
|///|Translate to physical memory|

|physical||
|-|-|
|p1 memory space||
|p2 memory space||

|p2||
|-|-|
|///|Translate to physical memory|

### Logical Address to Linear Address

<img src="/week10/images/addressTranslation.png" width=500>

- capability-based access control

### Segment Descriptor

32-bit machine
|31-24|23-20|19-16|15-8|7-0|
|-|-|-|-|-|
|base 31:24|23:20|seg limit 19:16|Access Rights [P][DPL][S][Type]|Base: 23:16|

|31-16|15-0|
|-|-|
|Base Address 15:00|Segment Limit 15:00|

### Descriptor Table

<img src="/week10/images/descriptorTable.png" width=500>

- GDT Global Descriptor Table
- LDT Local Descriptor Table

- register protection
  - privilege mode

#### Question 1
- If a program is copying data to a buffer located toward the end of a segment, is it possible to overflow the segment as the result of buffer overflow? Please explain.

## 10.5 Rings
([top](#directory))

### Rings and Privilege Level
- GDT issues many tickets?


rings 
  - 0 
  - 1 
  - 2
  - 3

0 most privileged
3 least privileged

- kernel runs in ring 0
- user program runs in ring 3

How do we mark the rings?
Descriptor
|||[][] 2 ring label bits|
|-|-|-|
|||DPL|

- DPL: Descriptor Privilege Level
- CPL: Current Privilege Level (ring)

|ACL|WHAT?|
|-|-|
|subject|CPL Current Privilege Level|
|action|instruction|
|object|register/memory|

### Data Access

|rings||
|-|-|
|0|&larr;|
|1||
|2||
|3||

- CPL: 0
- DPL: 0,1,2,3
- Allow
***

|rings||
|-|-|
|0||
|1||
|2||
|3|&larr;|

- CPL: 3
- DPL: 0,1,2
- Deny

### Code Access

- can only access code in the same ring

|rings||
|-|-|
|0|&larr;|
|1||
|2||
|3||

- CPL: 0
- DPL: 0
- Allow

### Register Access
- General Register
  - `eax`
  - `ebx`
  - `ecx`
  - `cs`
  - `ds`
  - `ebp`

- Special Purpose Register
  - Ring 0 only
    - `GDTR`
    - `LDTR`
    - `PTBR` page table base register (CR3)

### IO Access
- EFLAGS Register
- IOPL - 0 (3 is userspace)
  - restricted to process running in ring 0

## 10.6 System Calls
([top](#directory))

### Needs for Cross-Ring Invocation


|||
|-|-|
|ring 3||
|X not allowed but needed|&darr;|
|ring 0|disk IO|

ring 0 allows for doors in to ring
- each door has access control
- these are **system calls**

### System Calls

- jummp address not allowed

#### Exception

|ring 3|&rarr;|ring 0|
|-|-|-|
||[exception `int 0x80`]||
||[system call #]||

|Exception Table||
|-|-|
|0|entry point|
|1||
|2||

#### SYSENTER/SYSEXIT (index #)

|ring 3|&rarr;|&larr;|ring 0|
|-|-|-|-|
||[SYSENTER `index`]||
|||[SYSEXIT `index`]||

|System Index Table||
|-|-|
|0|entry point|
|1||
|2||

#### Gate
- call address

<img src="/week10/images/callGates.png" width=500>

## 10.7 Paging
([top](#directory))

<img src="/week10/images/paging.png" width=500>

<img src="/week10/images/pagingAccessControl.png" width=500>

- page-table entry
  - r/w bit
    - 0 read only
    - 1 read/write
  - u/s bit
    - 0 supervisor level
    - 1 user level
  - kernel

- if CPL = 0, 1, 2
  - supervisor level

## 10.8 Review
([top](#directory))

Back to Initial Question

- password (euid=0)
  - in ring 3
    - c1
    - c2
  - go through gate
  - in ring 0
    - ...
    - c99 cpu allow
  - cn

- copy password (euid=5000)
  - c1
  - ...
  - c99 cpu failure
  - cn

## 10.9 80x86 Protection Mode Questions
([top](#directory))

- Why can't a program directly write to the kernel memeory? What if the program is running with the root privilege?


|program||kernel|
|-|-|-|
|ring3|&rarr;X|ring0|
|cpl=3|&rarr;X|dpl=0|

same with root privilege

|program||gate|kernel|
|-|-|-|-|
|ring3|&rarr;|uid==0? OK|ring0|
|cpl=3|&rarr;|uid==0? OK|dpl=0|


- What are the differences between system calls and library calls?

|system||library|
|-|-|-|
|`open()`||`printf()`|
|ring 3 calls into ring 0||ring 3 calls into ring 3|
|privilege has changed||function call|

## 10.10 Summary
([top](#directory))

- 80x86 protection mode
  - (case study)
- Memory protection
- Rings
- How OS depends on the protection mode
