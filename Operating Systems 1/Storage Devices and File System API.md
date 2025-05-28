
- [[#Physical Storage Devices]]
	- [[#Disks and the OS]] - disks, flash NVM, RAID
		- [[#Hard Disk Drives (HDDs)]]
			- [[#Disk Components]]
			- [[#Reading Disk Sector]]
			- [[#Disk Performance]]
		- [[#API between Disks and the OS]]
		- [[#Disk Scheduling Policies]]
	- Performance
		- [[#Disk Scheduling Policies]]
	- [[#Solid State Drives]]
	- [[#Non-Volatile Memory(NVM)]]
	- [[#Redundant Array of Inexpensive Disks (RAID)]]
		- [[#Striping, Mirroring, Parity Disk]]
- [[#File System APIs for users and programs]]
	- files, directories, sharing, protection
	- [[#Components]]
	- [[#Abstractions]]
	- [[#Files]]
		- [[#File Abstraction vs Physical Reality]], [[#Basic File Operations]], [[#File Access Patterns]], [[#Directories]], [[#Path Name Translation]], [[#File Sharing]]
		- [[#Protection]]
			- [[#Unix Acess Rights]], [[#Root and Admin]]


## Intro
![[Screenshot 2025-05-20 at 3.34.49 PM.png|450]]
- example, we store swap files on storage devices. These are not in "memory"

What are the characteristics of physical storage devices?(structure, performance)

What API should users/programs use to access storage devices?
- files, directories, protection
- API's implemented via system daya structures, buffer cache and they need to be implemented to ensure reliability

## Physical Storage Devices

### Disks and the OS
![[Screenshot 2025-05-20 at 3.37.12 PM.png|450]]

#### Hard Disk Drives (HDDs)
![[Screenshot 2025-05-20 at 3.39.11 PM.png]]
	- Used across data centers still, in tons of old infrastructure
	- A lot of the software back then was designed in mind of using HDDs

![[Screenshot 2025-05-20 at 3.41.15 PM.png|450]]
very small.. only 3.5 inches. Any dust with the HDD can mess with reading/writing data, so there is usually a filter that handles this. Up to 15,000 rotations per minute.

Head is responsible for reading/writing the data from the track... the arm is responsible for moving the "platter" around I think. There can be multiple platters as well within a disk. The "cylinder" refers to like the entire space of the platters, so in the picture we have 3 platters and that is 1 cylinder. NOT FULLY SURE ABOUT CYLINDER DESCRIPTION CHECK FIRST
#### Disk Components
![[Screenshot 2025-05-20 at 3.46.03 PM.png|450]]
#### Reading Disk Sector

![[Screenshot 2025-05-20 at 3.52.36 PM.png]]

How does the driver know where to read in the disk? Short answer -> there is a data structure containing metadata telling the OS where to read at in the disk

*another example*
![[Screenshot 2025-05-20 at 3.52.56 PM.png|450]]

#### Disk Performance
![[Screenshot 2025-05-20 at 3.53.42 PM.png|450]]

If the OS is reading from the disk, it is incredibly slow... the goal is to keep everything in memory! Reading from disk is something we want to avoid.

![[Screenshot 2025-05-20 at 3.56.23 PM.png|450]]

### API between Disks and the OS

***in the past...***
- Specifying disks requests requires a lot of info:
	- Surface, track, sector, transfer size....
- Older disks required the OS to specify all this!! However there are con's behind this...
cons:
- lots of overhead and complexity is added to the OS. The OS has to know all the parameters for querying data from the disk.

***currently***
An interface was added between the disk and the OS, lots of the complexity was moved into the hardware.
![[Screenshot 2025-05-20 at 4.00.25 PM.png|450]]
- cons include the OS not having full control, trading ***complexity for simplification, which may remove some possible optimizations***
- blocks / variable sizes in the disk are static and not dynamic
	- this can lead to higher risk of unused memory

### Disk Scheduling Policies
![[Screenshot 2025-05-20 at 4.03.54 PM.png|550]]
Where are these algorithms implemented? Inside the disk.

### Solid State Drives
![[Screenshot 2025-05-20 at 4.07.54 PM.png|550]]
- compared to HDD it has lower capacity, but faster performance
- Compared to DRAM, it is larger but DRAM is faster
	- ***SDD is the middleground between DRAM and HDDs***
- When it mean's by storing data less reliably : increase in errors of reading/writing data...

SSDs are newer, so a lot of the file systems remain unchanged and are still built for HDDs. To make a file system work for an SSD you just remove certain parts of a file system... so just adjusting the system per say instead of rewriting it from scratch.

### Non-Volatile Memory(NVM)

![[Screenshot 2025-05-20 at 4.11.04 PM.png|550]]
NVM is a more recent technology. Unclear on whether it will be widely adopted.
### Redundant Array of Inexpensive Disks (RAID)

#### Striping, Mirroring, Parity Disk

![[Screenshot 2025-05-20 at 4.14.14 PM.png]]

In the diagram on the bottom, you read from left to right - disk 0, disk 1... disk n
- Read in parallel
![[Screenshot 2025-05-20 at 4.14.30 PM.png]]
This is nice... however this just double our cost in data being utilized.. so the fix is to use a *parity disk*
![[Screenshot 2025-05-20 at 4.16.10 PM.png]]
with parity disks, we store just enough data so in the case we lose anything. So not exactly copying the data and storing it as backup.
## File System APIs for users and programs

### Components
![[Screenshot 2025-05-20 at 4.20.31 PM.png]]
It would suck for a user to have to know the sector location and all the other components to grab data from.. this is abstracted away via files and directories.

### Abstractions
![[Screenshot 2025-05-20 at 4.21.49 PM.png|650]]
Files and directories are the abstraction!
#### Files
![[Screenshot 2025-05-20 at 4.24.00 PM.png|450]]

##### File Abstraction vs Physical Reality
![[Screenshot 2025-05-20 at 4.26.45 PM.png|450]]
*The left view*: what the users do not see, what could possibly happen if there was no abstraction - risk of data loss, zero protection
*The right view*: user view - what they see when using the file system. They do not have to worry about data being corrupted. this is handled via the interface / abstraction.

##### Basic File Operations
![[Screenshot 2025-05-20 at 4.27.29 PM.png]]

##### File Access Patterns
![[Screenshot 2025-05-20 at 4.28.31 PM.png|450]]
- Examples of random access file access pattern
	- For accessing swap file
	- Databases
	- A poker game, randomly pick a byte(card)
- Examples of indexed access
	- File system contains an index to blocks with particular contents
		- E.g., hash tables, dictionaries, hash sets

##### Directories
![[Screenshot 2025-05-20 at 4.30.06 PM.png]]
In early computers, we only had one directory.

![[Screenshot 2025-05-20 at 4.33.42 PM.png]]

##### Path Name Translation
![[Screenshot 2025-05-20 at 4.37.53 PM.png|450]]
- We open director "/",
- Search for the entry "one", get "location of "one"
- Open directory of "one", search for "two" and get location of "two"
- Open directory "two", search for "three" and get location of it
- Open file "three"
![[Screenshot 2025-05-20 at 4.39.35 PM.png|450]]
##### File Sharing
![[Screenshot 2025-05-20 at 4.40.15 PM.png|550]]

##### Protection
![[Screenshot 2025-05-20 at 4.41.10 PM.png]]
- `motd` means `message of the day`, so we can read but not write since we want people to just read the `motd`

###### Unix Acess Rights
![[Screenshot 2025-05-20 at 4.43.02 PM.png|450]]
###### Root and Admin
![[Screenshot 2025-05-20 at 4.43.35 PM.png]]
