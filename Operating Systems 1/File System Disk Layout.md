- [[#General]]
	- [[#File System Data Structures]]
	- [[#File System Challenge]]
- [[#File System Layout]]
	- [[#Blocks]] - files are represented as blocks in disk
	- [[#Block Management]] - how these blocks get stored
		- [[#Contiguous Layout]], [[#Linked Layout]], [[#Indexed Layout]], [[#Multi-Level Indexed Layout]],
	- [[#Unix Inodes]] - meta data on the disk about a file(s)
		- [[#Indirect Blocks]]
	- [[#Superblock]] - sort of the "root"
	- [[#Block Allocation (Bitmap)]] - tracks allocation
- [[#Common File Operations]]
	- [[#Hard Links]], [[#Symbolic (Soft) Links]]
	- [[#Create]], [[#Rename]], [[#Delete]]

## General
![[Screenshot 2025-05-22 at 3.52.49 PM.png|450]]

***File System API overview*** - covered in [[cse120/Operating Systems 1/Storage Devices and File System API]]
![[Screenshot 2025-05-22 at 3.53.31 PM.png|450]]
So there are 2 layers of the file system.
1. File system API, between the Applications -> OS
	1. Provided by the OS, think functions like `open()`
	2. Abstracts away low level details of blocks, sectors, physical disk layout
2. File System API / implementation , between OS -> disk
	1. Implementation of the "api", it handles taking the files and breaks it down into blocks, in which then gets stored onto the disk
	2. Store meta data
	3. Managing free space, caching...


### File System Data Structures
![[Screenshot 2025-05-22 at 3.54.54 PM.png|450]]

### File System Challenge
- Files grow and shrink
	- 6-8 orders of magnitude in file sizes
- Disk performance behavior
	- Highly nonuniform access
	- efficiency can be challenging
- Failures
![[Screenshot 2025-05-22 at 3.56.06 PM.png|450]]

## File System Layout
We init start with an empty disk, and we store files on the disk via fixed-size blocks.

The workflow
- User / application creates or writes a file
- The request goes to OS via file system API
- OS's file system takes the data and decides how to store it on disk, breaking it into blocks(various ways)
- The blocks are physical units of storage that live on disk

***High Level overview***
![[Screenshot 2025-05-22 at 4.23.08 PM.png|450]]

### Blocks
These blocks of data are literally on the disk itself! When a file is accessed, blocks are taken and loaded into memory.
![[Screenshot 2025-05-22 at 3.57.55 PM.png|450]]

### Block Management
- Several approaches into how we allocate blocks to files
	- How should we keep track of which blocks are for which file?

FILL THIS OUT BELOW

| Allocation | Pros                              | Cons                                               |
| ---------- | --------------------------------- | -------------------------------------------------- |
| Contiguous | Fast, simplifies directory access | Inflexible, causes fragmentation, needs compaction |
#### Contiguous Layout
![[Screenshot 2025-05-22 at 3.59.10 PM.png]]
	Fragmentation in memory can happen, not flexible

#### Linked Layout
![[Screenshot 2025-05-22 at 3.59.54 PM.png|450]]
	- More space will have to be allocated for metadata.. more overhead potential due to spending more time trying to locate where blocks are since they are not contiguous.
	- RAM is slower


#### Indexed Layout
![[Screenshot 2025-05-22 at 4.01.13 PM.png]]
- May need multiple index blocks...
- Index block can become huge in size, and it is not flexible
#### Multi-Level Indexed Layout
![[Screenshot 2025-05-22 at 4.01.21 PM.png]]

### Unix Inodes
![[Screenshot 2025-05-22 at 4.06.10 PM.png]]
#### Indirect Blocks
![[Screenshot 2025-05-22 at 4.08.04 PM.png|450]]

### Superblock
![[Screenshot 2025-05-22 at 4.11.47 PM.png]]

### Block Allocation (Bitmap)
![[Screenshot 2025-05-22 at 4.21.12 PM.png|450]]
***tradeoffs***
![[Screenshot 2025-05-22 at 4.21.21 PM.png|450]]



## Common File Operations

***aliasing : remember like referring to it via a different name. multiple names for it***

### Hard Links
![[Screenshot 2025-05-22 at 4.35.34 PM.png|450]]
- What happens when we delete a link?
	- It just gets unlinked from the inode it points to. In the pic above if we delete `link.txt` it simply removes the link to `inode`. `foot.txt` does not get affected.

### Symbolic (Soft) Links
![[Screenshot 2025-05-22 at 4.42.52 PM.png]]
### Create
![[Screenshot 2025-05-22 at 4.45.23 PM.png|450]]
### Rename
![[Screenshot 2025-05-22 at 4.45.43 PM.png|450]]

### Delete
![[Screenshot 2025-05-22 at 4.47.21 PM.png]]
