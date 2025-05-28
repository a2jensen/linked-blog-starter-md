Related - [[cse120/Operating Systems 1/Paging]], [[cse120/Operating Systems 1/Memory Management Overview#Virtual Memory]], [[cse120/Operating Systems 1/Operating Systems Home]]

Translation Lookaside Buffers:
- Cache address translations
Demand paging :
- Caching the actual data pages from disk into memory
Both serve different purposes.

***TLDR***
- [[#Inefficient Translations]]
- [[#Efficient Translations]]
	- [[#Translation Lookaside Buffers(TLB)]] - cache to store address translations(represented by PTEs)
		- [[#Address Translation with a TLB]]
		- [[#Managing TLBs]]
			- [[#Handling Misses]]
			- [[#Context Switches]]
- [[#Illusion Of More Memory Than Physically Present]]
	- [[#Demand Paging|Demand Paged Virtual Memory]] - store data on disk in swap file, load pages on demand. so check cache and load it into cache if it's not there
		- [[#Page Faults]]
			- [[#Casues of Faults]]
		- [[#Paging Policies]]
		- [[#Page Frame Questions]]
		- [[#Address Translation for a Swapped-Out Page]]
- [[#Advanced Functionality]]
	- [[#Shared Memory]]
	- [[#Copy on write]]
	- [[#Mapped Files]]
- 

## Inefficient Translations
![[Screenshot 2025-05-13 at 3.38.48 PM.png|450]]
Linear Page table:
2 : page table + data itself
N-level page table
N + 1 : N levels of page maps + data itself

## Efficient Translations

### Translation Lookaside Buffer(TLB)
***A hardware cache of recently used translations, cache stores the virtual page number and its corresponding page table entry.*** 
- Paging Covered in [[cse120/Operating Systems 1/Paging#Page Table Entries(PTEs)]]
- Remember Page Tables: Determine mapping of virtual addresses to physical addresses
	- [[cse120/Operating Systems 1/Paging#Page Table Entries(PTEs)|Page Table Entries]] - bits that are sort of like "metadata", tell us whether
		- page has been written to
![[Screenshot 2025-05-13 at 3.41.02 PM.png|450]]
- **extra bullet point**
	- ***on every address translation***
		- Consult the TLB first
		- If a miss, walk through the page table

Fully associative cache , reminder on those is in [[Caching Lecture]] and defined also in [[Computer Architecture#Key Concepts in Cache Design]] in which it covers:
- direct mapped cache and n-way associative cache
- Difference between Direct mapped cache and fully associative cache
	- *Direct : each memory block maps to exactly one cache line*
		- Cache index  / line determined utilizing block address and number of cache lines
	- *Fully associative: any memory block can go into any cache line*
		- We search through cache and insert at first open cache line

#### Address Translation with a TLB
![[Screenshot 2025-05-13 at 3.48.16 PM.png|450]]
#### Managing TLBs
![[Screenshot 2025-05-13 at 3.49.02 PM.png]]

##### Handling Misses
![[Screenshot 2025-05-13 at 3.49.35 PM.png]]

##### Context Switches
![[Screenshot 2025-05-13 at 3.52.54 PM.png]]

## Illusion Of More Memory Than Physically Present

![[Screenshot 2025-05-13 at 4.12.13 PM.png]]
***Options above, DRAM or Disk***

### Demand Paging
![[Screenshot 2025-05-13 at 4.15.44 PM.png]]
TLB: cache for page table entry, [[cse120/Operating Systems 1/Paging#Page Table Entries(PTEs)|PTE]] covers this but remember:
- a paging table entry is suppose to map a virtual page number to physical frame number
- Part of page table
Demand paging :
- Caching the actual data pages from disk into memory

![[Screenshot 2025-05-13 at 4.20.56 PM.png|450]]

##### Page Faults
![[Screenshot 2025-05-13 at 4.21.51 PM.png]]
- Hardware saves the faulting address in a special register
- Evict a page to memory

##### Casues of Faults
![[Screenshot 2025-05-13 at 4.33.25 PM.png]]


##### Paging Policies
![[Screenshot 2025-05-13 at 4.23.33 PM.png]]

##### Page Frame Questions
- Is it safe for the OS to reclaim a physical page from process A and grant it to process B?
		Yes, but the OS must *zero the page* first if its a newly allocated page
- When we evict a page, do we always need to write the contents to the disk?
	- Only modified (dirty) pages need to be written to the disk
	- Clean pages do not(already on disk)
	- Use the modified/dirty bit in the PTE
![[Screenshot 2025-05-13 at 4.31.31 PM.png|400]]


##### Address Translation for a Swapped-Out Page
![[Screenshot 2025-05-13 at 4.36.21 PM.png]]
- Walkthrough for virtual to physical address translation, since the page is "swapped-out" that mean's it is currently on disk and not in memory. It was before in memory however it got switched out at some point and put back on disk
1. We check if its in the [[#Translation Lookaside Buffer(TLB)|TLB]], which is the cache for [[cse120/Operating Systems 1/Paging#Page Table Entries(PTEs)|PTEs]], it is not since it is on the disk
2. We reload the TLB adding the PTE on for the virtual address, and recheck again in which it will be in TLB
3. Check if bits allow access, it was on disk so they do not so we end up with a page fault which will fetch page(not the PTE) and reload it on TLB
4. With the page now being in memory, the bits in the PTE allow access so we can combine PFN from PTE with offset


## Advanced Functionality

### Shared Memory
![[Screenshot 2025-05-13 at 4.40.42 PM.png|450]]
![[Screenshot 2025-05-13 at 4.42.30 PM.png|450]]
Again, this topic is familiar from [[PA3 Tiled Matrix Multiply#Loading In Tile A and B and Tile iteration Explained]]

### Copy on Write
![[Screenshot 2025-05-13 at 4.43.42 PM.png]]

### Mapped Files
![[Screenshot 2025-05-13 at 4.45.38 PM.png|450]]
![[Screenshot 2025-05-13 at 4.47.29 PM.png|450]]