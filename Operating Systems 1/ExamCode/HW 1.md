***Q1 : How do hardware and the OS work together to handle interrupts? When an interrupt happens, what tasks are handled by the hardware and what tasks are handled by the OS?***
Context : [[Classes/Operating Systems/Interactions With Hardware and Apps]]
Interrupts are a type of event which is trigged by hardware (examples include I/O hardware interrupts and timer). When an interrupt happens, there is an interrupt handler in the kernel inside the OS which will do the following :
	1. Disables interrupts at lower priorities
	2. Save state
	3. transfer control to the interrupt service routine

***Q2 For each of the three mechanisms that supports dual-mode operation — privileged instructions, memory protection, and timer interrupts — explain what might go wrong without that mechanism, assuming the system only had the other two.***
1. Without privileged instructions
	Without privileged instructions, a negative that can occur is that applications can end up having the ability to run these "privileged instructions" themselves. Applications would have direct access to I/O devices such as the disk or network. They would have the ability to also manage and manipulate other application memory.
2. Without memory protection
	Without memory protection, the Operating system is vulnerable to potentially malicious user programs or hardware which can somehow cause damage to the OS. In addition, applications would have access to the memory of other applications in the event of no memory protection.
	In addition, **user programs could read or write to any part of memory**, including the memory used by the OS or other user programs. This would lead to severe **violations of isolation and security**, potentially allowing one process to crash another or to escalate privileges by modifying OS code/data.
3. Without timer interrupts
	Without timer interrupts, the Operating System loses the ability to prevent programs from "hogging up the CPU". Examples include an infinite while loop or recursion which results in crazy amounts of memory being used.  Without the timer, OS loses the ability to reclaim control.

***Q3: Suppose you have to implement an operating system on hardware that supports interrupts and exceptions but does not have an explicit syscall instruction. Can you devise a satisfactory substitute for syscalls using interrupts and/or exceptions? If so, explain how. If not, explain why. (In this context, the syscall instruction is the instruction used by a user-level process to invoke a system call in the operating system.)***
Yes, it is possible and this can be done by placing the responsibility on the software for acting as the "trigger" for interrupts or exceptions. Now, we would also need the hardware to allow the applications to trigger these interrupts/exceptions and this can be done via a special instruction. The special instruction would be responsible for a change in the control flow, switching to kernel mode and performing the handler.
###### Example strategy using interrupts:
- Use a designated **software interrupt number** (e.g., `INT 0x80` in Linux on x86).
- When a user process needs to make a syscall, it sets up syscall **arguments in registers or memory**, then executes:
	  `INT 0X80`
- The CPU:
    - Traps to **kernel mode**
    - Jumps to the **interrupt handler**
- The OS:
    - Reads the syscall number and arguments
    - Performs the requested service (e.g., `read`, `write`)
    - Returns the result to the user program and resumes it

***Q4 : [Silberschatz] Which of the following instructions should be privileged? Give a one-sentence explanation for why.***
    a) Set value of timer  
	    Privileged, setting the value of the timer will control the CPU scheduling and resources usage. OS needs to handle this as we don't want apps adjusting.
    b) Read the clock 
			Non-Privileged, providing system clock information should be important to apps. They can only read the clock, not write any changes which prevents misuse.
    c) Clear memory  
		    Privileged, clearing memory can affect other processes going on in the OS. It can cause the problem of data corruption / loss.
    d) Turn off interrupts  
		    Privileged, giving access to this outside of the OS can cause future issues such as OS interrupt handler not running in the case of interrupts happening.
    e) Switch from user to kernel mode
			 Privileged, the OS should be responsible for letting in who has access to the sensitive resources, not the apps / hardware themselves.
***Q5***
a:
```
angelojensen@Angelos-MacBook-Pro-2 Desktop % ./a.out
1 sec
2 sec
3 sec
4 sec
5 sec
```

b:
```
angelojensen@Angelos-MacBook-Pro-2 Desktop % time ./a.out
1 sec
2 sec
3 sec
4 sec
5 sec
./a.out  5.00s user 0.01s system 99% cpu 5.020 total
```

***Q6***
Yes, it is possible for the program to print the `!=` message in the handles due to a number of reasons.
Specifically, it is not atomic in that the code doesn't "happen all at once" for assigning values to the `value` variable. Specifically this scenario could happen in which the Signal can be called to interrupt the program mid computation of updating the `value`.

```c
while (1) {
      value.one =  ones;
      Signal called here, results in != due to value.two not being set yet
      value.two = ones
      value = zeros; // value is {0,}
    }
```


***Q7
For each of the following Unix system calls, give a condition that causes it to fail: open, read, fork, exec, unlink (delete a file). (Hint: We discussed some in lecture and you can also explore the error semantics of these system calls using man on ieng6, e.g., man 2 fork.)***
1. open()
```
open("nofile.txt", read_only);
```
Can fail due to file not existing(file path is null) or not having sufficient permissions to read/write.
2. read()
```
read("", buffer, 100);
```
Can fail due to file not existing(file path is null) or not having sufficient permissions to read/write.
3. fork()
```
while (1) fork(); // will fail at some point
```

Limit for number of processes will be reached, resulting in fail at some point during the while loop
4. ecec()
	Will fail if file is not found or if it is not in the proper format for execution.
5. unlink()
	 Will fail if file does not exist or if you do not have proper permissions.

***Q8***
a. 6 processes
b. 2 