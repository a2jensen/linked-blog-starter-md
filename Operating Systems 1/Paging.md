Remember:
***Kernel***
core of an operating system, acting as an interface / bridge between hardware and software, managing resources, and enabling applications to run smoothly and securely

[[cse120/Operating Systems 1/Operating Systems Home]], paging already introduced in [[cse120/Operating Systems 1/Memory Management Overview]]
- [[cse120/Operating Systems 1/Memory Management Overview#Paging]]
- [[#Paging Mechanisms]]
	- [[#Information That A Page Table Contains]]
		- [[#Page Table Entries(PTEs)]]
	- [[#Representing Page Tables Efficiently]]
		- [[#Memory Requirements]]
		- [[#Managing Page Tables]] - managing huge tables
			- [[#Single-Level Page Tables]]
			- [[#Two-Level Page Tables]]
			- [[#Multi-Level Page Tables]]
				- [[#Address Translation x86-64]], simple address space here
	- [[#Page Table Management]] - where do we store page tables? in memory
		- [[#Storing]]
		- [[#Addressing]]
		- [[#Kernel Address Space]]
		- [[#More]]

# Paging Mechanisms

## Information That A Page Table Contains
### Page Table Entries(PTEs)
![[Screenshot 2025-05-08 at 6.06.36 PM.png]]
![[Screenshot 2025-05-08 at 6.07.28 PM.png|450]]
***It sort of stores the metadata***

## Representing Page Tables Efficiently

### Memory Requirements
[[Bytes Sizes Breakdown]] - for reference
![[Screenshot 2025-05-08 at 6.11.20 PM.png|400]]

How can we reduce the space required for page tables??
 - Try using larger pages....?
	 - 2MB or 1 GB in size per page
 - Some possible downsides include:
	 - Internal fragmentation, more memory may be wasted if page is not fully used
	 - Less flexibility - allocating memory might be harder
	 - Hardware Limitations
#### Managing Page Tables
![[Screenshot 2025-05-08 at 6.15.29 PM.png|450]]

***Thought Process***
to my understanding:
- single page tables is like one folder of pages, so if the single folder was incredibly large there would be a inefficient time to search for a single set or just one page - 
- two-level pages levels this up by taking that entire folder and splitting it into mini folders so at the surface level you have a set of folders, and within those folders stores a smaller set of papers. when querying for pages, the address space now allocates bits to help map which folder to go to, then some bits to indicate which page to jump to within the folder, and then offset bits to show where to jump to exactly within the page 
- then multi-level makes it so you can nest folders within folders within folders etc... this is my though process

![[Screenshot 2025-05-08 at 6.38.49 PM.png|450]]

##### Single-Level Page Tables
![[Screenshot 2025-05-08 at 6.18.18 PM.png|450]]
##### Two-Level Page Tables
![[Screenshot 2025-05-08 at 6.19.42 PM.png]]
![[Screenshot 2025-05-08 at 6.19.56 PM.png]]
![[Screenshot 2025-05-08 at 6.20.22 PM.png|450]]
![[Screenshot 2025-05-08 at 6.21.15 PM.png|450]]

![[Screenshot 2025-05-08 at 6.34.03 PM.png]]
##### Multi-Level Page Tables
![[Screenshot 2025-05-08 at 6.21.47 PM.png]]
![[Screenshot 2025-05-08 at 6.22.21 PM.png]]
![[Screenshot 2025-05-08 at 6.22.46 PM.png|450]]

## Page Table Management

### Storing
- Where do we store page tables?
	- Page tables are too large to store in the MMU
		- Store in memory instead
		- Special register holds the address of the top-level page map
### Addressing
![[Screenshot 2025-05-08 at 6.25.13 PM.png]]

### Kernel Address Space
![[Screenshot 2025-05-08 at 6.25.33 PM.png]]
- Left Diagram = what a user process can access
	- Process's view of memory
	![[Screenshot 2025-05-08 at 6.27.36 PM.png]]
- Right Diagram = what a user process can access + kernel only address space
	- Now includes the address space for the Kernel, in which the user process CANNOT access.
	- Reserved only for the Kernel, which remember is a major component of the OS acting as the interface for connecting hardware and software
![[Screenshot 2025-05-08 at 6.33.14 PM.png]]

### More
![[Screenshot 2025-05-08 at 6.33.28 PM.png]]