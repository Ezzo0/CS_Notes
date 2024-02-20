# Explanation
- OS is a body of software that is responsible for making it easy to run programs, allowing programs to share memory, enabling programs to interact with devices, etc..
- OS is in charge of making sure the system operates correctly and efficiently in an easy-to-use manner.
- OS is known as a resource manager.
- The primary way that the OS does the above points is through a general technique called **Virtualization**.
- Virtualization is, the OS takes a physical resource (such as the processor, or memory, or a disk) and transforms it into a more general, powerful, and easy-to-use virtual form of itself. Thus, OS sometimes is referred as a virtual machine.
- A typical OS exports a few hundred **system calls** that are available to applications. 
- Because the OS provides these calls to **_run programs, access memory and devices, and other related actions_**, we also sometimes say that the OS provides a standard library to applications.
- Turning a single CPU (or a small set of them) into a seemingly infinite number of CPUs and thus allowing many programs to seemingly run at once is called **virtualizing the CPU**.
- Each [[Process]] accesses its own **_private virtual address space_**, which the OS somehow maps onto the physical memory of the machine is called **Virtualizing Memory**.
- The software in the operating system that usually manages the disk is called the file system; it is responsible for storing any files the user creates in a reliable and efficient manner on the disks of the system.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 2
- [Lecture 1 - part 1](https://youtu.be/3uMbb9dLtlE)