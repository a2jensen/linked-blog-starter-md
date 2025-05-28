- [[#Problem 1]] - Virtual Page to Physical Page mappings
- [[#Problem 2]] - TLB
- [[#Problem 3]] - Bit allocation for VPN
- [[#Problem 4]] - Calculating offset and page numbers
- [[#Problem 5]] - Average memory access accounting page faults
- [[#Question 6|Problem 6]] - Page replacement - FIFO and LRU
- [[#Problem 7]] - Actions causing faults
- [[#Problem 8]] - Demand - paging
## Problem 1
[[cse120/Operating Systems 1/ExamCode/Nachos VM Worksheet]]

***Notes***
- `Process.PageSize = 128 bytes`
- `Virtual Address Space - 4 pages`
- `Physical Memory - 8 pages`

- `UserProcess.numPages = 4` 
- `Processor.numPhysPages = 8`

# Q1 - Q7

- What are the values of the UserProcess.pageTable mappings?
```
- pageTable[0].vpn = 0
- pageTable[0].ppn = 2
- pageTable[1].vpn = 1
- pageTable[1].ppn = 0
- pageTable[2].vpn = 2
- pageTable[2].ppn = 6
- pageTable[3].vpn = 3
- pageTable[3].ppn = 4
```

- 256 represented in binary
- `0x9`

- 768 represented in binary
- `0x11`

- Page 6

- Offset = 42

- 768 + 42 = 810


![[Screenshot 2025-05-16 at 11.51.08 AM.png]]
## Q8

- System.arraycopy just moves bytes between two Java arrays.
- ***In readVirtualMemory and writeVirtualMemory, it's used to copy data between a Java buffer and the machine's physical memory array.***
- It fails when the virtual address range spans multiple pages, because those pages are not guaranteed to be adjacent in physical memory.

So when we are reading / writing bytes from virtual memory, those bytes can ***POSSIBLY*** map to different blocks of memory within the physical address space, so they are not adjacent with each other. In the current naive implementation, the `system.arraycopy` only assumes that the mappings in physical memory are all adjacent.

So in the given example when we are reading `vaddr 0x7E which is 126 bytes with data length = 4(meaning we are reading 4 bytes starting at 126 bytes in virtual address space`,  from virtual memory and grabbing the physical address location via the the virtual page number, we run into an error.

Walkthrough of ***correct logic***:
- We call `arrayCopy`, in which we read through 4 bytes starting at `vaddr 0x7E`, grabbing the physical memory data that is being held there
- Start at address `0x7E` in the virtual address space, which is `126 bytes` in to page 1,
	- We read the first byte(126) which is in `vpn 0`, and that maps to `ppn 2`
	- We read the second byte (127) in `vpn 0` , and that maps to `ppn 2`
	- We read the third byte (128) which were now in `vpn 1`, and it maps to `ppn 0`
	- We read the last byte (129), inside `vpn 1` and that maps to `ppn 0`

However, the problem with the current implementation is that it's going to map the 4 bytes of physical addresses into 1 adjacent block, which is not correct since the physical addresses being held at those bytes are not adjacent with each other. ***SO A PORTION OF THE PHYSICAL MEMORY BEING GRABBED WOULD BE WRONG***

Walkthrough of ***incorrect logic***
- We call `arrayCopy`, in which we read through 4 bytes starting at `vaddr 0x7E`, grabbing the physical memory data that is being held there
- Start at address `0x7E` in the virtual address space, which is `126 bytes` in to page 1,
	- We read the first byte(126) which is in `vpn 0`, and that maps to `ppn 2`
	- We read the second byte (127) in `vpn 0` , and that maps to `ppn 2`
	- We read the third byte (128) which were now in `vpn 1`, and it maps to `ppn 2` since `arraycopy` assumes that the physical memory is continuous
	- We read the last byte (129), inside `vpn 1` and that maps to `ppn 2` since `arraycopy` assumes that the physical memory is continuous
## Problem 2
When using physical addresses directly, there is no virtual to physical translation overhead. Assume it takes 100 nanoseconds to make a memory reference. If we used physical addresses directly, then all memory references will take 100 nanoseconds each.

#### P 1
If we use virtual addresses with page tables to do the translation, then without a TLB we must first access the page table to get the approprate page table entry (PTE) for translating an address, do the translation, and then make a memory reference. Assume it also takes 100 nanoseconds to access the page table and do the translation. In this scheme, what is the effective memory reference time (time to access the page table + time to make the memory reference)?

***Answer***
200 nanoseconds - 100 to access page table and translate via PTE, then 100 nanoseconds to make the memory reference.

#### P 2
If we use a TLB, PTEs will be cached so that translation can happen as part of referencing memory. But, TLBs are very limited in size and cannot hold all PTEs, so not all memory references will hit in the TLB. Assume translation using the TLB adds no extra time and the TLB hit rate is 75%. What is the effective average memory reference time with this TLB?

***Answer***
We assume that no extra added time if we miss in the TLB, however if we do hit then it would be -100 nanoseconds due to us skipping the step of searching for the PTE in the page table. Given that hits happen 75% of the time, the average memory reference with this type of TLB is:

`(100 nanoseconds for physical memory access)` times `1 - (1 * 0.75)` = `125 nanoseconds/avg`

why `1 - (1-0.75)` -> `(1-0.75)` represents the chance of a hit happening, so we want to take the complement of that in order to find the chance of a miss happening. Putting this into an analogy, if we have 100 TLB accesses, 75 of those would hit, adding no nanoseconds, while 25 would add 75 nanoseconds.

For those 100 accesses, theres a possibility of `10000` nanoseconds max, given that 25 only miss that is `2500` seconds added. So simply divide `2500 / 100000` to get `0.25`
***The answer is 125 nanoseconds on average***
#### P 3
If we use a TLB that has a 99.5% hit rate, what is the effective average memory reference time now? (This hit rate is close to what TLBs typically achieve in practice.)

`50 / 100000` = `0.005 nanoseconds average`

***The answer is 100.005 nanoseconds on average***


## Problem 3
Consider a 32-bit system (both virtual and physical addresses are 32 bits) with 1K pages (pages of size 1024 bytes) and simple single-level paging.

***For context, a virtual address is split into two parts, offset which determines the specific byte within a VPN, and the VPN***

***Quick mixup i had so here's this clarification, which bits address where the physical page frame is? That purpose is delegated to the Page Table Entry(PTE), remember that PTE is an address which contains the needed bits to identify where the physical memory in the corresponding page is stored(PFN) + metadata bits***

a. With 1K pages (pages of size 1K), the offset is 10 bits. How many bits are in the virtual page number (VPN)?

***Answer***
We are given the following info
- Virtual address size: 32 bits
- Page size : 1024 bytes = 1 KB = 2^10 bytes -> offset = 10 bits
- Paging scheme is single-level paging
Since it is simply single level, the rest of the bits determines the VPN : 32 - 20 = `12`

b. For a virtual address of 0xFFFF, what is the virtual page number?

***Answer***
This isn hex - which mean's it is 16 bits total
`0x 1111 1111 1111 1111`
Interpret this as a 32 bit address by padding out zeros
`0x 0000 0000 0000 0000 1111 1111 1111 1111`
Get the top 22 bits: bits 31-10 (reading from left to right)
`0x 0000 0000 0000 0000 1111 11`
Shift it right by 10 bits, dropping the offset bits, we are left with
`0x 0000 0000 0000 0000 00000 0000 0011 1111`
- converting this to decimal, we get ***63*** as our VPN.

c. For a virtual address of 0xFFFF, what is the value of the offset?
Take the last 10 bits, and simply convert it to decimal.
`11 1111 1111` = ***1023*** is our offset.

d. What is the physical address of the base of physical page number 0x4?
***Key formula***
***physical address = Page Frame Number(PFN) x Page Size

***Answer***
Physical address = 4 times 1024 = 4096
4096 in decimal is 0x1000

e. If the virtual page for 0xFFFF is mapped to physical page number 0x4, what is the physical address corresponding to the virtual address 0xFFFF?

***Answer***
So we take 4096(0x1000) and add the offset of 1023(0x03FF) which we get 5119(0x13FF)

## Problem 4
Suppose we have a computer system with a 44-bit virtual address, page size of 64K, and 4 bytes per page table entry.

### part A
How many pages are in a virtual address space? (Express using exponentiation.)
***Answer***
- page size is 64k = 1024 bytes times 64 = 65536
	- Looking at this 2^16 = 65336, so 16 bits are reserved for the offset out of the 44 bits
	- There can possibly be 2^28 or 268435456 pages within the virtual address space.

### part B
Every process has its own page table, and every page table has a root page and many secondary pages. Suppose we arrange for each kind of page table page (root and secondary) to have the size of a single page frame. How will the bits of the address be divided up among the offset, index into a secondary page, and index into the root page?

***Answer***
16 bits for offset, then take the remaining 28 and split it into 2 -> 14 for root page and 14 for secondary index


### part C
[[Bytes Sizes Breakdown]]
Suppose we have a 4 GB program such that the entire program and all necessary page tables (using two-level pages from above) are in memory. (Note: It will be a lot of memory.) How much memory, in page frames, is used by the program, including its page tables?
- Asking how much memory total for allocated page frames/pages

***Answer: 65,540 page frames***
Information we know
- Virtual address size = 44 bits
- Page size = 64 KB = 2^16 bytes
- PTE size = 4 bytes
- One page frame = one page = 64 KB

*Finding memory for page frames*
- Each page uses 2^16 bytes = 65,536 data page frames
*Finding memory for page tables*
- 2^16 bytes per page / 4 bytes per entry = 2^14 pages
- Take that and then 2^16 data pages / 2^14 per second level table = 2^2 = 4 secondary page tables

Thus the total is then *65,540*

## Problem 5
Suppose we have an average of one page fault every 20,000,000 instructions, a normal instruction takes 2 nanoseconds, and a page fault causes the instruction to take an additional 10 milliseconds. What is the average instruction time, taking page faults into account? Redo the calculation assuming that a normal instruction takes 1 nanosecond instead of 2 nanoseconds.

- Given that 20,000,000 instructions = 20,000,000 nanoseconds of execution
- 1 of those executions adds a total of 10 milliseconds instead of just 1 nanosecond
	- 1 milliseconds = 1 000 000 nanoseconds
		- So 10 milliseconds = 10 000 000 nanoseconds

19 999 999 instructions run in 2 ns = 19 999 999 nanoseconds
1 instruction run in 1 ms = 10 000 000 nanoseconds, given this it adds up to:
19 999 999 + 10 000 000 = 29 999 999 nanoseconds
`29 999 999 nanoseconds / 20 000 000 instructions = 1.49999`
Average instruction time is therefore ***1.49*** seconds!


## Question 6
[[Classes/Operating Systems/Operating Systems Home]], [[Classes/Operating Systems/Page Replacement and Memory Allocation#First-In First-Out (FIFO)]]

If FIFO page replacement is used with four page frames and eight pages (numbered 0–7), how many page faults will occur with the reference pattern 427253323126 if the four frames are initially empty? Which pages are in memory at the end of the references? Repeat this problem for LRU.

Key idea, we evict the oldest page that was added in for FIFO
### FIFO
- Add in 4, 2, 7 - OLDEST : 4 2 7
	- 3 page frames taken up
- 2 - this is a hit - OLDEST 4 7 2
- Add in 5 - all page frames are taken up - OLDEST 4 7 2 5
- Attempt to add in 3 - no space so we evict 4 - OLDEST 7 2 5 3
- Add in 3 - this is a hit - OLDEST : 7 2 5 3
- 2 - this is a hit - OLDEST : 7 5 3 2
- 3 - this is a hit OLDEST : 7 5 2 3
- 1 - this is not a hit - no space so we evict 7, OLDEST: 5 2 3 1
- 2 - this is a hit OLDEST : 5 3 1 2
- 6 - this is not a hit, evict 5 -> OLDEST : 3 1 2 6
The pages in memory at the end are `3 1 2 6`

| Step | Page | Memory State | Page Fault? | Note           |
| ---- | ---- | ------------ | ----------- | -------------- |
| 1    | 4    | 4            | Yes         | Add 4          |
| 2    | 2    | 4 2          | Yes         | Add 2          |
| 3    | 7    | 4 2 7        | Yes         | Add 7          |
| 4    | 2    | 4 2 7        | No          | Hit            |
| 5    | 5    | 4 2 7 5      | Yes         | Add 5          |
| 6    | 3    | 2 7 5 3      | Yes         | Evict 4, add 3 |
| 7    | 3    | 2 7 5 3      | No          | Hit            |
| 8    | 2    | 2 7 5 3      | No          | Hit            |
| 9    | 3    | 2 7 5 3      | No          | Hit            |
| 10   | 1    | 7 5 3 1      | Yes         | Evict 2, add 1 |
| 11   | 2    | 5 3 1 2      | Yes         | Evict 7, add 2 |
| 12   | 6    | 3 1 2 6      | Yes         | Evict 5, add 6 |

### LRU
[[Classes/Operating Systems/Page Replacement and Memory Allocation#LRU (Least Recently Used)]]
Key idea, evict the page that hasn't been used for the longest time.
- Add in 4, 2, 7 - OLDEST : 4 2 7
	- 3 page frames taken up
- 2 - this is a hit - OLDEST 4 7 2
- Add in 5 - all page frames are taken up - OLDEST 4 7 2 5
- Attempt to add in 3 - no space so we evict 4 - OLDEST 7 2 5 3
- Add in 3 - this is a hit - OLDEST : 7 2 5 3
- 2 - this is a hit - OLDEST : 7 5 3 2
- 3 - this is a hit OLDEST : 7 5 2 3
- 1 - this is not a hit - no space so we evict 7, OLDEST: 5 2 3 1
- 2 - this is a hit OLDEST : 5 3 1 2
- 6 - this is not a hit, evict 5 -> OLDEST : 3 1 2 6

| Step | Page | Memory State | Page Fault? | Note                     |
| ---- | ---- | ------------ | ----------- | ------------------------ |
| 1    | 4    | 4            | Yes         | Add 4                    |
| 2    | 2    | 4 2          | Yes         | Add 2                    |
| 3    | 7    | 4 2 7        | Yes         | Add 7                    |
| 4    | 2    | 4 2 7        | No          | Hit → mark 2 most recent |
| 5    | 5    | 4 2 7 5      | Yes         | Add 5                    |
| 6    | 3    | 2 7 5 3      | Yes         | Evict 4 (LRU), add 3     |
| 7    | 3    | 2 7 5 3      | No          | Hit → mark 3 most recent |
| 8    | 2    | 2 7 5 3      | No          | Hit → mark 2 most recent |
| 9    | 3    | 2 7 5 3      | No          | Hit → mark 3 most recent |
| 10   | 1    | 2 5 3 1      | Yes         | Evict 7 (LRU), add 1     |
| 11   | 2    | 5 3 1 2      | No          | Hit → mark 2 most recent |
| 12   | 6    | 3 1 2 6      | Yes         | Evict 5 (LRU), add 6     |
## Problem 7
[[Classes/Operating Systems/TLBs and Swapping#Page Faults]]
[[Classes/Operating Systems/TLBs and Swapping#Casues of Faults]] - ANSWERS HERE
For each of the following actions, explain whether it will trigger a fault or not. If it will trigger a fault, briefly describe what kind of fault will occur (e.g., page fault, protection fault) and how the OS will handle it.

a. A process accesses memory in a page that is currently swapped out to disk
For context, this is when each virtual page is in memory or on disk. Unused data is kept on the disk in a space called swap file.
 
***Page fault occurs. OS will find a free space in physical memory and take the swap file and insert it in there.*** Instruction is then restarted.

b. A process tries to dereference a NULL pointer (i.e., access memory at address 0x00)

***Protection fault occurs, one of the causes for page faults*** OS will send a segmentation fault back.

c. A process calls malloc
 No fault occurs immediately. The OS will first reserve the virtual space/pages for the specified number of bytes but nothing will happen yet.
***Page fault occurs when the process attempts to write to the allocated virtual memory.*** OS will throw an exception and then allocate space for a physical page/frame. Afterwards the virtual space that we allocated earlier gets updated to point to the physical space that was allocated.

d. A process calls a function, which pushes new variables onto the stack, extending beyond the current bounds of the stack

***Page fault occurs***. OS will allocate new space on the physical page.

e. A child process reads from a page that is shared with its parent using copy on write

Nothing.

f. A child process writes to a page that is currently set to read-only due to copy on write


***Protection fault***. This is due to the permissions not matching the access type - operation is not permitted on the page.

## Problem 8
[[Classes/Operating Systems/TLBs and Swapping#Demand Paging]]
Consider a demand-paging system with the following utilizations:
    CPU utilization: 20%
    Paging disk: 97.7% (demand, not storage)
    Other I/O devices: 5%

***Context***: demand paging is when pages are loaded into memory when first accessed, otherwise they are stored on disk. If it is not in memory we throw a page fault.

Given the utilization we can say that the
- CPU is mostly idle. What this mean's is that the actual runtime of a process is very low at the moment, the runtime is being held back via other facts such as faults being thrown...
- Paging disk has a high usage rate, meaning the disk is handling lots of page faults. remember what paging disks are, they are a section of disk used to store pages that are not currently in physical memory(RAM). 
	- So the disk holds data that the system will need eventually, but not at the current moment. (Holds evicted pages and such)
- Other I/O, regular actions going on such as network, mouse usage, keyboard usage...

For each of the following, say whether it will (or is likely to) improve CPU utilization. Briefly explain your answers.
   a. Install a faster CPU
	**Likely not, the main problem is that pages are not in memory a lot, so speeding up the CPU will just cause faults to be thrown at a quicker rate***
   b. Install a bigger paging disk
	***Likely not, the main source of the problem is increasing main memory***
   c. Increase the degree of multiprogramming
   ***Likely not, i think with speeding up the rate and the amount of processes running concurrently this will lead to more memory needed as we now need to manage more processes, which in turn mean's more physical memory will have to be added.***
   d. Decrease the degree of multiprogramming
   ***Likely, decreasing the amount of current concurrent processes will lead to less physical memory due to less demand. This potentially frees up more space in physical memory due to less processes running concurrently.***
   e. Install more main memory
   ***Likely, with more physical/main memory we can store more pages in-memory. Ensuring that more pages can be added and accessed without having to go to the disk as often***
   f. Install a faster hard disk, or multiple controllers with multiple hard disks
   ***Likely, given a faster hard disk or multiple controllers we can handle page faults at a faster rate, making way for the CPU to have more focus on the processes' runtime. Although this may provide a minimal improvement compared to other solutions such as adding main memory.***
   g. Add prefetching to the page replacement algorithms, - [[Classes/Operating Systems/TLBs and Swapping#Paging Policies]]
   ***Likely, as this loads in data into main memory that has a higher chance of being accessed directly after***
   h. Increase the page size
   ***Likely not, although we increase page size our main memory would still be the exact same which is the size we want to increase. We also run the risk of more internal fragmentation.***