- [[#Key terms / ideas to know]]
- [[#High Level Overview Of Classes We Edited]]
- [[#KThread SelfTest() Tracethrough]]
- [[#Alarm Class]] - Task 1
- [[#KThread Class]] - Task 2
- [[#Condition2 Class]] - Task 3
	- [[#Difference Between Condition and Condition2]]
		- [[#Table Of Differences Between Semaphore and Interrupt]]
	- [[#Recap Of Condition Variables]]
- [[#Personal Quick Summary Of Classes After Project Was Finished]]
	- [[#Difference Between KThread `sleep()` and Condition `sleep()`]]

NOTE AS OF 4/26: The actual logic for fork and join in nachos is that fork just marks a thread as ready and adds it to the ready queue. The join function is the function which actually runs the threads! So some of the notes below may get the wrong logic so please fix them if you find.

###### Key terms / ideas to know:
- How Threading Works In Nachos:
	1. The program starts with a single thread of execution - main thread
	2. Any additional child threads must be created within the functions running within the main thread
	Applying this to selfTest(), ***main thread is selfTest***, ***child thread(s) is created within selfTest through `new KThread()`***
- Fork : Creates a child process/thread and then runs it(how it's commonly done)
	- ***NOTE FOR FORK() in NACHOS*** , `fork()` activates the thread and makes it eligible to run. The creation of a thread is done with the KThread constructor. So it is done slightly differently in where:
		1. We create thread object `new KThread()`
		2. Call `fork()` to add it to the ready queue and mark it as ready. It may or may not run right away, it depends on the scheduler.  It does not immediately run — it only runs when the scheduler picks it (e.g. after a Yield(), I/O wait, or when the current thread finishes or when join() called)
- Yield : Context switches and runs another thread
- Join : In nachos, parent threads wait for another thread to finish before continuing on
- PCB(process control block): Where we save state of processes too
- TCBs(thread control block): Stores the state of each thread(s) of a process
### High Level Overview Of Classes We Edited
##### KThread Class
- Class that handles thread implementation within a single core CPU simulation
	- `fork()` marks a thread as ready to run
	- `join()` is used by the parent to first let any child threads run and finish before continuing on. It is called on the child threads initialized within the main thread, the main thread is blocked if a child thread is running and it will only run until after the child thread is finished.
		- Claude description as well: method that blocks the calling thread until the thread it's called on has finished execution
			- In the test example [[#KThread SelfTest() Tracethrough]], within the main thread(`selfTest`) we create the child thread `child`, fork it -> `child.fork()`, and then we call `child.join()`. On `join()` call, we wait for `child` to fully run before advancing anywhere else in the function!.
	- Other functions automatically provided: (creation, execution, and termination) for the lifecycle of thread
	- provides the fundamental `sleep()` and `ready()` methods that directly control thread state
##### Condition2 Class
- Handles implementing condition variables using enabling/disabling interrupts and a queue that manages threads that are waiting. Refer to [[#Recap Of Condition Variables]]
	- The queue is called `waiter_queue`, this manages waiting/sleeping threads and when they are pulled off the queue they are marked ready to run.
		- Difference between `sleepingThreads` queue in `Alarm`, this queue tracks threads waiting for a specific condition to become true
		- *Condition1/2 queue is about thread coordination based on program logic (waiting for a condition that another thread will signal)*
	- Interrupts are used directly in the functions for atomicity
***KEY TAKEAWAY: queue to manage when threads are sleep and when there ready to run, direct use of interrupts for atomicity***
##### Alarm Class
- Handles the functionality of OS ensuring control over threads with a timer. Specifically with allowing threads to sleep up to a certain time and ensures that 1 thread doesn't completely hog up the CPU I assume. NOTE: The Alarm class handles timer functionality for threads. The timer interrupt is always initialized and running in the background(context switching occurs i assume), but threads only interact with it when they explicitly call methods like waitUntil() to sleep for a specific duration.
	- We have a queue `sleepingThreads` within `Alarm` class which keeps track of sleeping threads.
		- This queue keeps track of threads waiting for a specific time to elapse before waking up again
		- *Alarm's queue is about time-based scheduling (waiting for a specific amount of time to pass).*
	- `timerInterrupt()` function which is the timer interrupt handler. Called by the machines timer every 500 clock ticks and ensures that the current thread yields and forces a context switch to a thread in the queue that should be run
	- `waitUntil`, put thread to sleep until a certain amount of ticks
	- `cancel(thread)` cancel threads timer, essentially waking it up and getting it off the sleeping queue


### KThread SelfTest() Tracethrough
This test demonstrates the lifecycle of a thread and how different threads interact with each other at the same time through context switching

The test is designed to demonstrate two main things:
- thread context switching through the `yield()` mechanism. This is the `PingTest` section where main thread is `PingTest(0)`, and forked thread is `PingTest(1)`. Shows how scheduling works between threads, and how the queue works
- join functionality, where it handles running threads, and this test shows how the main thread is blocked until the child thread is finished running. demonstrated in the `joinTest()` function call

Both the `PingTest()` and `joinTest()` section sort of have the same functionality of context switching constantly between the main thread and child/forked thread. Difference with `joinTest()` is with the added `join` call, which deals with the main thread waiting on the child thread to finish before going on.


**Tracethrough Of SelfTest()**
3 total threads setup, and essentially this test constantly context switches between running each thread

*Brief 3 Step Overview*
1. The main thread (running `selfTest()` itself), 
2. The "forked thread", setup within `selfTest`and is called `forked thread`
We alternative between these threads due to functionality of `PingTest()` which is called on via the the main thread and child thread
Afterwards we call `joinTest1` and within it:
3. We create another thread `child1` and we call `fork` on it , marking it as ready to run. After when which alternatives between the main thread and `child1` within `JoinTest()`

*Full Walkthrough Line By Line*
`selfTest()`
```java
new KThread(new PingTest(1)).setName("forked thread").fork();
```
- New KThread is created and named "forked thread", and then `fork()` is called which in nachos marks the new thread we made as ready to run.
- PingTest object is created with which=1, which is tied to the `forked thread`
At this point 2 threads exist. The main thread and the child/forked thread.
```java
new PingTest(0).run();
```
- PingTest(0) is called
	- Runs the main thread, `selfTest()` as `PingTest(0).run()`
		- Calls run() within PingTest, 5 iterations are made
			- During each iteration, we print to the terminal and then we YIELD, causing a context switch to the`forked thread`. Here PingTest(1) is called
			- Same occurs for PingTest(1), we print within the `forked thread` and then context switch to the main thread
The log looks something along the lines of
```
*** thread 0 looped 0 times
*** thread 1 looped 0 times
*** thread 0 looped 1 times
*** thread 1 looped 1 times
...
*** thread 0 looped 4 times
*** thread 1 looped 4 times
```

Next we make a function call to:
```java
joinTest1();
```
Here we create a new thread `child1`. we call `fork()` which doesn't create a child of `child1` - it activates `child1` itself by marking it as ready and adding it to the queue. 

We alternate between running the main thread and child thread, `child1`:
- JoinTest1() is called - psuedocode below of the function:
	- creates a new thread `child1` 
	- Call fork() on it, which adds it to the ready queue. Allows it to be scheduled and run
	- The main thread (which is running `joinTest1()`) then enters a busy-wait loop where it yields 5 times to give `child1` a chance to run:
		- The main thread prints `busy`, then `yield()` context switching to `child1`
		- `child1` prints `I love nachos`, finishes and then since its finished it context switches back to main thread which is on the queue ready to run
		- This process repeats over and over until 5 iterations are made
	- Afterwords, the main thread calls `join()` on the `child1` thread,
		- Here, it depends on the status of the child thread but:
			- If `child1` is still running, it prints `I love nachos` again, after it finishes and returns
			- If `child1` is finished running, main thread can run
	- Main thread can continue on to the next line, where we print out to the terminal and check if the thread `child1` is finished.



### Alarm Class
[Github PR](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/commit/65960dc784a0979483b35ac1f58a22223ee03bc2)

### KThread Class
[Github PR](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/commit/53d0d5752d26fab9427cfdceda9f133439d713e2)

### Condition2 Class
Implementing these 3 functions below:
- wait(): release the lock, go to sleep, wake up and re-acquire the lock when signaled (releasing the lock and going to sleep is atomic) 
	» Also sleep() in Nachos 
-  signal(): wake up a waiting thread, if any 
	» Also wake() in Nachos or notify() 
-  broadcast(): wake up all waiting threads, if any 
	» Also wakeAll() in Nachos or notifyAll()
 - sleepFor(): release the lock, go to sleep, wake up and re-acquire the lock when signaled or when the timer goes down to zero (releasing the lock and going to sleep is atomic) 
	» we use the 
#### Difference Between Condition and Condition2
The key difference between the implementation given to us:
1. In the original Condition class:
    - Implemented using semaphores
    - Each waiting thread creates its own semaphore with an initial value of 0
    - The P() operation blocks the thread until the semaphore's value is incremented
    - The V() operation increments the semaphore, unblocking the thread
2. In your Condition2 class:
    - Implemented directly using interrupt handling
    - Threads are directly added to a wait queue and put to sleep
    - Wake operations make threads ready again
    - Uses interrupt disable/restore to ensure atomicity instead of relying on semaphores
***Rather than using semaphores as intermediaries that control when threads block and wake, you're directly managing the threads in a queue.***

##### Table Of Differences Between Semaphore and Interrupt

| Aspect                   | Semaphore-based (Condition)                                            | Interrupt-based (Condition2)                      |
| ------------------------ | ---------------------------------------------------------------------- | ------------------------------------------------- |
| sleep(), blocking method | Thread creates a semaphore with initial value 0                        | Thread is directly added to waitQueue             |
| wake() waking method     | Thread is unblocked when semaphore is incremented to 1 via V()         | Thread is removed from waitQueue and marked ready |
| Atomicity Approach       | Relies on semaphore's atomic operations                                | Uses direct interrupt disable/restore             |
| Data Structure           | Queue of semaphores                                                    | Queue of thread objects                           |
| Lock Handling            | Lock released before waiting, reacquired after. DONE within semaphores | Lock released before waiting, reacquired after    |
| Thread State Control     | Indirect (via semaphore)                                               | Direct (manipulating thread state)                |

#### Recap Of Condition Variables
The condition variables are used hand in hand with locks to ensure that threads:
1. Wait for specific condition to become true
	1. In the semaphore approach: Each waiting thread has its own semaphore with a counter that blocks the thread when 0 and wakes it when incremented
	2. In the interrupt-based approach: Threads are added to and removed from queues directly, with interrupt disable/restore ensuring atomicity of these operations
2. Signal to other threads that condition becomes true

***Condition2 Functions***
With condition2, 
- queue is of Threads instead of Semaphores. 
- For every function we disable and enable interrupts to ensure everything is atomic
	- With the original condition class, conditions were implemented using Semaphores, so in the condition functions the Semapores handled the functionality of enabling/disabling the interrupts. `P() and V()` function calls respectively inside the Semaphore class. [Link to functions P() / V()](https://github.com/ucsd-cse120-sp25/nachos_sp25_a2jensen_cjcanaday_sebastian-dv/blob/proj1/nachos/threads/Semaphore.java)
	- With the condition2 class, we handle interrupts within the functions themselves(sleep(), wake(), wakeAll()). We handle putting the threads to sleep and marking them as ready.
- 

***Test 1 functionality***
We test functionality of `wake` and `sleep`, alternating between 2 threads
- We create 2 threads `ping and pong`
- We call `fork`, which in nachos just tells the threads it’s ready to run by adding it to ready queue
	- runs the `run` in Interlocker class which implements `Runnable`
		- print out the name
		- then we call `wake`, which signals other thread to wake up. Look at the queue, grab the first thread and mark it as ready!
		- then we call `sleep`, putting the thread that just rant to sleep. this releases the lock, adds thread to queue, and puts it to sleep
		Running this through the test. Ping is forked first, so it acquires the lock first and runs the code above, pong is put to sleep in the meantime and is added to the queue. After first iteration, ping gets put to sleep and added to the queue and pong gets woken up acquiring the lock and getting removed from queue. They keep alternating over and over until 10 iterations
- Afterwards, we call yield on the main thread(represented by the selfTest)
	- Now for 50 iterations, we print out the name / message then we yield, causing a context switch to another thread by the CPU in the waiting queue.
		- In this case, i think no other thread is ever in the queue so the main thread yields after every iteration and then automatically gets the CPU bacl

***Test 2 functionality***
This test demonstrates the producer-consumer pattern using condition variables:
- We create a consumer thread and a producer thread, both sharing a lock and condition variable
- The consumer runs first:
    - Acquires the lock
    - Checks if the list is empty; since it is, it calls `sleep()` which:
        - Releases the lock
        - Adds the consumer thread to the wait queue
        - Puts the consumer thread to sleep
- The producer then runs:
    - Acquires the lock (which was released by the consumer)
    - Adds 5 integers (0-4) to the list, printing "Added X" after each addition
    - After adding all 5 items, calls `wake()` which:
        - Removes the consumer from the wait queue
        - Marks it as ready to run
    - Releases the lock
- The consumer wakes up and reacquires the lock
    - Verifies that the list has 5 values
    - Removes and prints each value from the list
    - Releases the lock
- After both threads complete, we either:
    - Wait for them using join() (if implemented)
    - Or use the alternative yielding approach (explaining all those "thread yielding" messages)

***Test 3 functionality : sleepFor()***
1. The main thread creates the lock and condition variable
2. The main thread creates the child thread (but it's not running yet)
3. The main thread calls `child.fork()`, which adds the child thread to the ready queue
4. The main thread acquires the lock
5. The main thread records the start time and prints that it's sleeping
6. The main thread calls `sleepFor(4000)`, which:
    - Adds the main thread to the wait queue
    - Releases the lock
    - Goes to sleep

At this point, the lock is released, and the scheduler can run the child thread:

7. The child thread calls `waitUntil(1000)` to wait 1000 ticks
8. The child thread prints that it's acquiring the lock
9. The child thread acquires the lock (which was released when the parent went to sleep)
10. The child thread prints that it's waking the parent
11. The child thread calls `cv.wake()`, which wakes up the parent thread
12. The child thread releases the lock and prints that it's releasing the lock

Then the parent thread wakes up:

13. The parent thread reacquires the lock (since the child released it)
14. The parent thread records the end time and prints how long it slept
15. The parent thread releases the lock

This test demonstrates the key functionality of condition variables - they allow threads to coordinate with each other through signaling. The parent thread waits on a condition (using sleepFor), and the child thread signals when that condition is met (using wake).

The test also confirms that your sleepFor implementation handles being woken up before the timeout expires correctly!


#### Personal Quick Summary Of Classes After Project Was Finished

- KThread is used to handle the init of thread itself
- `condition1` class uses semaphores as condition variables with the KThreads in order to provide the functionality of threads waiting on a "condition" in order to run, semaphores act as an abstraction and provide functionality of atomicity and to ensure the implementation of critical sections(only 1 thread can be inside a piece of code when getting/releasing a lock and handling other functionalities such as being put as a waiter)
- `condition2` provides the same exact functionality i listed above(atomicity, providing atomicity)except we use interrupts directly and theres not additional abstraction of semaphores. we use a queue directly that holds the waiting threads that are waiting on the condition variable of being waked up(setting them to ready)
	- the main use of condition variables is to provide the functionality of communication between threads to ensure that the lock is being properly released/acquired and to ensure that threads are properly getting scheduled based on the fact that there waiting on the condition variable
- for the `alarm` class, this is used by the threads in order to provide the extra functionality of them waiting for a certain amount of ticks. it essentially is the OS timer interrupt, essentially implementing how the OS has control over the threads by providing timer/ticks, avoiding the problem of threads running infinitely. here we use a queue as well for threads specifically with timers and are currently sleeping, except these threads are waiting/sleeping based on the timer assigned to them.

##### Difference Between KThread `sleep()` and Condition `sleep()`
- `sleep()` in KThread is a function that puts the thread directly to sleep.
	-  `sleep()` for both versions(semaphore and direct interrupt manipulation) of condition is like an abstraction over KThread `sleep()`, where it puts the thread to sleep based on the condition variable!
		- For semaphore: we can assume condition1 `sleep()` would be called for a thread and here the thread will be put to sleep when the semaphore is at value 0(done by `P()` function in semaphore class, it stay's asleep until the condition variable(semaphores in this case) is set to 1, done by `V()` function in semaphore class. While asleep remember the lock is released!
		- For direct interrupts: with condition2 `sleep()`, the thread would be put to sleep directly(like using KThread `sleep`), it then gets added to a waiterQueue paired with interrupts(interrupts to ensure atomicity). It then gets woken up when another thread calls `wake` on it. While asleep remember the lock is released!
			- With condition2, less abstraction