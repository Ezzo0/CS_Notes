# Explanation
- Address space is the running program’s view of memory in the system.
- The address space of a process contains all of the memory state of the running program.
- when we describe the **address space**, what we are describing is the **abstraction** that the OS is providing to the running program.
- The program really isn’t in memory at physical addresses 0;rather it is loaded at some arbitrary physical address(es).
- When the OS does this, we say the OS is **virtualizing memory**, because the running program thinks it is loaded into memory at a particular address (say 0) and has a potentially very large address space.![[Address Space.png|400]]
- When, for example, process A in Figure 13.2 tries to perform a load at address 0 (which we will call a virtual address), somehow the OS, in tandem with some hardware support, will have to make sure the load doesn’t actually go to physical address 0 but rather to physical address 320KB (where A is loaded into memory).
##### Goals
- One major goal of a virtual memory (VM) system is **transparency**.
- The OS should implement virtual memory in a way that is invisible to the running program.
- Another goal of VM is efficiency. The OS should strive to make the virtualization as efficient as possible, both in terms of time and space.
- In implementing time-efficient virtualization, the OS will have to rely on hardware support, including hardware features such as TLBs.
- Finally, a third VM goal is protection. The OS should make sure to protect [[Process]]-es from one another as well as the OS itself from processes.
- Protection thus enables us to deliver the property of isolation among processes.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 13.
- [Lecture 3 - part 2](https://youtu.be/I0RIlSN0DzM)