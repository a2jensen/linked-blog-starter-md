***Lectures***
- [[cse120/Operating Systems 1/Intro]]
	- [[cse120/Operating Systems 1/Memory, Disk, Cloud, and the CPU]] - notes on the high level of hardware components and user apps
	- [[cse120/Operating Systems 1/OS components overview]]
- [[cse120/Operating Systems 1/Interactions With Hardware and Apps]]
- [[cse120/Operating Systems 1/Processes]]
- [[cse120/Operating Systems 1/Threads]]
- [[cse120/Operating Systems 1/Synchronization]] - keep threads in sync to coordinate actions correctly and ensure proper sharing of shared resources 
	- [[cse120/Operating Systems 1/Synchronization#Mutual Exclusion|Mutual Exclusion/Mutex]] - *synchronization primitive* allowing only 1 thread in critical section. the other *sync primitive* is "coordination"
		-  [[cse120/Operating Systems 1/Synchronization#Locks|Locks]] - *tool* to implement the synchronization primitive mutual exclusion 
	- [[cse120/Operating Systems 1/Semaphores]] - *tool* to implement synchronization primitives, specifically mutex + coordination via counter variable
	- [[cse120/Operating Systems 1/Condition Variables and Deadlock]] - *tool* to implement synch primitives. used with locks to provide mutex + coordination via condition variable
- [[cse120/Operating Systems 1/CPU Scheduling]]
- Memory Management
	- [[cse120/Operating Systems 1/Memory Management Overview]]
	- [[cse120/Operating Systems 1/Paging]] - Intro to Page tables and the variations of them, PTEs
	- [[cse120/Operating Systems 1/TLBs and Swapping]] - Caching for PTEs and Demand Paging
	- [[cse120/Operating Systems 1/Page Replacement and Memory Allocation]]
- File System
	- [[cse120/Operating Systems 1/Storage Devices and File System API]]
	- [[cse120/Operating Systems 1/File System Disk Layout]]
	- [[cse120/Operating Systems 1/File Caching and Reliability]]

***Misc***
- [[cse120/Operating Systems 1/Project 1 Notes]]
- [[cse120/Operating Systems 1/Project 2 Notes]]
- [[cse120/Operating Systems 1/Project 3 Notes]]
- [[cse120/Operating Systems 1/ExamCode/HW 1]]
- [[cse120/Operating Systems 1/ExamCode/HW 2]]
- [[cse120/Operating Systems 1/ExamCode/HW 3]]
- [[cse120/Operating Systems 1/HW 4]]

***TLDR***
- [[#Paste]]
- [[#Key Terms + Ideas]]
- [[#Critical Section]]
- [[#Synchronization Primitive]]
	- [[#Mutex (Mutual Exclusion)]] - type of primitive
	- [[#Key Synchronization Primitives Summary]]
	- [[#Synchronization Real Life Analogies Example]]
- [[#Key Vocab Words]]
### Paste
Get into server

server SSH key PW: `BubstaMilo`

```
ssh account_@ieng6.ucsd.edu
cs120sp25
```

Important : REMEMBER TO ALWAYS DO GIT STATUS
```
```

Check of when we last submitted a proj

```
Run `ls -l proj0.tar.gz` in the nachos directory (the same directory that you submitted from) and look at the timestamp returned.
```

### Key Terms + Ideas

##### Concurrency
- In the context of operating systems, it refers to the ability for multiple tasks to be in progress at the same time even if they are not exactly executing all at the same time
	- See the [[cse120/Operating Systems 1/CPU Scheduling]], where on a single core CPU, the CPU deals with scheduling multiple threads to run - giving the illusion that one thread has full control over the CPU when in reality the CPU is constantly context switching between multiple threads
		- On a multi-core CPU, it is possible for multiple threads to truly run at the same time (in parallel), but they may still be executing different instruction paths, depending on the program’s flow and workload.

###### Concurrency Vs Parallelism
- Concurrency = multiple tasks in progress (may or may not be running at the same time)
- Parallelism = multiple tasks literally executing at the same moment on different cores

| Situation                                     | Concurrent? | Parallel? |
| --------------------------------------------- | ----------- | --------- |
| Threads on 1 core, interleaved (time slicing) | ✅ Yes       | ❌ No      |
| Threads on 2 cores, running simultaneously    | ✅ Yes       | ✅ Yes     |
| One thread running, others waiting            | ✅ Yes       | ❌ No      |

##### Synchronization Primitive
- Building block within the operating system that coordinates the execution of multiple threads at the same time, especially when accessing shared resources
##### Critical Section
- Part of code that accesses shared resources, and must not be executed by more than 1 thread at a time to prevent the problem of race conditions
##### Mutex (Mutual Exclusion)
- A **mutex** is a **synchronization primitive** or tool used to **enforce mutual exclusion** for a critical section.
- It ensures that **only one thread** can enter the critical section at a time by locking and unlocking it.

| Aspect             | Critical Section                   | Mutex                            |
| ------------------ | ---------------------------------- | -------------------------------- |
| What is it?        | Code block that must be protected  | Mechanism to protect the code    |
| Purpose            | Prevent concurrent access          | Enforce exclusive access         |
| Implemented using  | Mutex, semaphores, spinlocks, etc. | OS-provided synchronization tool |
| Language-specific? | No, it's a concept                 | Yes, actual object/functionality |

###### Key Synchronization Primitives Summary
- Mutual exclusion (preventing concurrent access)
- Coordination (ensuring proper ordering or signaling between threads)
These primitives are implemented via the tools below
1. Locks
	- Only provide mutual exclusion
2. Semaphores
	1. Provide mutual exclusion(binary semaphores)
	2. Enable coordination between threads(counting semaphores)
3. Condition variables
	1. Synchronization point to wait for events
	2. Used with locks or inside monitors
4. Synchronized execution using high-level language support

###### Synchronization Real Life Analogies Example
1. Crowded lab space. Goal: Only 8 people or fewer can be in the lab
	***What Primitive To Use : Counting Semaphores***
2. Too much talking, avoid multiple people talking at the same time
	 ***What to use: Lock or binary semaphore suffices, we only want 1 person in the "critical section"***
 3. Feeding the picky eaters
	 - Each kid wants to eat specific foods for dinner (e.g., kid 1: 1 cookie + 2 sandwiches, kid 2: 5 cookies + 1 carrot) 
	 - Cooks prepare food (e.g., 12 cookies, 30 carrots) 
	 - Goal: a kid can only take food when they can take their whole dinner at once-
	 ***What to use: Locks with condition variable would suffice. The condition variable would be utilized to check if a "kid is ready to take food", we combine it with locks to avoid the problem of multiple kids getting the same meal at the same time, leading to an error where there's not enough resources/meals to give. 
	 Condition variables can be used to help put the kids to "sleep"/"wait" when the cooks are still preparing. We can wake them when the food is ready to take....***

***Kernel***
core of an operating system, acting as an interface / bridge between hardware and software, managing resources, and enabling applications to run smoothly and securely

***Driver***
A driver is ==a software program that allows your computer's operating system to communicate with and control a specific piece of hardware==. Think of it as a translator, enabling the OS to send commands to the hardware and receive information back from it. Without the correct drivers, hardware devices may not function properly or at all.

***What is the shell?***
The shell is program that is the interface for the operating system. We can type in commands like (`ls cd git`) and runs them for you. It is the command interpreter, your personal assistant for interacting with files, running programs, and scripting.

You’re probably using one of these common shells:
- `bash` (Bourne Again SHell)
- `zsh` (Z Shell)
- `fish` (Friendly Interactive SHell)
- `sh` (original Bourne shell)

Example of what shell actually does when I type in `ls`:
Here’s what the shell does:
1. **Reads** your input.
2. **Parses** it (e.g., figures out what `ls` means).
3. **Locates** the `ls` program (usually in `/bin/ls`).
4. **Creates a new process** using `fork()` (except when you use `exec`).
5. In that new process, runs `ls` with an `exec()` call.
6. Waits for it to finish, then prompts you again.

### Key Vocab Words
- ***State*** : Either running, ready, waiting / blocked. relevant, [[cse120/Operating Systems 1/Processes#Process State]]
- ***Concurrency*** : tasks running at the same time
-  ***Race condition*** : results depend on timing of codes execution. Context switches that occur at untimely points in execution, such as an interrupt occurring before `eax` could be moved back into T1 memory address.
- ***Critical Section*** : section of code where race conditions can occur due to multiple threads accessing shared resources at the same time
- ***Mutual Exclusion*** : Synchronization primitive that ensures that only 1 thread is running in the "critical section", granting it access to the shared resources. 
	- A check to see if there is a thread in critical section, if there is then any other threads are blocked from running code within the "critical section". Example analogy can be a bathroom in an airplane, only 1 person(thread) can be inside the bathroom("critical section")
- ***PCB(process control block)***: Where we save state of processes too
- ***TCBs(thread control block)***: Stores the state of each thread(s) of a process
- ***Process*** : Address space and resources, abstraction for a program. a program in execution.
- ***Threads*** : Sequential execution stream within a process(PC, SP, registers). 
	- A more intuitive definition: a single sequential flow of activities being executed in a process

