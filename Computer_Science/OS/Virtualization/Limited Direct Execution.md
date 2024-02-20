# Explanation
- In order to virtualize the CPU, the operating system needs to somehow share the physical CPU among many jobs running seemingly at the same time. 
- The basic idea is simple: run one [[Process]] for a little while, then run another one, and so forth. By **time sharing** the CPU in this manner, virtualization is achieved.
- There are a few challenges, however, in building such virtualization machinery.
	1. performance: how can we implement virtualization without adding excessive overhead to the system?
	2. control: how can we run processes efficiently while retaining control over the CPU?
		- Without control, a process could simply run forever and take over the machine, or access information that it should not be allowed to access.
##### Basic Technique: Limited Direct Execution
- The “direct execution” part of the idea is simple: just run the program directly on the CPU.
- Figure 6.1 shows this basic direct execution protocol (without any limits, yet), using a normal call and return to jump to the program’s main() and later back into the kernel.
    ![[Direct Execution Protocol (Without Limits).png|500]]
- This approach gives rise to a few problems in our quest to virtualize the CPU. 
	- The first: if we just run a program, how can the OS make sure the program doesn’t do anything that we don’t want it to do, while still running it efficiently? 
	- The second: when we are running a process, how does the operating system stop it from running and switch to another process, thus implementing the time sharing we require to virtualize the CPU?
##### Problem #1: Restricted Operations
- The approach we take is to introduce a new processor mode, known as **user mode**.
- Code that runs in user mode is **restricted in what it can do**.
- In contrast to user mode is **kernel mode**, which the operating system (or kernel) runs in.
- In this mode, code that runs can do what it likes, including privileged operations such as issuing I/O requests and executing all types of restricted instructions.
- what should ? To enable this, virtually 
- All modern hardware provides the ability for user programs to perform a **system call** to allow a user process to perform some kind of privileged operation, such as reading from disk .
- To execute a system call, a program must execute a special **trap instruction**. 
	- This instruction simultaneously jumps into the kernel and raises the privilege level to kernel mode. 
	- Once in the kernel, the system can now perform whatever privileged operations are needed (if allowed), and thus do the required work for the calling process. 
- When finished, the OS calls a special **return-from-trap instruction**, which returns into the calling user program while simultaneously reducing the privilege level back to user mode.
- **_How does the trap know which code to run inside the OS?_**
	- The kernel does so by **setting up a trap table at boot time**.
	- When the machine boots up, it does so in privileged (kernel) mode, and thus is free to configure machine hardware as need be.
	- By setting up trap table, the OS tell the hardware what code to run when certain exceptional events occur.
	- To specify the exact system call, **a system-call number** is usually assigned to each system call. 
	- The OS, when handling the system call inside the **trap handler**, examines this number, ensures it is valid, and, if it is, executes the corresponding code. 
	- This level of indirection serves as a form of **protection**; user code cannot specify an exact address to jump to, but rather must request a particular service via number.
##### Problem #2: Switching Between Processes
- **_A Cooperative Approach: Wait For System Calls_**
	- One approach that some systems have taken in the past is known as the **cooperative approach**.
	- Processes that run for too long are assumed to periodically give up the CPU so that the OS can decide to run some other task.
	- Applications also transfer control to the OS when they do something illegal. 
		- For example, if an application divides by zero, or tries to access memory that it shouldn’t be able to access, it will generate a trap to the OS. 
		- The OS will then have control of the CPU again (and likely terminate the offending process).
- **_A Non-Cooperative Approach: The OS Takes Control_**
	- Without some additional help from the hardware, it turns out the OS can’t do much at all when a process refuses to make system calls.
	- A timer device can be programmed to raise an interrupt every so many milliseconds.
	- When the interrupt is raised, the currently running process is halted, and a pre-configured **interrupt handler** in the OS runs.
- **_Saving and Restoring Context_**
	- Note that there are two types of register **saves**/**restores** that happen during this protocol. 
	- The first is when the timer interrupt occurs; in this case, the user registers of the running process are implicitly saved by **the hardware, using the kernel stack of that process**. 
	- The second is when the OS decides to switch from A to B; in this case, the kernel registers are explicitly saved by **the software** (i.e., the OS), but this time into **memory in the process structure of the process**. 
	- The latter action moves the system from running as if it just trapped into the kernel from A to as if it just trapped into the kernel from B.
	   ![[Limited Direct Execution Protocol (Timer Interrupt).png|500]]
# Sources
- Operating Systems: Three Easy Pieces - Chapter 6.
- [Lecture 1 - part 3](https://youtu.be/LVxN7ZkGh3w)