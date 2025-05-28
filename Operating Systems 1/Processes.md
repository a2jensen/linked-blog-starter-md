- [[#Process]] - OS Abstraction for a running program. "a program in running execution"
	- [[#Process Components]]
	- [[#Process Vs Program]]
	- [[#Basic Process Address Space]]
	- [[#Process State]]
		- [[#Processing Illusion]]
	- [[#Process Data Structure]]
	- [[#Process Creation]]
		- [[#Tree Example Of Processes]]
		- [[#Process Creation API]]
			- [[#Process Creation With CreateProcess(Windows)]]
			- [[#Process Creation With fork(Unix)]]
			- [[#Starting a New Program with exec (Unix)]](does not create new process but is still apart of API so i added it here)
	- [[#Process Termination]]
	- [[#wait() function - pausing until child process finishes]]
- [[#Lecture Q's]]
-  [[#General Q's About Process]]

***CONTEXT: What is the shell?*** 
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
### TLDR
- Processes : OS Abstraction for a running program
- Process Management :
	- API's used to interact with the processes
- Process vs Program: process is instance of program in
		execution


## Process
![[Screenshot 2025-04-08 at 3.41.24 PM.png|400]]
### Process Components
A process contains all state for a program in execution 
▪ A memory address space 
▪ Code and data for the executing program 
▪ An execution stack encapsulating the state of procedure calls 
▪ The program counter (PC) indicating the next instruction 
▪ A set of registers with current values 
▪ A set of operating system resources 
	» Open files, network connections, etc. 
A process is named using its process ID (PID)
![[Screenshot 2025-04-08 at 3.44.37 PM.png|400]]
- in picture multiple instances of google chrome running
	- All same program, but different processes

### Process Vs Program
![[Screenshot 2025-04-08 at 3.51.09 PM.png|400]]

### Basic Process Address Space
![[Screenshot 2025-04-08 at 3.51.27 PM.png|400]]
- Stack and Heap both placed at opposite ends so they grow organically, and they don't collide
	- Usually so big that they never reach, unless you have something like infinite recursion

### Process State
![[Screenshot 2025-04-08 at 3.58.50 PM.png|400]]
#### Processing Illusion
![[Screenshot 2025-04-08 at 3.59.23 PM.png|450]]

### Process Data Structure
![[Screenshot 2025-04-08 at 4.00.11 PM.png|400]]

### General Q's About Process
- How many processes can be in running state simultaneously? ***4***
- What state is a process in most of the time? ***Sleeping / Ready***
- What happens if you run exec bash in your shell? 
	- ***Answer :*** exec command replaces current shell with a new `bash` process. `bash` takes in the new place, original shell does not run. It does not create a new process, unlike just running `bash` alone. 
	- Think of it as swapping your current shell for a new bash one
-  What happens if you run exec ls in your shell?
	- ***Answer*** it will replace your current shell with the `ls` command. You won't go back to your shell - `ls` finishes, and the process ends entirely

Questions I asked myself? what is a shell? refer to here [[#]]



### Process Creation
![[Screenshot 2025-04-08 at 4.05.56 PM.png]]
- So the OS starts off by making the initial process
ex. Typing a command into the shell creates a "process"

#### Tree Example Of Processes
![[Screenshot 2025-04-08 at 4.08.18 PM.png|400]]
- Systemd is the first process created
	- Every other process is a child of it

#### Process Creation API
- Make processes via a System call, (API call)
	- Either create from scratch or clone from an existing process

##### Process Creation With CreateProcess(Windows)
- Makes one from scratch
![[Screenshot 2025-04-08 at 4.11.33 PM.png|450]]

##### Process Creation With fork(Unix)
- Makes one through cloning existing process
![[Screenshot 2025-04-08 at 4.14.01 PM.png|450]]
***Visualization of fork***
![[Screenshot 2025-04-08 at 4.15.01 PM.png]]
- `fork()` returns 0 in child process since no new process was made within the child...
- `fork()` in parent returns child's PID since there was a call within parent to make a new process

***Fork code example***
![[Screenshot 2025-04-08 at 4.18.43 PM.png|450]]
Example outputs
```
returns second printf(`my childs pid ... `)
returns first printf(`i am child, my PID is....`)
```

```
returns first printf(`i am child, my PID is....`)
returns second printf(`my childs pid ... `)
```

- Output varies because it depends on the scheduler - OS decides which one gets run first on the CPU. varies run to run

![[Screenshot 2025-04-08 at 4.20.21 PM.png|400]]

***Why fork??***
- Useful when the child is cooperating with parent
- Relies on parent's data to accomplish task
```
while (1) { 
	int sock = accept(); 
	int child_pid = fork(); 
	if (child_pid == 0) { 
	Handle client request and exit 
	} else 
	{ 
		Continue 
	} 
	}
```


#### Starting a New Program with exec (Unix)
![[Screenshot 2025-04-09 at 5.26.53 PM.png|400]]

***Visualization***
![[Screenshot 2025-04-09 at 5.28.08 PM.png|400]]

### Process Termination
![[Screenshot 2025-04-09 at 5.30.16 PM.png|450]]

### wait() function - pausing until child process finishes
![[Screenshot 2025-04-09 at 5.31.22 PM.png|450]]
![[Screenshot 2025-04-09 at 5.31.35 PM.png|450]]


## Lecture Q's
![[Screenshot 2025-04-09 at 5.43.06 PM.png|450]]