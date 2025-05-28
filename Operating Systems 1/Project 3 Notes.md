- [[#General]]
- [[#Task 1]]
- [[#Task 2]]

Refer to [[cse120/Operating Systems 1/TLBs and Swapping#Demand Paging]], [[cse120/Operating Systems 1/Project 2 Notes]]
## General

Relevant links to have open:
- [General Docs](https://cseweb.ucsd.edu/classes/sp25/cse120-a/projects/project3.html), [SysCall](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/test/syscall.h)
- [VMKernel](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/vm/VMKernel.java), [VMProcess](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/vm/VMProcess.java), [TranslationEntry](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/machine/TranslationEntry.java), [Nachos.conf](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/proj3/nachos.conf), [CoffSections](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/machine/CoffSection.java)
- [TA tips](https://cseweb.ucsd.edu/classes/sp25/cse120-a/projects/proj3-tips.txt), [TA 2 tips](https://cseweb.ucsd.edu/classes/sp25/cse120-a/projects/project3_guide.txt) - GO THROUGH THIS
- [Discussion](https://piazza.com/class_profile/get_resource/m8kx460ag1r2mu/mb1c6ek8tty5cu)

You project will implement the following functionality:
- Demand Paging. Pages will be in physical memory only as needed. When no physical pages are free, it is necessary to free pages, possibly evicting pages to swap.
- Lazy Loading. To fulfill the spirit of demand paging, processes should load no pages when started, and depend on demand paging to provide even the first instruction they execute. When you are done, loadSections will not allocate even a single page.
- Page Pinning. At times it will be necessary to "pin" a page in memory, making it temporarily impossible to evict.

### Key Design Aspects
- TranslationEntry bits
- Swap File
- Global Memory Accounting

### COFF Files
[[cse120/Operating Systems 1/COFF Files]] - more here
COFF (Common Object File Format) is a standard file format for executable, object code, and DLL files. It is one of multiple executable formats that an OS can recognize.

An operating system when it needs execute a program needs it in a compiled, encoded binary file that it can understand. The format tells the OS:
- where the code and data are
- how to map them into memory
- entry point / starting address location
- permissions for each section(e.g., read-only, executable)

***Key idea: think of coff files as an executable file of a user program. We cannot read it directly as it's encoded in binary, but the operating system knows how to load the coff file and run it***

with the idea of [[2 - Encoding#Encoding TMs|encoding]], where we encode data to fit the context of a problem
When i run something along the lines of 
```
new UserProcess().execute("test.coff");
```
We are essentially telling Nachos "hey kernel, here's a user program in COFF format. Load it into memory, set up the page table, and start running it in user mode."

### Class Overview
Whats being given.... remember both classes are an extension of User versions. So have access to their functions as well.

```
VMProcess class
saveState
restoreState
loadSections
unloadSections
handleException

var pageSize, char dbgProcess, dbgVM
```

```
VMKernel class
init
selfTest
run - starts running user programs
terminate - terminates kernel
dummy1, dbgVM 
```
## Task 1
Implement demand paging.
Design aspects involved: valid bit, used bit

***loadSections***
```
check if space is not fully filled in physical memory
init new entry in our pageTable via TranslationEntry[numPages]
init linkedList - keeps track of allocated pages in process

check memory avaiailability
for loop that iterating through every virtual page, and allocate PP for it
	- take vpn and allocate it via allocatePage()
	- if out of memory, exit the creation of this process, and
		- free all memory that had been allocated beforehand
	- check if VPN corresponds to a read only section in coff
		- if so, mark the physical page as read-only
	- now, link the VPN to the PPN via translationEntry function call

we do not load any data in memory to the pageTable....since we are doing demand paging.
```


***handlePageFault***
```

```

## Task 2
Design aspects involved, dirty bit