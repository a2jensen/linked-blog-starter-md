OS is essentially the bridge between hardware and software. It acts as the controller for access to the hardware resources and provides abstractions to applications. Application's don't need to worry about managing their resources with the hardware, operating system does it for them.

Basic Idea from the book:
OS takes the hardware and makes virtual versions of themselves. It handles tricky problems related to concurrency. It deals with storing files persistently and ensuring their safe long term.

Remember concurrency is essentially tasks running at the same time, which may or may not run parallel.

![[Screenshot 2025-04-01 at 4.37.46 PM.png|200]]

![[Screenshot 2025-04-04 at 2.55.06 PM.png|300]]

Hardware resources include:
- Computation(CPUs)
- Volatile storage (memory) and persistent storage (disk, etc.) 
- Communication (network, etc.) ▪ Devices (keyboard, camera, monitor, etc.)

OS operations include:
- Resource allocation + reclamation
- Protection - between and from applications

### What Is Part Of An OS?
- Windowing system? depends
- Web browser? No
- Network stack? Yes
- Memory allocation? Split between application layer and OS
	- For example C standard library that implements`malloc or free` is on the application layer. 
	- OS handles the physical and virtual memory, using system calls like `brk`, `sbrk`, and `mmap`.

### OS Design and Properties
Goals in designing can conflict...
Fairness vs. efficiency
Security vs performance

OSes face constraints and tradeoffs 
	▪ Different objectives for different applications 
• OSes must adapt over time due to evolution of: 
	▪ Hardware ▪ Applications 
• Design principles can guide us 
	▪ Abstraction ▪ Modularity ▪ Simplicity ▪ Caching

### Seperation of Policy and Mechanism
Fundamental design principle in computer science 
• Mechanism: tool that achieves some effect 
• Policy: decision about what effect should be achieved 
• CPU scheduling example: 
	▪ Treat all users equally ▪ Treat all applications equally ▪ Prioritize some applications over others 
• Separation leads to flexibility!
### What Course Actually Covers
Principles of OS design - theory based.


### Reading Notes
- OS is the resource manager
- Can be referred to as virtual machine, takes the physical hardware and transforms it into a virtual form of itself
- CPU, disk, memory is a resource
- Gives the illusion that the 1 CPU is infinite CPUs
- Concurrency is a big part, focuses on threads. Think of threads as functions running within the same memory space.
- Persistence is a another major focus, deals with storing data persistently on things like HDD or SSD
- Software in the OS that deals with the bullet point above is called file system, responsible for storing files onto the disks of the system