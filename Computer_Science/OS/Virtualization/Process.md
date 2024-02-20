# Explanation
- The process is a running program.
- Os run one process, then stopping it and running another, and so forth, the OS can promote the illusion that many virtual CPUs exist when in fact there is only one physical CPU (or a few).
- This basic technique, known as **time sharing of the CPU**, allows users to run as many concurrent processes as they would like.
	- The counterpart of time sharing is space sharing, where a resource is divided (in space) among those who wish to use it.
- The potential cost of time sharing technique is performance, as each will run more slowly if the CPU(s) must be shared.
- To implement virtualization of the CPU, and to implement it well, the OS will need both some low-level machinery (**mechanisms**) and some high-level intelligence (**policies**).
- Mechanisms are low-level methods or protocols that implement a needed piece of functionality.
- Policies are algorithms for making some kind of decision within the OS. For example, given a number of possible programs to run on a CPU, which program should the OS run?
##### Process API
- These APIs, in some form, are available on any modern operating system:
	1. **Create**: An operating system must include some method to create new processes.
	2. **Destroy**: An operating system must include some method to halt a runaway process.
	3. **Wait**: OS wait for a process to stop running.
	4. **Miscellaneous Control**: Most operating systems provide some kind of method to suspend a process (stop it from running for a while) and then resume it (continue it running).
	5. **Status**: There are usually interfaces to get some status information about a process.
##### Process Creation
- The first thing that the OS must do to run a program is to load its code and any static data (e.g., initialized variables) into memory, into the address space of the process.
	 ![[Loading a program.png|500]]
- In early (or simple) operating systems, the loading process is done eagerly, i.e., **all at once before running the program**. 
- Modern OSes **_perform the process lazily_**, i.e., **by loading pieces of code or data only as they are needed during program execution**.
- Once the code and static data are loaded into memory, there are a few other things the OS needs to do before running the process. 
	- Some memory must be allocated for the program’s run-time stack (or just stack).
	- OS allocates this memory and gives it to the process. The OS will also likely initialize the stack with arguments; specifically, it will fill in the parameters to the main() function, i.e., argc and the argv array.
	- The OS may also allocate some memory for the program’s heap.
	- The OS will also do some other initialization tasks related to input/output (I/O).
	- Jumping to the `main()` routine. Now, the OS transfers control of the CPU to the newly-created process, and thus the program begins its execution.
##### Process States
- A process can be in one of three states:
	1. **Running**: A process is running on a processor. This means it is executing instructions.
	2. **Ready**: A process is ready to run but for some reason the OS has chosen not to run it at this given moment.
	3. **Blocked**: a process has performed some kind of operation that makes it not ready to run until some other event takes place.
		- A common example: when a process initiates an I/O request to a disk, it becomes blocked and thus some other process can use the processor.
	![[State Transitions.png|500]]
	- As you can see in the diagram, a process can be moved between the ready and running states at the discretion of the OS.
		- Being moved from ready to running means the process has been **_scheduled**_.
		- Being moved from running to ready means the process has been **_descheduled_**
		- Once a process has become **blocked** (e.g., by initiating an **I/O operation**), the OS will keep it as such until some event occurs (e.g., **I/O completion**); at that point, the process moves to the ready state again.
	- Sometimes a system will have an **initial state** that the process is in when it is being created.
	- Also, a process could be placed in a **final state** where it has exited but has not yet been cleaned up (in UNIX-based systems, this is called the **_zombie state_**).
		- This final state can be useful as it allows other processes (usually the parent that created the process) to examine the return code of the process and see if the just-finished process executed successfully or not.
##### Data Structures
- To track the state of each process, for example, the OS likely will keep some kind of process list (also called the task list) for all processes that are ready and some additional information to track which process is currently running. 
- The OS must also track, in some way, blocked processes; when an I/O event completes, the OS should make sure to wake the correct process and ready it to run again.
- Sometimes people refer to the individual structure that stores information about a process as a **Process Control Block (PCB)** (also sometimes called a **process descriptor**).
# Sources
- Operating Systems: Three Easy Pieces - Chapter 4.
- [Lecture 2 - part 1](https://youtu.be/oTd72Yp2m8w)
- [Lecture 2 - part 2](https://youtu.be/Q09UgVfragU)