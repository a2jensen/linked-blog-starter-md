TLDR, some of this was covered in previous :
- Concurrency and Q's
- [[#Problems That Come Up With Concurrency]]
	- [[#Interleaved Schedules Problem]]
	- [[#Source Of Concurrency Problems]]
		- [[#Shared Resources]]
- [[#Synchronization]] - keep threads in sync to coordinate actions correctly and ensure proper sharing of shared resources
	- [[#Mutual Exclusion]]
		- [[#Critical Section Goals]]
		- [[#Mechanisms For Building Critical Sections]]
		- [[#Locks]]  
			- [[#Implementing Locks(Attempt 1) Using Spinlock]]
			- [[#Implementing Locks(Attempt 2) Disabling Interrupts]]
			- [[#Implementing Locks(Attempt 3) Test And Set]]
			- [[#Problems With Spinlocks]]
				- [[#Summary With First Couple Attempts]]
			- [[#Implementing Locks(Attempt 4) Use Queue To Block Waiters]]
			- [[#Implementing Locks(Attempt 5) Use Queue To Block Waiters + Test and Set]]
- [[#Key terms For This Topic]], [[cse120/Operating Systems 1/Operating Systems Home#Key Terms]] for all key course terms

### Concurrency and Q's
![[Screenshot 2025-04-15 at 5.17.39 PM.png|400]]
### Problems That Come Up With Concurrency

##### Interleaved Schedules Problem
![[Screenshot 2025-04-15 at 3.42.10 PM.png|400]]
![[Screenshot 2025-04-15 at 3.43.00 PM.png|450]]
This leads to a bank account balance of $400 instead of the intended $300.
With multiple threads, we end up with multiple cooperating threads and non-deterministic results. Here schedule ordering does not matter.

#### Source Of Concurrency Problems
![[Screenshot 2025-04-15 at 3.45.10 PM.png|400]]
##### Shared Resources
1. Local Variables - not shared
	- Refer to data on each thread's own stack
	- NEVER pass/share/store a pointer to a local variable on the stack between threads
2. Global variables / static objects - shared
	- Stored in the static data segment, can be accessed by any thread
3. Dynamic Objects and Other Heap Objects - shared
	1. Allocated from the heap with malloc/free or new/delete

### Synchronization
Idea of synchronization: keep threads in sync to coordinate actions correctly and ensure proper sharing of shared resources

- Interleaved executions and shared resources can lead to race conditions
	- Where : Results depend on timing execution of the code
- Critical Sections
	- Sections of code in which only 1 thread can be executing it
- Locks can solve this problem via mutual exclusion
	- Locks are useful, but they are limited. ***Locks only provide mutual exclusion***
		- With mutual exclusion, it does not solve all synchronization problems... we want other semantics for example:
			- Wait for shared resources to become available
			- Allow multiple threads to generate different resources
			- Use certain conditions to decide when to enter a critical section
			So an alternative to locks can be ... [[cse120/Operating Systems 1/Semaphores]]

### Mutual Exclusion
Also covered in [[cse120/Operating Systems 1/Threads]]
![[Screenshot 2025-04-15 at 4.02.11 PM.png|400]]
![[Screenshot 2025-04-15 at 4.02.38 PM.png|400]]

#### Critical Section Goals
![[Screenshot 2025-04-15 at 4.04.11 PM.png|400]]
Three major goals for critical section:
1. Safety Property
2. Liveness
3. Performance property
![[Screenshot 2025-04-15 at 4.04.28 PM.png|400]]

#### Mechanisms For Building Critical Sections
- Atomic read/write
- Locks
	- Primitive, minimal semantics, used to build others
- Semaphores and condition variables
	- Basic, easy to get the hand of, harder to program with
- Monitors
	- High-level, requires language support, operations implicit
- Messages
	- Atomic Transfer of data across a channel
	- Direct application to distributed systems

#### Locks
![[Screenshot 2025-04-15 at 4.11.58 PM.png|400]]
In the case where we forgot either acquire or lock, then this can result in threads never getting to enter the critical section and the thread within the critical section to stay there
***Summary***
- Use a queue to block waiters
- Leave interrupts enabled within the critical section
- Use disabling interrupts and/or spinning only to protect the critical sections within acquire/release 

##### Using Locks Example
![[Screenshot 2025-04-15 at 4.14.55 PM.png|600]]

- When orange tries to acquire lock, it is blocked until blue thread finishes and releases lock.
- `return balance` is good to have outside of critical section due to it being:
	- Balance is a local variable, so we want to follow practice of not sharing it among other threads
	- Returning outside of the critical section ensures that the "final" balance is returned after all threads did their calculations

#### Implementing Locks(Attempt #1) : Using Spinlock
![[Screenshot 2025-04-15 at 4.23.01 PM.png|500]]
No this does not work, two independent threads may notice that a lock has been released and then require it
- Say 2 threads were in the `while(lock->held)`, the moment a lock is released the 2 threads would then enter the critical section

***How To Fix?***
We can make the implementation of acquire/release to be atomic(i.e. all or nothing for code executions, no stopping in the middle)
How can we make a piece of code atomic?
![[Screenshot 2025-04-15 at 4.26.41 PM.png|400]]

#### Implementing Locks(Attempt #2) : Disabling Interrupts
![[Screenshot 2025-04-15 at 4.29.30 PM.png|400]]
There is no way for another thread to disable interrupts since it is not in control of the CPU, so it does technically work... 
	1. Connecting to [[cse120/Operating Systems 1/Project 1 Notes]], Nachos is a single core CPU simulation, so disabling interrupts is fine as there's no other CPU cores(single-CPU, multi-threaded). If it were a multi core CPU simulation, then it's possible for other threads running on other cores to still cause interrupts. Key thing to know!
***Negatives On Disabling Interrupts***
- On multi-core CPU's this implementation does not work, it disables interrupts for only your specific core the thread is in. So technically threads in other cores can still enter the critical section since interrupts were not disabled
- Prevent notifications of external events, which leads to no context switching happening
![[Screenshot 2025-04-15 at 4.31.47 PM.png|400]]

#### Implementing Locks(Attempt #3) Test And Set
![[Screenshot 2025-04-15 at 4.34.08 PM.png|400]]
- Value of flag afterword would be true
- Return of result is the old value, which is false.
![[Screenshot 2025-04-15 at 4.34.30 PM.png|400]]
- Thread will exit when the function returns False, will keep going as long as `held` is true. The value of held would be the `previous state`, so once we see the state is false then we exit.
- Yes, the memory is shared among all cores in the CPU
- Yes but there are limitations

### Problems With Spinlocks
![[Screenshot 2025-04-15 at 4.41.17 PM.png|400]]
### Summary With First Couple Attempts
![[Screenshot 2025-04-15 at 4.41.54 PM.png|400]]
![[Screenshot 2025-04-15 at 4.47.11 PM.png|400]]

#### Implementing Locks(Attempt #4) Use Queue To Block Waiters
![[Screenshot 2025-04-15 at 4.43.31 PM.png|500]]
This is how nachos implements locks. This is due to nachos using a single core CPU.
#### Implementing Locks(Attempt #5) Use Queue To Block Waiters + Test and Set
![[Screenshot 2025-04-15 at 4.45.13 PM.png|400]]
#### Key terms For This Topic
 - State : Either running, ready, waiting / blocked. relevant, [[cse120/Operating Systems 1/Processes#Process State]]
- Concurrency : tasks running at the same time
-  Race condition : results depend on timing of codes execution. Context switches that occur at untimely points in execution, such as an interrupt occurring before `eax` could be moved back into T1 memory address.
- Mutual Exclusion : Property that ensures that only 1 thread is running in the "critical section". Essentially a check to see if theres a thread in critical section, if there is then any other threads are blocked from running code within the "critical section". Example analogy can be a bathroom in an airplane, only 1 person(thread) can be inside the bathroom("critical section")
- PCB(process control block): Where we save state of processes too
- TCBs(thread control block): Stores the state of each thread(s) of a process
-  Race condition : results depend on timing of codes execution. Context switches that occur at untimely points in execution, such as an interrupt occurring before `eax` could be moved back into T1 memory address.
- Critical Section : section of code where multiple threads are executing it and it can possibly result in a race condition.
- Mutual Exclusion : Property that ensures that only 1 thread is running in the "critical section". Essentially a check to see if theres a thread in critical section, if there is then any other threads are blocked from running the threads. 