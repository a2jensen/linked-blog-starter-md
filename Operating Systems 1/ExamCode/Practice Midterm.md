#### Key Vocab Terms
***Procedure Call:*** A **procedure call** (or function call) is when a program **invokes a subroutine** (i.e., jumps to a block of code) and expects it to return control back to the point where it was called.  
Example:
```
int x = add(3, 5);  // calls the 'add' procedure
```
***Test and set instruction:*** A **test-and-set** is an **atomic hardware instruction** used for synchronization.  
It does two things as a single atomic action:
1. Returns the old value of a memory location.
2. Sets that memory location to `true` (or `1`).
This is often used to implement **locks or spinlocks**.
***Starvation:*** Starvation happens when **new processes keep jumping ahead of others indefinitely**, which **doesn't** happen in FIFO or Round Robin.

***Question 1***

a. Yes, test-and-set instruction modifies the value of the lock
b. idk
c. I am going to say FIFO, starvation from my memory refers to when threads are being starved of resources(i.e. they have been waiting a long time). Consider that the first thread in the queue takes a long time to run, other threads waiting would be starved
d. Sometimes
e. Sometimes


***Question 2***
a. Interrupt
b. Exception
c. Interrupt
d. Exception
e. Neither
d. Exception


***Question 3***
a. 
User kernel threads
Pros:
- Applications may have more control of what to do
- Applications may run faster due to threads being on the application side
Cons:
- OS may make untimely interrupts to the threads
- Scheduling for the threads won't be as precise. Sometimes the OS may schedule threads to run when other's that are waiting may be of higher priority

Kernel threads
Pros:
- OS has more control, better chance of properly scheduling the threads

Cons:
- Applications may run slower I would assume, since thread

b.
Reminder of system calls: They come from the application side I think

It would add a pro to the usage of kernel-level threads, now applications may reap the benefits of using kernel-level threads while ensuring that their applications run's as fast as using user-level threads.

***Question 4***
a. 
We can use a scenario where 1 car, x, is driving to the north side.
We have 1 car, y, waiting to travel to the north side
We can say that as soon as the 1 car arrives, a car, z, immediately on the other side of the bridge executes depart()

Walkthrough of arrive(when cars arrive at the bridge):
- Called when cars arrive at the bridge
	- First while loop will make the car wait I assume if there is a car crossing the bridge from the opposite direction
	- Once we break out(in other words bridge is free to cross)
		- We increment num_cars += 1
		- We set cur_dir to our direction

Walkthrough of depart(when cars I assumed crossed the bridge already and are leaving it)
- decrement number of cars -= 1, representing 1 car leaving the bridge
- We check if there are any cars on the bridge, if theres none then we can set the current direction to open, ensuring that any waiting cars can go ahead

a.
Now, let's say we have 1 car departing the bridge
1. Lets say we had car arrive with cur_dir = North
2. No cars are on the bridge, so cur_dir = north and num_cars = 1. The car is crossing now in the north direction
3. Lets say we had a car arrive with cur_dir = South
4. It will be stuck busy-waiting in the while loop due to the other car crossing the bridge, stuck at `A.0`
5. Consider this, the car driving north gets across and now calls Depart()
	1. It decrements cars to 0 and sets the cur_dir to open
	2. Right at line `D.2`,  the car heading south that was busy waiting  at `A.0` gets out of the while loop since cur_dir is set to open. HOWEVER, at the same exact time another car arrives again that wants to head north
	3. Here the car heading north, at the same time as the car south, will pass through the while loop at `A. 0`
	4. This results in both cars arriving at both ends of the bridge to be in section `A.1`, here the number of cars will increment to 2
		1. Now, at `A. 2` one of the cars will mark the cur_dir as their direction and head out. Then the following car will mark the cur_dir as their direction and head out. This results in 2 cars now crossing the bridge from opposite directions

b.
We can place locks in `A1 - A2`for arrive to create a mutual exclusion zone/ critical section, so that only 1 car can be "leaving their entrace of the bridge"

We can place locks in `D0 - D4` for depart to ensure that cars leave in a sequential sequence. since it was specified there was only 1 lane.

Not fully sure, but I assume we can place condition variables within `A0 / D1-D2` sections. In the case where cars cannot enter the bridge on arrival, we can initialize a condition that will put a car to sleep. It will get waken up when the car in depart section `D1 - D2`wakes the sleeping car up. The condition variable can be implemented through using binary semaphores.

c.
```arrive
{
A.1
A.2
}
```

```depart
{
D0
...
D4
}
```

d. 
Well I think it depends on how the condition variable is implemented, say we had the condition variable implemented with a FIFO queue.
1. Consider the case where 100 cars arrive at the south side of the bridge looking to head north. Lets say we had 1 car then arrive on the north side of the bridge looking to head south, well that 1 car would be the very last in the queue and it would have to wait for the 100 cars to pass before going. This can lead to starvation.

e.
No because while crossing the bridge a car is never gonna be requesting a resource that any waiting cars are also requesting on. I assume there is not gonna be a deadlock.


***Question 5***
a.
- Ignore priorities
- New thread A is created
	- Print `fee`
	- Fork `A`, printing `foe`
	- Main thread `threadtest`, calls join on thread `A`
		- thread A now needs to run to unblock `ThreadTest`
			- new thread created,`B`, we print `foo`
			- fork `B`, print `far`
			- Call join on `B`
				- `B` runs, printing `fie`
			- `A` finishes running, prints `fum`
	- Main thread finishes, printing `fun`

b.
- Account priorities, Fork is what marks it ready to run.
- New thread A is created, priority of 10 is set on it, print `fee`
- Fork it and mark it as ready to run, since A has priority then it starts running
	- New thread B is created, priority 20 set, print `foo`
		- Fork it, since B has higher priority it starts running
			- B prints out `fie` and then returns
	- Return to thread `A`, prints out `far`
	- Call join on thread t2, however it has already ran so A continues and finishes, printing out `fum`
- We print `foe`
- Then we call join, and A is finished running so we finally print out `fun`

**Event.** An Event synchronization object has two methods, wait and signal, and an internal state that can have one of two values, _unsignaled_ and _signaled_. When created, an Event is in the _unsignaled_ state. When a thread calls wait, it blocks if the Event is _unsignaled_ and otherwise returns immediately. When a thread calls signal, it sets the Event to _signaled_ and wakes up all blocked threads. Once an Event is _signaled_, it remains _signaled_ until deleted.

Use pseudocode to implement Event:
```java
class Event {
   Condition cv = new Condition()
   Lock lock = new Lock()
   signal_state;
   void Event () {
     this.signal_state = "unsignaled"
     i think initialize new lock with it and new condition
   }
   void wait () {
    acquire lock
		if event is unsignaled:
			put thread to sleep via condition variable i think
		return
	release lock
   }
   void signal () {
     acquire lock
	set event to signal
		use condition variable to wake up all blocked threads
	while (signaled)
		busy-wait until its deleted
	 release lock
   }
 }
```

**CountdownEvent.** Microsoft .NET provides a synchronization primitive called a [CountdownEvent](https://docs.microsoft.com/en-us/dotnet/api/system.threading.countdownevent). Programs use CountdownEvent to synchronize on the completion of many threads (similar to [CountDownLatch](https://docs.oracle.com/en/java/javase/13/docs/api/java.base/java/util/concurrent/CountDownLatch.html) in Java). A CountdownEvent is initialized with a _count_, and a CountdownEvent can be in two states, _nonsignalled_ and _signalled_. Threads use a CountdownEvent in the nonsignalled state to _Wait_ (block) until the internal count reaches zero. When the internal count of a CountdownEvent reaches zero, the CountdownEvent transitions to the signalled state and wakes up (unblocks) all waiting threads. Once a CountdownEvent has transitioned from nonsignalled to signalled, the CountdownEvent remains in the signalled state. In the nonsignalled state, at any time a thread may call the _Decrement_ operation to decrease the count and _Increment_ to increase the count. In the signalled state, _Wait_, _Decrement_, and _Increment_ have no effect and return immediately.

Use pseudocode to implement a thread-safe CountdownEvent using locks and condition variables by implementing the following methods:

```java
class CountdownEvent {
  counter
  lock
  cv
  CountdownEvent (int count) {
	  initialize all variables
	  counter > 0 set to non signalled
	  otherwise, set to signalled
	  }
  void Increment () { 
	  acquire lock
		if counter == 0
			transition from signal to nonsignalled
		counter ++
		put thread to sleep via CV
	  release lock
 }
  void Decrement () {
	  acquire lock
		if counter == 1
			transition from nonsignal to signalled
		counter --
		wake up threads via cv
	  release lock
  }
  void Wait () {
	  acquire lock
		put thread to sleep via cv
	  release lock
  }
}
```