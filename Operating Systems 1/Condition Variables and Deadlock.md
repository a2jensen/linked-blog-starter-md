***Key Vocabs***
- mutex = mutual exclusion
- condition variable

***Key Recap***
Mutual exclusion(mutex), certain sections of code, critical sections, should only have 1 thread allowed inside of it
- Locks are the first solution to it - need to have lock to enter
	- acquire(), release()
- Semaphores - counter that represents resources available
	- 
- Condition variables 
	- Producer-Consumer With Condition Variables
	- Signal Semantics - Mesa Vs Hoare
		- Common practice to use Mesa semantics usually
		- Producer-Consumer with Condition Variables(Signal Semantics)
	- Common Pitfalls with Condition Variables
		- 1. Cannot be tested, it is a data structure
		- 2. Do not call release before wait. Wait automatically handles logic for releasing and reacquiring the lock
		- 3. Need to hold the lock while testing the condition
- Monitors
	- Producer-Consumer with a monitor
- Deadlock - summary here
	- Definitions and Conditions
	- Strategies for Dealing With Deadlock
	- How To Prevent / Avoid
	- How To Detect and Recover

### Deadlock
Incorrect use of synchronization can block all threads
![[Screenshot 2025-04-22 at 4.28.44 PM.png|450]]

#### Definition and Conditions
![[Screenshot 2025-04-22 at 4.29.50 PM.png|450]]
![[Screenshot 2025-04-22 at 4.31.52 PM.png|450]]

###### Conditions RAG(Resource Allocation Graph)
![[Screenshot 2025-04-22 at 4.33.36 PM.png|450]]
![[Screenshot 2025-04-22 at 4.37.51 PM.png|450]]
- ***Left Graph***
	- There is a deadlock Every resource is asking for something a lock/semaphore that is being used by another resource
- ***Right Graph***
	- It is not a deadlock. There exists a cycle but theres no deadlock
	- Why?
		1. **T4** is not waiting for any resource and can execute immediately, releasing **S2**.
		2. Now **T1** can get **S2**, run, and then release **S1**.
		3. **T3** now has access to **S1**, runs, and releases **L2**.
		4. **T2** now gets **L2**, runs, and potentially releases **S1/S2**.
		5. There’s no circular blocking because **at least one thread can proceed at each stage**.