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