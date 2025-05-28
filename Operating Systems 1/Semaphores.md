- [[#Producer-Consumer Problem]]
	- [[#With Locks]]
	- [[#With Locks + Sleep/Wake]]
- [[#Semaphores]] - intro , benefits over locks
	- [[#Blocking Semaphores]]
	- Semaphore Types
		- [[#Semaphore Type Binary Semaphore]]
		- [[#Semaphore Type Counting Semaphore]]
	- [[#Producer-Consumer with Semaphores]]
		- Brute force and follow up solutions covered here
	- [[#Readers-Writers Problem with Semaphores]]
	- [[#Implementing Semaphores]]
- Lecture Questions

Key Vocab:
Semaphores: A synchronization variable that takes on non-negative integer values. It is literally just a counter that reflects the number of "resources" available (buffer slots, access tokens, etc.). The key difference with the semaphore is that you can't perform reads/write to it freely - protected by atomic operations
### Producer-Consumer Problem
![[Screenshot 2025-04-17 at 5.13.05 PM.png]]
***Examples***
Real-life example:
- Chefs produce pizza
- Waits "consume" pizza to deliver to customers
- Limited counter space to hold the food
Applying to the context of OS:
- Memory pages
- Disk blocks
- I/O

![[Screenshot 2025-04-17 at 5.14.25 PM.png|400]]

***First Solution***
![[Screenshot 2025-04-17 at 5.16.02 PM.png|400]]
Whats wrong with this is multiple threads can insert within the buffer at the same time...potentially leading to overflow in the buffer.

Naive solution: Problem of counter being overflowed
Locks solution: Do not provide ordering/sequencing, how does the producer know when to stop producing? How does the consumer know when it can consume??
Locks with sleep/wake solution: Problem of potentially both producer/consumer being asleep
##### With Locks
One solution is to use ***locks***
![[Screenshot 2025-04-17 at 5.16.27 PM.png|400]]
Limitations do arise though,
- Locks provide mutual exclusion
- Locks do not provide ordering or sequencing... how does the producer know when to stop producing? How does the consumer know when it can consume??

##### With Locks + Sleep/Wake
***Followup solution : Add in sleep/wake to manage buffer capacity***
![[Screenshot 2025-04-17 at 5.18.40 PM.png|450]]
One problem that arises... when they both go to sleep.. then no further actions will be taken and that will result in the producer/consumer threads never waking up.


### Semaphores
-  A synchronization variable that takes on non-negative integer values 
- Semaphores in Operating Systems are **synchronization tools** used to manage **concurrent access to shared resources** — like memory, files, or devices — by **multiple processes or threads**.
- A **semaphore** is essentially a counter used to control access. You can think of it as a **traffic light**:
	- **Green (go):** The resource is free, and a process can proceed.
	- **Red (stop):** The resource is busy, and the process must wait.
-  ***Semaphores*** support two core operations: 
	▪ wait(): an atomic operation that waits for the semaphore to become greater than 0, then decrements it by 1 
		» Also P() after the Dutch word for “try to reduce” 
	▪ signal(): an atomic operation that increments the semaphore by 1 
		» Also V() after the Dutch word for increment 
	▪ Initialize the semaphore to some value 
	▪ Cannot read the semaphore’s value directly

***WHY USE SEMAPHORES and Benefits Over Locks***
- Avoid race conditions(multiple threads messing with shared data at the same time) and to ensure consistency
- Can be used to solve traditional synchronization problems
	- For example: [[#Producer-Consumer Problem]] and [[#Readers-Writers Problem with Semaphores]]
	- Enforces critical section(mutual exclusion), enable coordination between threads(scheduling)
- Semaphores have a value, enabling more semantics:
	- When at most one, can be used for mutual exclusion(1 thread in critical section)
	- When greater than 1, can allow multiple threads to access resources
***Drawbacks For Semaphores...***
- No coordination between the semaphore and the controlled data
- Used for both critical sections and coordination - this can be confusing
Whats the fix??? Covered in next topic/lecture [[Deadlock]]


#### Blocking Semaphores
![[Screenshot 2025-04-17 at 5.27.32 PM.png|450]]

###### Spinning and Blocking Version
![[Screenshot 2025-04-17 at 4.07.55 PM.png|300]]

#### Semaphore Type : Binary Semaphore
Remember, semaphores come in two types
- Binary semaphore
	- Represents access to a single resource
	- Guarantees mutual exclusion to a critical section
	- Has count = 1
	
![[Screenshot 2025-04-17 at 5.30.41 PM.png|450]]
- Semaphore behaves like a lock, they all execute sequentially and they depend on each other
![[Screenshot 2025-04-17 at 5.31.24 PM.png|450]]

#### Semaphore Type : Counting Semaphore
 - Counting Semaphore
	- Represents a resource with many units available
	- Multiple threads can pass the semaphore at once
	- Number of threads determined by the semaphore "count"
	- Has count = N

***Expanding the example in previous section, now with s = 2***
![[Screenshot 2025-04-17 at 5.31.50 PM.png|450]]
#### Producer-Consumer with Semaphores
- Added a count i.e. "semaphore"
![[Screenshot 2025-04-17 at 5.32.49 PM.png|450]]
![[Screenshot 2025-04-17 at 4.24.38 PM.png|450]]

***First version***
![[Screenshot 2025-04-17 at 4.26.14 PM.png|450]]
Producer waits until theres space, consumer waits until there's at least one item
- ***Wrong*** since both producer and consumer threads can manipulate buffer at the same time
What we can do to fix this:
- ***Add in constraint that only 1 thread can manipulate buffer at once***
- Add in ***Lock or semaphore*** - THIS SHOULD WORK

***Second Version - Lock Added / Semaphore Added***
![[Screenshot 2025-04-17 at 5.34.35 PM.png|450]]
What are the critical sections??  - Answer This
What if we call acquire before calling wait?? - Answer This
#### Readers-Writers Problem with Semaphores
![[Screenshot 2025-04-17 at 5.37.00 PM.png|450]]
![[Screenshot 2025-04-17 at 5.37.36 PM.png|450]]
![[Screenshot 2025-04-17 at 5.38.17 PM.png|450]]


#### Implementing Semaphores
![[Screenshot 2025-04-17 at 5.38.48 PM.png|450]]

##### Test-and-set
- **Test-and-Set** is a **hardware atomic instruction**. Atomically check and lock a variable (e.g., for spinlocks)
	- It **reads** a memory location and **sets it to true** in **one atomic step**.
- In semaphore, used to make sure P() and V() are atomic (safe critical section)
```c
function testAndSet(boolean *target):
    old = *target
    *target = true
    return old
```

- If `old == false`, you successfully "acquired" it (lock was free).
- If `old == true`, someone else already has it, so you must spin or wait.
So when we **implement semaphores** (P and V operations),  
**test-and-set** is used to **atomically** control the update of the internal **value**.
Semaphores need:
- **P() (down / wait)**: decrement the count; if the count is negative, block.
- **V() (up / signal)**: increment the count; if there are waiting threads, wake one.
    
But to **safely** modify the **semaphore's value**,  
**P() and V() must be atomic** — **they cannot be interrupted**.
That's where **test-and-set** comes in:
- It is used to build a **spinlock** around P() and V(). 
- That **spinlock** ensures that **only one thread** at a time updates the semaphore's internal count.