### Q1
***a : TRACETHROUGH***
Related : [[WI 25 Project Notes]], [[Classes/Operating Systems/Threads]]
Trace through:
`selfTest()` is the main thread, `t1` is the child thread initialized within selfTest()
- New thread is created `t1` , then we print ***`fee`***
- main thread fork it, which in nachos puts `t2` in the ready queue, then we print ***`foe`***
	- **NOTE:** `fork()` _does not_ immediately run it unless a context switch happens.
- main thread calls `t1.join()`, in which main thread is blocked until `t1` finishes
	- `run()` in `A` now is running `t1 thread`
		- We create a new thread `t2`, and print out ***`foo`***
		- We fork it, which in nachos puts it in the ready queue, then we print ***`far`***
		- We call `t2.join()`, in which  the parent thread,`t1` , is blocked until `t2` finishes
			- We are now running `run()` in `class B`
				- We print out ***`fie`*** , and we finish running thread `t2`
		- `t1` now is freed up, and finishes out `run()` in `class A` printing ***`fum`***
- `main thread selfTest()` is now freed up, and prints ***`fun`***

***b : OUTPUT***
```
fee
foe
foo
far
fie
fum
fun
```

***c : queue states***

|                           |        |             |
| ------------------------- | ------ | ----------- |
| Start                     | [ ]    | main thread |
| Create `t1`               | [ ]    | main thread |
| After `fee`               | [ ]    | main thread |
| After `t1.fork()`         | [ t1 ] | main thread |
| After `foe`               | [ t1 ] | main thread |
| After `t1.join()`         | [ ]    | t1          |
| Create `t2`               | [ ]    | t1          |
| After `t2.fork()`         | [ t2 ] | t1          |
| After `far`               | [ t2 ] | t1          |
| After `t2.join()`         | [ ]    | t2          |
| After `t2` finishes       | [ ]    | t1          |
| After `fum` (t1 finishes) | [ ]    | main thread |
| After `fun`               | [ ]    | main thread |

### Q2
***psuedocode**
Related : [[Classes/Operating Systems/Synchronization#Locks]]

```c
struct lock {
    bool held;
};

// IMPLEMENTING XCHG HERE IN ACQUIRE AND RELEASE
void acquire(struct lock *lck) {
    bool old = true;  // initialize to true to enter loop
    while (old) {
        bool temp = true;
        XCHG(&lck->held, &temp);
        old = temp;
        // if old == false, then that indicates we got the lock!
    }
}

void release(struct lock *lck) {
    lck->held = false;
    // we do not need release here i assume, since release should be uncontested
}

```
### Q3
***psuedocode***
NOTE: refer to [[Classes/Operating Systems/Semaphores#Test-and-set]]
***a***
```java
monitor Barrier {
    count = 0
    condition allHere

    procedure Done(n):
        count += 1
        if count < n:
            wait(allHere)
        else:
            reset count to 0
            notify all waiting threads on allHere
}
```

***b***
```java
class Barrier {
    lock
    condition allHere
    count = 0

    procedure Done(n):
        acquire lock
        increment count
        if count < n:
            while count < n:
                wait on allHere
        else:
            reset count to 0
            wake up all waiting threads on allHere
        release lock
}

```

***c***
```java
class Barrier {
    lock
    semaphore sem initialized to 0
    count init 0

    procedure Done(n):
        acquire lock
        increment count
        if count == n:
            reset count to 0
            signal semaphore (n-1) times
            release lock
        else:
            release lock
            wait on semaphore
}
```
### Q4
Related : [[Classes/Operating Systems/Threads]], [[Classes/Operating Systems/Synchronization]]
***psuedocode***
- Key points: code within `paddle, wave, done` functions are "mutual exclusion zones"
```java
class Surfing {
    lock
    condition left_Wait
    condition right_Wait
    State state = CALM
    Direction waveDirection

    constructor Surfing():
        initialize lock
        initialize left_Wait
        initialize right_Wait
        state = CALM

    procedure paddle(Direction dir):
        acquire lock
        while state == CALM or (state == BREAKING and waveDirection != dir and waveDirection != BOTH):
            if dir == LEFT:
                wait on left_Wait
            else if dir == RIGHT:
                wait on right_Wait
        release lock

    procedure wave(Direction dir):
        acquire lock
        state = BREAKING
        waveDirection = dir
        if dir == LEFT:
            wake all threads waiting on leftWait
        else if dir == RIGHT:
            wake all threads waiting on rightWait
        else if dir == BOTH:
            wake all threads waiting on leftWait
            wake all threads waiting on rightWait
        release lock

    procedure done():
        acquire lock
        state = CALM
        release lock
}

```
### Q5
Related : [[Classes/Operating Systems/Condition Variables and Deadlock]]
![[Screenshot 2025-04-25 at 5.54.00 PM.png|450]]

1. Childi has the resources to do his task, so he frees up both a thesaurus and coffee cup
2. Jason is able to get the freed up coffee cup and do his task, therefore freeing up both coffee cups when he is done with his task
3. Tahani at the same time also gets access to the thesaurus, allowing her to run her task. Freeing up the thesaurus and dictionary after
4. Lastly, Eleanor now can access the dictionary, allowing her to run her task

Therefore, the system is NOT deadlocked in this state.