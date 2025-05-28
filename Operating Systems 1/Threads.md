- [[#Threads]]
	- [[#Processes Vs Threads]]
		- Process : Address space and resources
		- Threads : Sequential execution stream within a process(PC, SP, registers)
	- [[#Address Space]]
	- [[#Thread Control Blocks]], [[#Thread State Graph]]
	- [[#TCBs and Hardware State]]
		- [[#Thread Queues]]
	- [[#Thread Scheduling]]
		- [[#Non-Preemptive Thread Scheduling]]
		- [[#Preemptive Thread Scheduling]]
	- [[#Kernel Level Threads]]
	- [[#User Level Threads]]
	- [[#Combining Kernel and User-level Threads]]
	- [[#Three Multithreading Models]]
- [[#Processes and Threads Questions]]
- [[#Why Use Threads?]]
- [[#Threaded Creation Examples]]
	- [[#In-Depth Trace Through Of Diagram On Why Counter Did not correctly increment]]
	- [[#Use Of Atomicity]], help's solve the errors covered in examples

### Threads
Sequential execution stream within a process(PC, SP, registers). 
- Another definition that is more intuitive to understand:
	- a single sequential flow of activities being executed in a process
#### Processes Vs Threads
Modern OSes(windows, linux, OS X) separate the concept of processes and threads
- Process : Address space and resources, a program in execution
- Threads : Sequential execution stream within a process(PC, SP, registers). a single sequential flow of activities being executed in a process
Each thread is bound to a single process, process can handle multiple threads
Threads are used and they cooperate between each other in multithreaded programs
- To share resources, access data structures(example can be threads accessing a memory cache in web server)
- Coordinate their execution(example where some threads may depend on each other)
1. Processes are a more sound choice for logically seperate tasks
Threads become the basic unit of scheduling, where processes act as containers in which threads execute
![[Screenshot 2025-04-15 at 5.01.40 PM.png|200]]


#### Analogies / Example:
***House and people within the house:***
House = Process
- The process is like a house:  
- It has its own space (memory, resources).
- It’s isolated from other houses (other processes).
- You can run many houses independently on a street (like the OS running multiple processes).
People = Threads
- Threads are like people living in the house:  
- They share the house’s resources (kitchen = memory, bathroom = I/O, etc.).
- They can work together or do different tasks at the same time (multithreading).
- Communication is easy because they share the same space.

***Google chrome:***
Google Chrome = Process
- When you open Chrome, the operating system creates a process for it. This process has its own memory space, open files, etc.
- This is like launching a new house (from our earlier analogy).

Threads in Chrome = Tabs, UI rendering, background tasks
Chrome is multi-threaded, so within the Chrome process, you’ll find multiple threads doing different things:
- One thread might handle the user interface.
- One thread might handle JavaScript execution on a webpage.
- Another thread might be fetching network data.
- One thread might be handling tab rendering or animations.
They all share the same memory space and resources of the Chrome process, but work concurrently on different tasks.
Also, fun fact:
- Modern Chrome actually launches multiple processes, not just threads.  
- Each tab may run in its own process (for stability/security).
- And each of those processes may still have multiple threads inside them (for parallelism).

**£And with being concurrent, there all active and doing things at the same time but there not exactly doing the same thing which is what separates it from parallelism***
#### Address Space
![[Screenshot 2025-04-15 at 5.02.56 PM.png|250]]

Multi threaded:
![[Screenshot 2025-04-15 at 5.03.26 PM.png|250]]

#### Thread Control Blocks
![[Screenshot 2025-04-15 at 5.04.02 PM.png|450]]
##### Thread State Graph
![[Screenshot 2025-04-15 at 5.04.38 PM.png|400]]
##### TCBs and Hardware State
![[Screenshot 2025-04-15 at 5.05.39 PM.png|400]]
##### Thread Queues
![[Screenshot 2025-04-15 at 5.10.10 PM.png|400]]
![[Screenshot 2025-04-15 at 5.10.36 PM.png|400]]

#### Thread Scheduling
When should we stop running one thread and switch to another? There are different types of how threads give up and switch via preemptive vs non-preemptive scheduling
##### Non-Preemptive Thread Scheduling
![[Screenshot 2025-04-15 at 5.12.24 PM.png|400]]
###### Yield
![[Screenshot 2025-04-15 at 5.13.20 PM.png|400]]
![[Screenshot 2025-04-15 at 5.13.54 PM.png|400]]

###### Thread Context Switch
![[Screenshot 2025-04-15 at 5.14.27 PM.png|400]]

##### Preemptive Thread Scheduling
![[Screenshot 2025-04-15 at 5.15.45 PM.png|400]]
![[Screenshot 2025-04-15 at 5.16.00 PM.png|400]]
###### Web Server Example Process Vs Threads
![[Screenshot 2025-04-15 at 5.19.08 PM.png|400]]
![[Screenshot 2025-04-15 at 5.19.20 PM.png|400]]

#### Kernel Level Threads:
![[Screenshot 2025-04-15 at 5.23.53 PM.png|400]]
![[Screenshot 2025-04-15 at 5.24.16 PM.png|400]]
![[Screenshot 2025-04-15 at 5.24.47 PM.png|400]]
#### User Level Threads:
![[Screenshot 2025-04-15 at 5.29.30 PM.png|400]]
![[Screenshot 2025-04-15 at 5.29.58 PM.png|400]]
#### Combining Kernel and User-level Threads
![[Screenshot 2025-04-15 at 5.30.30 PM.png|400]]

#### Three Multithreading Models
- Many-to-one : many user-level threads mapped to a single kernel thread. used in user-level threads
- One-to-one : each user thread maps to a single kernel threads, used in kernel-level threads
- Many-to-many : allows many user-level threads to be mapped to many kernel threads, used in user-level threads
![[Screenshot 2025-04-15 at 5.31.21 PM.png|350]]
![[Screenshot 2025-04-15 at 5.31.55 PM.png|350]]

#### Processes and Threads Questions
![[Screenshot 2025-04-15 at 5.32.42 PM.png|450]]
#### Why Use Threads?
1. You can parallelize  your programs, allowing for faster program runtimes.
	- Examples include adding another thread to a program that deals with adding large arrays.
2. Avoid blocking program progress due to slow I/O operations. For example, say your program performed different types of I/O, some states may be stuck in "waiting". During this time is when having other thread(s) would come in handy and perform other important operations at the same time!
	- Using threads is a natural way to avoid getting stuck; while one thread in your program waits (i.e., is blocked waiting for I/O), the CPU scheduler can switch to other threads, which are ready to run and do something useful
	- Web servers, database management systems, make use of threads in their implementations
***Key Reasons To Use Either Threads / Processes***
- Processes are more sound for separate tasks that require little sharing of data
- Threads share an address space and thus make it easy to share data between each other, which is why the examples I listed like database management, working with large array programs make use of threads even though they technically can be done with processes as well.
	- ***Key connection*** [[PA3 Tiled Matrix Multiply]], [[PA6 CNN Tiled]], here the utilization of threads really helped speed up the program. For example with matrix multiply, 
		- we started with the sequential implementation
		- then the threaded implementation where the computations ran in parallel. each thread -> responsible for performing computations for an entire row and entire col
		- Then the final optimization specifically for tiled matrix multiply was making use of each thread sharing data between each other via utilizing local memory in which each thread was responsible for loading their assigned item into local memory. So when we ran partial dot product in our tiled implementation, the threads could access the other elements in their row from local memory without having to worry about loading them in.
			-> Each thread only had to load one item from the array into local memory instead of the regular threaded implementation where it did the whole row / col

#### Threaded Creation Examples
![[Screenshot 2025-04-15 at 1.03.11 PM.png|400]]
- This example will give non-deterministic results, this is due to the CPU scheduler being random.
![[Screenshot 2025-04-15 at 1.04.52 PM.png|400]]
Here, the results are also non-deterministic. It should produce 20,000,000 output for the counter variable however it does not. It varies every run. Once again the reason is due to "uncontrolled scheduling".

For example, at some points the counter should increment by 1 but due to interrupts such as a timer interrupt, it does not get incremented at all. This is due to how OS saves the state of the currently running thread and how it switched between thread 1 and 2.

Reading pages below explaining why it happens:![[Screenshot 2025-04-15 at 1.08.44 PM.png|400]]![[Screenshot 2025-04-15 at 1.09.02 PM.png|350]]

#### In-Depth Trace Through Of Diagram On Why Counter Did not correctly increment:
![[Screenshot 2025-04-15 at 1.11.13 PM.png|400]]
***Key Terms***
- Race condition : results depend on timing of codes execution. Context switches that occur at untimely points in execution, such as an interrupt occurring before `eax` could be moved back into T1 memory address.
- Critical Section : section of code where multiple threads are executing it and it can possibly result in a race condition.
- Mutual Exclusion : Property that ensures that only 1 thread is running in the "critical section". Essentially a check to see if theres a thread in critical section, if there is then any other threads are blocked from running the threads. 

#### Use Of Atomicity
The idea behind making a series of actions atomic is simply expressed with the phrase “all or nothing”; it should either appear as if all of the actions you wish to group together occurred, or that none of them occurred, with no in-between state visible. Sometimes, the grouping of many actions into a single atomic action is called a transaction, an idea developed in great detail in the world of databases and transaction processing

We can use this to fix the problem of interrupts happening mid computation executions.
For example, what if we had a super instruction that looked like this:
```
memory-add 0x8049a1c, $0x1
```

Assume this instruction adds a value to a memory location, and the hardware guarantees that it executes atomically; when the instruction executed, it would perform the update as desired. It could not be interrupted mid-instruction, because that is precisely the guarantee we receive from the hardware: when an interrupt occurs, either the instruction has not run at all, or it has run to completion; there is no in-between state. 

Essentially, the `memory-add` instruction executes these 3 all at the same time or it doesn't at all.``
```
mov 0x8049a1c, %eax 
add $0x1, %eax 
mov %eax, 0x8049a1c
```