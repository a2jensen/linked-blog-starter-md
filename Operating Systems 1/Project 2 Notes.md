[[cse120/Operating Systems 1/Operating Systems Home]]
The goals of the second project are to implement a core set of system calls and to support multiprogramming of user-level programs.
- Write user-level processes
- Connect the system call functionality for user-level processes, system call API already written.

TLDR
- [[#UserKernel Overview]]
	- [[#In simple terms]]
- [[#UserProcess Overview]]
- [[#handleSysCall()]]
- [[#Task 1]]
	- [[#StubFileSystem Notes]]
	- [[#FileTable]]
	- [[#create() and open()]] 
	- [[#read() and write()]]
	- [[#Edge Cases]] - REFER TO THIS FOR CURRENT PROBLEMS
- [[cse120/Operating Systems 1/ExamCode/Nachos VM Worksheet]]
- [[#Task 2]]
	- [[#Load sections]]
	- [[#ReadVirtualMemory + Write Virtual Memory]]
- [[#Task 3]]
	- [[#***handleExec***]]
	- [[#***handleJoin***]]
	- [[#***handleExit***]]


***Important Links/Notes***
Tabs I had opened for Task1:
- [Project1Docs](https://cseweb.ucsd.edu/classes/sp25/cse120-a/projects/project2.html)
- [UserProcess](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/task1/nachos/userprog/UserProcess.java), [UserKernel](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/userprog/UserKernel.java), [Syscall](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/test/syscall.h)
- Discussion slides for [project overview](https://piazza.com/class_profile/get_resource/m8kx460ag1r2mu/ma7c6b1mmhw286)
- Tips from [TA 1](https://cseweb.ucsd.edu/classes/sp25/cse120-a/projects/project2_guide.txt), Tips from [TA 2](https://cseweb.ucsd.edu/classes/sp25/cse120-a/projects/p2.tips)
- [machine/StubFileSystem.java](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/machine/StubFileSystem.java) - interface for OpenFiles classes i think
				- [machine/OpenFile.java](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/machine/OpenFile.java)
				- [machine/OpenFileWithPosition.java](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/machine/OpenFileWithPosition.java)
- [[cse120/Operating Systems 1/ExamCode/HW 3]]

![[Screenshot 2025-05-07 at 12.41.09 PM.png]]

#### UserKernel Overview
A kernel that can support multiple user processes. The class is just an abstraction over `ThreadedKernel`, with added functionality for user mode support
###### In simple terms:
- **`ThreadedKernel`** is the **real kernel**.
    - It implements the thread scheduler, file system, console, and all OS-level functionality.
    - All system-wide services live here.
- **`UserKernel`** is a **specialization of `ThreadedKernel`**.
    - It adds support for **running user programs**.
    - Initializes things like the console for stdin/stdout.
    - It doesn't override or replace core kernel services like the file system — it just adds **user-mode environment setup**.

```Class contents
// init kernel and create a console with exception handler
constructor

// returns current process
currentProcess()

// called by processes whenever user instruction causes error
exceptionHandler() 

// run user programs via creating process and running shell with
run()

// terminate kernel
terminate()
```

#### UserProcess Overview
```Class contents
// make new process
constructor

execute()

saveState()
restoreState()
readVirtualMemoryString()
readVirtualMemory()
	readVirtualMemory() // handles actual implementation
writeVirtualMemory()
	writeVirtualMemory() // hamdles actual implementation
load()
loadSections()
unloadSections()
initRegisters()
handleHalt()
handleExit()
handleSyscall()
handleException()
)
```

### handleSysCall()
- Function that will handle different types of syscalls
![[Screenshot 2025-05-07 at 4.19.59 PM.png|450]]

### Parameters
### Task 1
- Tips from [TA 1](https://cseweb.ucsd.edu/classes/sp25/cse120-a/projects/project2_guide.txt)
	- Check out these files: 
	- NOTE: These classes are accessibily via the kernel, don't directly call them.
			- [machine/StubFileSystem.java](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/machine/StubFileSystem.java) - interface for OpenFiles classes i think
				- [machine/OpenFile.java](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/machine/OpenFile.java)
				- [machine/OpenFileWithPosition.java](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/machine/OpenFileWithPosition.java)
		- about virtual memory:
	        1) Your only calls to Machine.processor().getMemory() should be in
	          readVirtualMemory() and writeVirtualMemory()!  (or some function
	          called by them.)  That way, those two VM functions serve as your
	          interface to the virtual mapping, and you can thus generally ignore
	          the non-contiguity of mapped physical memory addresses.
	- don't stress, most of what you need is in existing UserProcess,
      UserKernel, filesystem/openfile docs, syscall.h.  Rely upon
      syscall.h as the 'spec' for your syscalls!
- Tips from [TA 2](https://cseweb.ucsd.edu/classes/sp25/cse120-a/projects/p2.tips)

![[Screenshot 2025-05-07 at 12.45.39 PM.png|450]]
- Semantics/specifications documented here in [test/syscall.h](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/test/syscall.h)
- Calling conventions are documented in the comments to [UserProcess.handleSyscall](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/main/nachos/userprog/UserProcess.java)
- code for `halt` and skeleton code for `exit` can be found in UserProcess.java, we want to implement the following syscalls with the same pattern.
- Note that you are _not_ implementing a file system. Rather, you are simply giving user processes the ability to access a file system that Nachos already implements.
	- All we are doing i am going to assume is calling functions, just sort of connecting the user with the API.

##### StubFileSystem Notes
 ***Use stubFileSystem via machine.stubFileSystem()! (specified in discussion) instead of directly using openFileWithPosition or OpenFile***
 
***StubFileSystem*** sort of is like the public API. Within the class another class is defined, ***StubOpenFile*** which acts as a child for the classes:
- OpenFile 
- OpenFileWithPosition

How so?
- When we `stubFileSysem.open()`, what gets returned is a `StubOpenFile` object which extends `OpenFileWithPosition`, and that class in turn extends `OpenFile`, making it so that the `StubOpenFile` inherits all properties of its parent classes.
	- The confusion arose when the `open` function says it returned a type `OpenFile`, which is technically true but I think `OpenFile` is sort of like the super parent class, and with `StubOpenFile` it extends that + `OpenFileWithPosition`

##### FileTable
- Still need to handle setting the keyboard and screen for first 2 elements in the array. Refer to slide 
![[Screenshot 2025-05-07 at 3.32.37 PM.png]]
##### create() and open():
- vaName is the starting address of the string name
	- Want to use readVirtualMemoryString to get string
Reasoning for `ThreadedKernel.fileSystem.open(...)`
- Remember, file system is something that the OS handles. ThreadedKernel represents the kernel inside the OS, so we want it to handle the file system. 
	- Handles file system, threads, scheduler, console
	- This is why we don't directly call `StubFileSystem` class, it is already connecteed to `ThreadedKernel` as one of the variables in `ThreadedKernel` is the `FileSystem` class being init, and `StubFileSystem` is a child of `FileSystem` therefore making it callable from the `ThreadedKernel`
	- ***Use the ThreadedKernel object for handling file system, it represents the Kernel, the abstraction in the OS for handling file system calls and what not***
	- ***`ThreadedKernel.fileSystem` is like a built-in system service***

##### read() and write()
- I know calling `Machine.stubFileSystem.open()..` returns a type OpenFile.... now I think I want to access the functions within OpenFileWithPosition since those actually have the logic of reading stuff within a file... however it seems to be it returns of type StubOpenFile... which is an extension of OpenFileWithPosition.
	- Refer to this [convo](https://chatgpt.com/c/681bd27f-347c-800d-b66a-983877edf0ee)
- I AM ABLE TO REFER TO OPENFILEWITHPOSITON through the StubOpenFile object
	- StubOpenFile object extends OpenFileWithPosition
		- OpenFileWithPosition extends OpenFile....
	I can access the `read` methods which handle functionality. So I guess I don't need to directly access the classes themselves, just the StubOpenFile object

##### Edge Cases
- [x] Overloading File System ✅ 2025-05-12
- [x] Close properly removes files ✅ 2025-05-12
- [x] `Test4` not working, infinite loop when asking for input ✅ 2025-05-19
- [x] Invalid addresses, look more into this ✅ 2025-05-19


### Task 2

***What is coff***
- COFF stands for Common Object File Format. It’s a standard binary file format for executable programs, object code, and shared libraries.
In nachos, a .coff file is a compiled user program that Nachos can load and run.


 Implement support for multiprogramming. The initial Nachos code is restricted to running only one user process, and your task is to make it work for multiple user processes.
- Handle allocating physical pages in memory such that multiple processes do not overalap each other in terms of allocating memory.
	- create and initialize the pageTable data structure for each user process, which maps the process's virtual addresses to physical addresses.
	- Modify UserProcess.loadSections() so that it allocates the pageTable and the number of physical pages based on the size of the address space required to load and run the user program (and no larger). 
		- If the new user process cannot allocate sufficient physical pages for its address space, exec should return an error.
		- All of a process's memory should be freed on exit (whether it exits normally, via the syscall exit, or abnormally, due to an illegal operation)
	- Modify UserProcess.readVirtualMemory and UserProcess.writeVirtualMemory, which copy data between the kernel and the user's virtual address space, so that they work with multiple user processes.

Summary of work done.
```
For this task I implemented loadSections, readVirtualMemory, and writeVirtualMemory. I also added helper functions, allocatePage() and freePage(), to allocate and free physical memory when requested by user programs, while ensuring mutual exclusion of the linked list data structure that keeps track of physical pages allocated.

loadSections() assigns physical pages to virtual pages and tracks assignments using a list TranslationEntry obkects; checks readOnly on coff; and loads coff onto pages.

readVirtualMemory() calculates the vpn and offset of the provided vaddress, looks up the corresponding entry in the page table, verifies there is space remaining, then reads from the proper page, and continues reading from the next allocated page if needed.

writeVirtualMemory() performs essentially the same steps as readVirtualMemory, but writes to memory rather than reading.
```


Psuedocode for `loadSections()`  function

1. Checks memory availability.
2. Allocates page table entries.
3. Maps virtual → physical pages.
4. Loads code/data into memory.
5. Clears unused memory.
6. Fills out the TranslationEntry[] page table for the process.

#### Load sections
Sets up the page table for a user process, assigns physical pages to virtual pages and we handle edge cases such as insufficient memory. PageTable is an array of `TranslationEntry[]` aka PTEs, holds metadata of the VPN and its PPN plus bit info as well.

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
	- load the .coff section of the file into in memory
	- clear unused VPN pages if it is not coff
close coff file
```

#### ReadVirtualMemory + Write Virtual Memory
Read data from this process's virtual memory and load it into the local array.

readVirtualMemory() calculates the vpn and offset of the provided vaddress, looks up the corresponding entry in the page table, verifies there is space remaining, then reads from the proper page, and continues reading from the next allocated page if needed.
***Psuedocode for ReadVirtualMemory***
```
set variables:
- total : tracks total amount of bytesRead
- currVaddr : tracks first byte of virtual memory to read
- arrOffset : offset to write to in the local array
- memory : grabbing the memory via Machine.processor

- while we still have bytes to read in from Virtual memory
	- calculate the VPN and the offset numbers
	- Access virtual page via pageTable[vpn]
	- set byte to true, saying that it is accessed
	- find the remaining bytes left to read from page
	- find the remaining bytes read across all virtual pages of that process
		- take the minimum between those, remember we are chunk reading. not reading eveything all at once
	- then take the min(bytesToRead, min(remainingBytesPage, remainingOverall))
	- Then copy the specified bytes into the local array
	- correctly update following variables
		- Decrement bytesToRead, decreasing each iteration
		- increment offset
		- Increment total bytes read
		- increment the current virtual address, ensuring that we are moving alongside the virtual pages currently
- return total bytes read
```
***Psuedocode for WriteVirtualMemory***
- Its essentially the same as the psudeocode above, except we are now writing to it. No big difference except we are writing into our local array from virtual memory instead of the other way around with ReadVirtualMemory


### Task 3

```
For this task I implemented the handleExec, handleExit, and handleJoin syscall functions. The exec function executes the given file name. The exit syscall frees up memory, closes file descriptors, and terminates a process. Lastly, the join function blocks until the child process finishes before the root/parent continues. I also altered the halt syscall to ensure that only the root process could call it directly. To ensure that each syscall worked properly I wrote some tests
```

##### ***handleExec***
Exec function executes given file name as a "new child process".
*flow*
- "us", the parent process calls `exec("example.coff", args)` -
	- `handleExec` reads in the file + given argument strings
	- creates a new `UserProcess` object(new child process) and inserts it into:
		- children : keeps track of current children processes (this is a hashmap)
		- childrenStatus : keeps track of the status of current children processes(hashmap)
			- This keeps track of if its running or not I am pretty sure
	- executes the newly created process
	- we return PID of the new child to parent process

***IMPORTANT NOTE***
- In `exec`, we only make the child process start running and then we return childPID asap, we do not wait for it to finish
- The child runs concurrently, doing its own thing.
- The parent and child can now run in parallel, unless the parent explicitly waits.
Calling exec("program") is like saying:
"Hey OS, start running this new program in the background — let me know how it goes later."

***Psuedocode***
```
- read in file name
- read in arguments
- create a new child process - tracked via a hashmap <Int, UserProcess> and also the status is tracked via hashmap <Int, Int>
	- assign the parent as the current root process, get the PID of child process
- make the child process start executing
	- if it fails to start executing, return -1
- return childPID ASAP
```

##### ***handleJoin***
Makes the parent process wait for a child process to finish before executing. We use the `join` method implemented in the KThread class

Function takes in a specified childID, looks for it in the childprocess hashmap and calls join.

```
- take the given childID, find the child process in the hashmap via ID
	- check if the process we grabbed exists, otherwise return -1
- call join on the child process we grabbed
- Once its done executing:
	- Remove it from children hashmap
	- remove it fromc current childrentStatus as well
- Return 1 indicating that children has sucessfully ran
```

##### ***handleExit***
Clear out the fileDescriptors belonging to the file process as well as any children associated with the process. 

```
- close the file descriptors, nulling them out
- unlink all child processes via iterating through children hashmap
- if processes have reached 0, we terminate the process
- mark the process as finished running
- return 0
```


