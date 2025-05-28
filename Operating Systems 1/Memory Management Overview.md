[[cse120/Operating Systems 1/Operating Systems Home]], [[cse120/Operating Systems 1/Paging]]

| Term                   | Description                                                                                                                                                                                                                                               |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Virtual Address**    | The address used by process. It is part of the virtual address space and does not correspond directly to a physical location until translated.                                                                                                            |
| ***Virtual Page(s)***  | The processes memory(aka a ***bunch of virtual addresses***, it is split into fixed page size chunks. The memory within a page is just a ***range of virtual addresses*** which can be decoded via the page table into the corresponding physical memory. |
| **Physical Address**   | Where the data actually resides in RAM (or disk).                                                                                                                                                                                                         |
| ***Physical Page(s)*** | A fixed-size block in physical memory (RAM), (aka a ***fixed range of physical addresses***). It is the real backing store for a virtual page once it is mapped by the OS. Data from a virtual page is actually stored here after address translation.    |
| **Page Table**         | Data structure maintained by the OS that maps virtual pages to physical page frames.                                                                                                                                                                      |
| ***PTE***              | Entries in Page Table, contains metadata like valid bit, physical frame number, access permissions, dirty bit, etc., for a specific virtual page.                                                                                                         |

TLDR
- [[#Basic Process Address Space]]
- [[#Overview]] - the big picture here - How the memory of a process gets managed!
	- [[#Challenges]]
	- [[#Goals]] - multitasking, transparency, protection, efficiency
		- [[#Multitasking with Static Relocation]]
		- [[#Dynamic Memory Relocation]]
- [[#Virtual Memory]] - abstraction that OS provides for managing memory. Virtual memory is just addresses to map to physical memory which is where the actual data is stored
	- [[#Address Translation]] - Goes over multiple methods to store translations for virtual to physical
		- [[#Base and Bound]] - old way
		- [[#Evolution Of Memory-Management Techniques]]
		- [[#Segmentation]] - store your translations in segments (vary in size)
			- [[#Address Translation]]
			- [[#Tradeoffs]]
		- [[#Paging]] - store your translations in page tables (all fixed sizes)
			- [[#Address Translation]]
				- [[#Page Tables]]
			- [[#Advantages and Limitations]]
			- More covered in [[cse120/Operating Systems 1/Paging]] - in depth on Page table entries
			- More covered in [[cse120/Operating Systems 1/TLBs and Swapping]] - in depth on caching and swapping actual page files
			- More covered in [[cse120/Operating Systems 1/Page Replacement and Memory Allocation]] - in depth on replacing pages 

## Basic Process Address Space
- Reminder
![[Screenshot 2025-05-06 at 3.43.08 PM.png|450]]


## Overview
Goals of memory management:
- multitasking, transparency, isolation, efficiency
Mechanisms to achieve goals:
- Physical and virtual addressing
- Segmentation and paging
- page table management, TLB, VM tricks

#### Challenges
![[Screenshot 2025-05-06 at 3.45.33 PM.png|450]]

***How it worked in the early days***
![[Screenshot 2025-05-06 at 3.46.09 PM.png|450]]
***Limitations Include***
- Slow, only runs one program at a time.
- Potentially lots of overhead due to loading and storing a ton of memory
- Process can read or write to the OS's memory if it is malicious or has ton of memory
- What if a process runs forever??? -> not a memory management problem. this is a scheduling problem [[cse120/Operating Systems 1/CPU Scheduling]]
#### Goals
![[Screenshot 2025-05-06 at 3.50.09 PM.png|550]]

##### Multitasking with Static Relocation
![[Screenshot 2025-05-06 at 3.53.38 PM.png|550]]
***Limitations / Drawbacks***
- No protection, still uses physical addresses. OS memory can still be tampered with
- Wasteful to be rewriting the memory on every new process
- Low memory utilization
	- Addresses are fixed after loading
	- Cannot relocate at runtime to fill holes
- No sharing
	- One segment per process
	- Cannot share parts of the process address space
- Entire address space needs to fit in memory
![[Screenshot 2025-05-06 at 3.55.16 PM.png|250]]

##### Dynamic Memory Relocation
Everything referenced in your program are virtual addresses(stack, heap, etc.) , they get all translated to physical addresses . refer to [[#Virtual Memory]]
![[Screenshot 2025-05-06 at 3.58.23 PM.png|550]]

## Virtual Memory

![[Screenshot 2025-05-06 at 4.02.15 PM.png|450]]
- Gives the illusion that the process is the only 1 that has full control of CPU
- 64 bit address space is typically used nowadays, meaning pointer is 8 bytes wide and program and reference up to 2^64 bytes of data
- Virtual address space is larger so that it can store references to data on disk I think
***KEY POINT : VIRTUAL MEMORY IS USUALLY LARGER THAN PHYSICAL MEMORY***

***TLDR***
- ***Virtual memory is the illusion; physical memory is the reality.***
- ***Virtual memory is like a pointer, except we cannot use the "pointer" / address until we have translated it***

***Virtual Memory*** - The illusion, what the process sees is just a bunch of virtual addresses. These addresses when accessed, get translated and then you can access the data from RAM . Each process gets a limited amount of GB's but it thinks that it has "access" to all pages within the virtual memory, however this is not true. 
The actual data is stored in physical memory, virtual memory is just ***addresses*** that you use to map to the real data in physical memory!

***Physical Memory(RAM)*** - Small, fast memory where only active pages are loaded. This is what the "hardware knows" and works with,.

***Disk*** - The backing store - pages not in RAM are here. When a page fault occurs, the OS may fetch a page from disk into RAM, potentially evicting another using a page replacement policy.

How it works:
- A process is launched → gets a full virtual address space (say 4 GB)
- Only some of that space is actually loaded into physical memory (RAM)
- The rest is left unallocated, swapped out, or loaded on-demand
- If a process tries to access a page not in RAM → page fault → bring it in from disk → might evict something else using a replacement policy

![[Screenshot 2025-05-06 at 4.07.29 PM.png]]

### Address Translation
Mapping virtual to physical addresses.
![[Screenshot 2025-05-06 at 4.08.27 PM.png]]

#### Base and Bound
![[Screenshot 2025-05-06 at 4.10.10 PM.png]]
***Limitations***
- Not dynamic, the segment is decided when loading the process. Cannot dynamically change after that.
- Fragmentation - hard to use memory efficiently
	- External fragmentation: wasted memory between segments
	- Internal fragmentation: wasted memory inside segments
- Cannot share memory between processes
#### Segmentation
![[Screenshot 2025-05-06 at 4.18.22 PM.png|450]]

##### Address Translation
![[Screenshot 2025-05-06 at 4.19.15 PM.png|450]]
- Virtual address has segment and offset sections within the bits
	- Segment section to determine which section/process to jump to
	- Offset in the diagram is to determine which segment to jump to within the segment itself

###### Example
![[Screenshot 2025-05-06 at 4.22.12 PM.png]]

##### Tradeoffs
![[Screenshot 2025-05-06 at 4.22.55 PM.png]]
***Limitations***:
- Still the problem of Fragmentation - hard to use memory efficiently
	- External fragmentation: wasted memory between segments
	- Internal fragmentation: wasted memory inside of segments 
- Large segment tables can be complex to manage

## Paging
![[Screenshot 2025-05-06 at 4.28.49 PM.png|455]]
Virtual pages represent the process's address space. To access one, the system consults the page table using the virtual page number. The page table entry tells us whether the page is valid and, if so, where the corresponding physical frame is, along with other metadata like permissions.

##### Address Translation
![[Screenshot 2025-05-06 at 4.31.28 PM.png]]
- With paging, the pages are all 1 fixed size, so sort of a simplification of the [[#Segmentation]] approach where in that the sizes can vary I think with the segments in each process
- With the virtual address we have the bits needed to determine the VPN and the offset
	- Bits to determine which page to jump to
	- Bits to determine where to jump to within the page

###### Page Tables
![[Screenshot 2025-05-08 at 2.14.06 PM.png|450]]
![[Screenshot 2025-05-06 at 4.36.37 PM.png|450]]

***Example***
![[Screenshot 2025-05-06 at 4.40.02 PM.png|450]]

##### Advantages and Limitations
![[Screenshot 2025-05-06 at 4.42.05 PM.png|450]]
![[Screenshot 2025-05-06 at 4.42.41 PM.png|450]]