[[cse120/Operating Systems 1/Memory Management Overview]], [[cse120/Operating Systems 1/Paging]]

- [[#Intro]]
	- [[#Running at Memory Capacity]]
- [[#Page Replacement Policies]]
	- [[#Locality]]
		- Algorithms - only some were covered below. all can be found in overview slide
			- [[#Optimal or MIN(Belady's Algorithm)]]
			- [[#Random]]
			- [[#First-In First-Out (FIFO)]]
			- [[#LRU (Least Recently Used)]]
			- [[#Clock]]
	- [[#Synchronous vs Asynchronous]]
	- [[#Handling Multiple Processes]]
	- [[#Working Set Model]]
	- [[#Thrashing]]
- [[#Virtual Memory Allocation]]
	- [[#Growing The Stack]]
	- [[#Managing Heap Memory]]

# Intro
![[Screenshot 2025-05-15 at 4.02.09 PM.png|450]]
The evicting of pages from *PHYSICAL MEMORY(RAM)*

*Refresher*
***Virtual Memory*** - The illusion, what the process sees. Each process gets a limited amount of GB's but it thinks that it has "access" to all pages within the virtual memory, however this is not true
***Physical Memory(RAM)*** - Small, fast memory where only active pages are loaded, covered in [[cse120/Operating Systems 1/Memory Management Overview#Virtual Memory|Management Overview]]. This is what the "hardware knows" and works with,.
***Disk*** - The backing store - pages not in RAM are here. When a page fault occurs, the OS may fetch a page from disk into RAM, potentially evicting another using a page replacement policy.

![[Screenshot 2025-05-15 at 5.23.29 PM.png|450]]
![[Screenshot 2025-05-15 at 5.23.59 PM.png|450]]

### Running at Memory Capacity
- Expect to run with most physical pages in use
	- OS maintains a small buffer of free page frames
- Every demand paging request requires an eviction
	- OS must choose which pages to replace...

## Page Replacement Policies
![[Screenshot 2025-05-15 at 3.57.00 PM.png|450]]

Lots of different page replacement policies:
![[Screenshot 2025-05-15 at 4.02.50 PM.png|250]]

#### Optimal or MIN(Belady's Algorithm)
![[Screenshot 2025-05-15 at 5.43.47 PM.png|450]]
#### Random
![[Screenshot 2025-05-15 at 5.44.06 PM.png|450]]
#### First-In First-Out (FIFO)
![[Screenshot 2025-05-15 at 5.45.10 PM.png|450]]
![[Screenshot 2025-05-15 at 5.45.45 PM.png|450]]
### Locality
- Locality - programs tend to access certain code and data frequently
- Temporal Locality
	- Locations referenced recently are likely to be referenced again
- Spatial Locality
	- Locations near recently referenced locations are likely to be referenced again
***goal : leverage temporal locality to improve our page replacement policy***

### LRU (Least Recently Used)
![[Screenshot 2025-05-15 at 4.14.21 PM.png|550]]
***Cons***
- Evicting by the least recently used sounds good, but it's possible to do better by evicting by the least often used.

***How to approximate LRU***
- Use the reference bit in the PTE, [[cse120/Operating Systems 1/Paging#Page Table Entries(PTEs)|PTEs]]
	- Set when the bit when a page is accessed
	- Clear the reference bits periodically
- Reference bits tell us if a page was used recently or not
![[Screenshot 2025-05-15 at 4.19.03 PM.png|300]]


### Clock
![[Screenshot 2025-05-15 at 4.22.13 PM.png]]
- When bit is 0, has not been used recently
- When bit is 1, that mean's it was used so we set to 0 and move on
***Pros***
- Favors [[#Locality|Temporal Locality]]
- Low overhead
- Simple to implement, considers how recently a page was used
***Cons***
- Worse case make take a long to evict


### Synchronous vs Asynchronous
![[Screenshot 2025-05-15 at 4.29.06 PM.png]]
Essentially a thread in the background that is ready with clean pages on hand to use when evictions happen.

Downside with asynchronous : extra memory needs to be allocated to manage the pool of unused, clean pages. BUT usually it's small enough to not make a difference

MOST SYSTEMS USE ASYNCHRONOUS

### Handling Multiple Processes
![[Screenshot 2025-05-15 at 4.35.06 PM.png|450]]
MOST SYSTEMS USE GLOBAL REPLACEMENT
### Working Set Model
![[Screenshot 2025-05-15 at 4.35.37 PM.png|450]]

### Thrashing
![[Screenshot 2025-05-15 at 4.36.43 PM.png|450]]


## Virtual Memory Allocation
Goes over what the operating system does when a process wants to allocate more memory.
![[Screenshot 2025-05-15 at 4.39.50 PM.png|450]]
### Growing The Stack
![[Screenshot 2025-05-15 at 4.43.04 PM.png|450]]

### Managing Heap Memory
![[Screenshot 2025-05-15 at 4.44.05 PM.png]]
The heap has internal fragmentation, so a library keeps track of where in the heap memory is free to use... what happens when the memory that needs to get allocated does not fit onto the heap??? Well we will need to grow the heap...
![[Screenshot 2025-05-15 at 4.45.02 PM.png]]

