
- [[#Intro]] - review on file system(inodes, superblock)
- [[#Optimized Disk Layout]]
	- [[#BSD Fast File System]]
		- - [[#Cylinder Groups]]
		- [[#Disk Layout + Policy]]
	- [[#Cylinder Groups vs Block Groups]]
- [[#File Buffer Cache]]
	- [[#Writing To Cache Policies]] - write-through, write-back
- [[#File System Reliability]]
	- [[#Crash Consistency Problem]]
	- [[#fsck]], [[#Ordered Writes]], [[#journaling]]

## Intro
Reminder on these key components, covered in [[cse120/Operating Systems 1/File System Disk Layout]]

***inodes***
![[Screenshot 2025-05-27 at 3.44.37 PM.png|450]]

***file system layout***
![[Screenshot 2025-05-27 at 3.44.53 PM.png|450]]

![[Screenshot 2025-05-27 at 3.45.44 PM.png|450]]

![[Screenshot 2025-05-27 at 3.45.29 PM.png|450]]


## Optimized Disk Layout
![[Screenshot 2025-05-27 at 3.47.52 PM.png|450]]
Cons:
- Fixed max number of files
- Inodes are far away from data blocks, lots of overhead seeking

### BSD Fast File System
- increased block size
	- 4 KB instead of 512 bytes
	- bit map instead of a free list
- place data in groups, and ensure spatial locality by placing them near each other
	-  place inodes in the same cylinder group as their associated directory
	- place the data blocks in the same cylinder group as their associated inode

***disk cylinders refresher***
![[Screenshot 2025-05-27 at 3.51.38 PM.png|450]]

#### Cylinder Groups
- A group of consecutive cylinders
- data in the same cylinder can be accessed quickly

![[Screenshot 2025-05-27 at 3.52.38 PM.png|450]]

#### Disk Layout + Policy
![[Screenshot 2025-05-27 at 3.53.03 PM.png|450]]
![[Screenshot 2025-05-27 at 3.53.15 PM.png|450]]

### Cylinder Groups vs Block Groups
![[Screenshot 2025-05-27 at 3.56.08 PM.png|450]]


## File Buffer Cache
Cache for files

![[Screenshot 2025-05-27 at 4.05.28 PM.png|450]]

#### Writing To Cache Policies

On a write, applications may assume that data has been written to the disk, *however, this is not necessarily true with caching*

***Write-through caching***
- Write to storage immediately
- Pros:
	- Simple, cache stays consistent with storage
- Cons:
	- More overhead added to waiting for writes to disk

***Write-back caching***
- Gather (buffer) writes in memory and then write all buffered data back to the storage device (e.g., every 30 seconds in Unix)
- Pros:
	- Fast writes since we do not depend on disk write completion, optimized writing since we are batch loading, decreases the number of writes
- Cons:
	- May lose data, it feels less "safe"

## File System Reliability
Loss of data has catastrophic effects
Three threats
- Accidental or malicious deletion of data
	- Fix -> back up entire file system periodically
- Disk failure
	- Fix -> replicate data (RAID) 
- System crash or loss of power - *primary focus*
	- Fix -> Ensure consistency within file system

### Crash Consistency Problem
![[Screenshot 2025-05-27 at 4.20.50 PM.png|450]]

***Example : Only Write Data Block***
![[Screenshot 2025-05-27 at 4.26.33 PM.png|450]]
***It is as if the data was never written, the data block is never accessible since there is no inode***

***Example : Only Write Inode***
***Outcome:***
- Could read garbage data
- File system is inconsistent (bitmap and inode disagree)

***Example : Only Write Bitmap***
***Outcome***
- Space leak, you end up marking blocks of data as being used when they actually aren't
	- You end up wasting space.

### Solution
***Keep these file system operations as atomic***
	- These file system operations include multiple writes to inode block, data block, bitmap, indirect block, etc.
How can we keep these operations atomic? With locks?
- The problem with locks is that it won't fix the fundamental problem of preventing crashes

##### fsck
![[Screenshot 2025-05-27 at 4.38.21 PM.png|450]]
***Cons***
- The file system checker does not know what the proper state should be, so it may not restore data to a valid state
- If the data block was never written, then there is no way to recover it in that scenario
- More overhead of running the file system checker - can take hours to run on large disk volumes

##### Ordered Writes
![[Screenshot 2025-05-27 at 4.46.05 PM.png|450]]
![[Screenshot 2025-05-27 at 4.46.27 PM.png|450]]
- Cons
	- can leak resources
	- Must wait for writes to complete, making it slower
	- Difficult to find a safe interruptible ordering for all operations

![[Screenshot 2025-05-27 at 4.48.24 PM.png|450]]


##### Journaling