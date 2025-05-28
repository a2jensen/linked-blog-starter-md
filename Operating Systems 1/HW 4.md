## Problem 1
On a Unix-style file system, how many disk read operations are required to open and read the first block of the file "/usr/include/X11/Xlib.h"? Assume (1) that the superblock is in memory, but nothing else; (2) that all directories and inodes are one block in size; (3) that once a block is read, it is cached in memory (and does not need to be read again); (4) inode timestamps (like last access) do not need to be updated (the inode does not need to be written after access).

Context:
Directories are a special kind of file whose data blocks store filenames and inode mappings, there just an abstraction over file

Key idea: read in inode of the current directory, then we read in the directory file.
[[cse120/Operating Systems 1/File System Disk Layout#Blocks]]

1. First read the superblock - in memory
2. Read inode for `/` directory file - within superblock so in memory
	1. Read `/`. we are now within `/` directory. 1 read
		1. Read inode within root `/` , looking for pointer to `/usr`  file directory. 2 reads
		2. Read in `/usr` directory file, we are now within `/user`. 3 reads
			1. Read the inode within `/usr` and find the pointer to `/include`. 4 reads
			2. Read in `/include` file directory. 1. We are now in `/usr/include` 5 reads
					1. Read the inode within `/include` and find the pointer to `/X11`. 6 reads
					2. Read in `/X11` file directory. We are now in `/usr/include/X11`. 7 reads. 
							1. Read inode within `/X11` and jump to `XLIB.h`. 8 reads.
							2. Read in first block for `XLIB.h`. 9 reads.

Given that superblock was in memory already, we did not make a disk read with that one. That gives us 9 total reads. 

Algo
- We read in the inode in the current file directory we’re within to get a pointer to the file directory or file we want to reach. 
- Afterwards, then we read in that file or file directory data block that we were looking for when reading the inode.
- Repeat