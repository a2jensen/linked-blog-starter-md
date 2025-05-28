- Separation of Policy and Mechanism - Intro
- CPU Scheduling
	- Policy vs Mechanism
	- Scheduler
		- CPU Scheduling Policies - brief intro/summary/in practice
		- Scheduling Goals
		- Scheduling Challenges
		- Preemptive vs Non-Preemptive Scheduling
	- Scheduling Policies
		- [[#First-come first-served or first-in first-out(FCFS / FIFO)]]
		- [[#Shortest Job First (SJF)]]
		- [[#Shortest Remaining Time to Completion First (SRTCF)]]
		- [[#Round Robin]]
			- [[#FCFS vs. Round Robin - ex 1]]
			- [[#FCFC vs Round Robin - ex 2]]
		- [[#Priority Scheduling]]
		- [[#Multi-level Feedback Queues(MLFQ)]]
	- Handling I/0
	- Scheduling Overhead
	- CPU Utilization
#### Seperation of Policy and Mechanism
- Mechanism: tool that achieves some effect
- Policy: decision about what effect should be achieved 
- Example: card keys instead of physical keys
- Separation leads to flexibility!
#### CPU Scheduling
- Share CPU Resources by time-slicing the CPU
- Processing ilusion: every process think it owns the CPU


### Scheduling Policies
#### First-come first-served or first-in first-out(FCFS / FIFO)
You can argue for it being fair or unfair.
![[Screenshot 2025-04-24 at 4.14.27 PM.png|450]]
#### Shortest Job First (SJF)
- Run the job with the shortest run time first
- non-preemptive
***Example***
![[Screenshot 2025-04-24 at 4.16.20 PM.png|450]]
In the example above, since it is preemptive, job 1(length 7) keeps running despite other jobs coming in with shorter length. The jobs roll in over time, not all at once. If they did come in all at once, then we can assume we choose the shortest one and run that first.

![[Screenshot 2025-04-24 at 4.18.09 PM.png|450]]

#### Shortest Remaining Time to Completion First (SRTCF)
![[Screenshot 2025-04-24 at 4.18.40 PM.png|450]]
#### Round Robin
![[Screenshot 2025-04-24 at 4.21.47 PM.png|450]]
Expanding on the con: context switch that requires a lot more resources will add more overhead.

##### FCFS vs. Round Robin - ex 1
We assume they all arrive at the start, not over time.
![[Screenshot 2025-04-24 at 4.30.02 PM.png|450]]
##### FCFC vs Round Robin - ex 2
![[Screenshot 2025-04-24 at 4.35.00 PM.png|450]]

#### Priority Scheduling
![[Screenshot 2025-04-24 at 4.36.12 PM.png|450]]

#### Multi-level Feedback Queues(MLFQ)
![[Screenshot 2025-04-24 at 4.37.06 PM.png|450]]


| Scheduling Algorithm                 | Preemptive? | Starvation Possible? | Why / Why Not                                                                  |
| ------------------------------------ | ----------- | -------------------- | ------------------------------------------------------------------------------ |
| FIFO (First In First Out)            | ❌ No        | ❌ No                 | Processes are handled in arrival order. No one is skipped or delayed unfairly. |
| Shortest Job First (SJF)             | ❌ No        | ✅ Yes                | Long jobs can be postponed forever if short jobs keep arriving.                |
| Shortest Remaining Time First (SRTF) | ✅ Yes       | ✅ Yes                | Long-running jobs can be continuously preempted by new shorter jobs.           |
| Priority Scheduling                  | ✅/❌ Both    | ✅ Yes                | In preemptive form, low-priority processes can be indefinitely delayed.        |
| Round Robin                          | ✅ Yes       | ❌ No                 | Each process gets a fixed time slice; fair rotation ensures progress.          |
|                                      |             |                      |                                                                                |

| Primitive           | Purpose                             | Blocking Behavior     | Allows Multiple Threads? | Can Signal Specific Thread? | Common Use Case                                      |
|---------------------|--------------------------------------|------------------------|---------------------------|-----------------------------|------------------------------------------------------|
| **Lock (Mutex)**     | Mutual exclusion (1 thread at a time) | Yes (if already held)  | ❌ No                     | ❌ No                       | Critical section protection                         |
| **Semaphore**        | Resource counting / signaling         | Yes (if count is 0)    | ✅ Yes (if count > 1)     | ❌ No                       | Limiting access to finite resources (e.g., 3 printers) |
| **Condition Variable** | Wait for a condition to be true     | Yes (explicitly waits) | ✅ Yes                    | ✅ Yes (can `signal` one)   | Blocking threads until a condition is met (e.g., buffer not empty) |
