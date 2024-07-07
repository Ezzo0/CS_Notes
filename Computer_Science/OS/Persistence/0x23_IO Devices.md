# Explanation
##### System Architecture
- The picture shows a single CPU attached to the main memory of the system via some kind of **memory bus** or interconnect. 
- Some devices are connected to the system via a general **I/O bus**, which in many modern systems would be **PCI** (or one of its many derivatives); graphics and some other higher-performance I/O devices might be found here. 
- Finally, even lower down are one or more of what we call a **peripheral bus**, such as **SCSI**, **SATA**, or **USB**. These connect slow devices to the system, including disks, mice, and keyboards.
- System designers have adopted this hierarchical approach, where components that demand high performance (such as the graphics card) are nearer the CPU. Lower performance components are further away.
	 ![[Prototypical System Architecture.png|450]]
- Modern systems increasingly use specialized chipsets and faster point-to-point interconnects to improve performance.
- Along the top, the CPU connects most closely to the memory system, but also has a high-performance connection to the graphics card.
- The CPU connects to an I/O chip via Intel’s proprietary **DMI (Direct Media Interface)**, and the rest of the devices connect to this chip via a number of different interconnects.
- On the right, one or more hard drives connect to the system via the **eSATA** interface; **ATA** (the **AT Attachment**, in reference to providing connection to the IBM PC AT), then **SATA** (for **Serial ATA**), and now **eSATA** (for **external SATA**) represent an evolution of storage interfaces over the past decades, with each step forward increasing performance to keep pace with modern storage devices.
- Below the I/O chip are a number of **USB (Universal Serial Bus)** connections, which in this depiction enable a keyboard and mouse to be attached to the computer.
- On the left, other higher performance devices can be connected to the system via **PCIe (Peripheral Component Interconnect Express)** (such as **NVMe** persistent storage devices).
	 ![[Modern System Architecture.png|450]]
##### A Canonical Device
- A device has two important components. The first is the hardware **interface** it presents to the rest of the system.
- The second part of any device is its **internal structure**. This part of the device is implementation specific and is responsible for implementing the abstraction the device presents to the system.
	 ![[A Canonical Device.png|450]]
##### The Canonical Protocol
- Device interface is comprised of three registers: 
	1. A **status** register, which can be read to see the current status of the device. 
	2. A **command** register, to tell the device to perform a certain task.
	3. A **data** register to pass data to the device, or get data from the device.
- The protocol has four steps. 
	1. In the first, the OS waits until the device is ready to receive a command by repeatedly reading the status register (**polling** the device).
	2. Second, the OS sends some data down to the data register.
		- When the main CPU is involved with the data movement (as in this example protocol), we refer to it as **programmed I/O (PIO)**. 
	3. Third, the OS writes a command to the command register.
	4. Finally, the OS waits for the device to finish by again polling it in a loop.
- The problem you might notice in the protocol is that **polling** seems inefficient; specifically, it wastes a great deal of CPU time just waiting for the device to complete its activity, instead of switching to another ready [[0x01_Process|process]] and thus better utilizing the CPU. 
##### Lowering CPU Overhead With Interrupts
- Instead of polling the device repeatedly, the OS can issue a request, put the calling process to sleep, and context switch to another task. 
- When the device is finally finished with the operation, it will raise a hardware interrupt, causing the CPU to jump into the OS at a predetermined **interrupt service routine (ISR) or an interrupt handler**.
- Note that using **interrupts** is not always the best solution.
- Imagine a device that performs its tasks very quickly: the first poll usually finds the device to be done with task. 
- Using an interrupt in this case will actually slow down the system: switching to another process, handling the interrupt, and switching back to the issuing process is expensive.
- If the speed of the device is not known, or sometimes fast and sometimes slow, it may be best to use a **hybrid** that polls for a little while and then, if the device is not yet finished, uses interrupts.
##### More Efficient Data Movement With DMA
- When using programmed I/O (PIO) to transfer a large chunk of data to a device, the CPU is once again overburdened with a rather trivial task, and thus wastes a lot of time and effort that could better be spent running other processes.
	 ![[DMA_problem.png]]
- The solution to this problem is something we refer to as **Direct Memory Access (DMA)**.
- A DMA engine is essentially a very specific device within a system that can orchestrate transfers between devices and main memory without much CPU intervention.
	 ![[DMA.png]]
- When the DMA is complete, the DMA controller raises an interrupt, and the OS thus knows the transfer is complete.
##### Methods Of Device Interaction
- Over time, two primary methods of device communication have developed.
	1. The first, oldest method is to have explicit **I/O instructions**.
		- These instructions specify a way for the OS to send data to specific device registers and thus allow the construction of the protocols described above. 
		- To send data to a device, the caller specifies a register with the data in it, and a specific port which names the device. Executing the instruction leads to the desired behavior.
		- Such instructions are usually **privileged**.
	2. The second method to interact with devices is known as **memory-mapped I/O**.
		- With this approach, the hardware makes device registers available as if they were memory locations.
		- To access a particular register, the OS issues a load (to read) or store (to write) the address; the hardware then routes the load/store to the device instead of main memory.
##### Fitting Into The OS: The Device Driver
- Our final problem is How can we keep most of the OS device-neutral, thus hiding the details of device interactions from major OS subsystems?
- The problem is solved through the age-old technique of **abstraction**.
- At the lowest level, a piece of software in the OS must know in detail how a device works. We call this piece of software a **device driver**, and any specifics of device interaction are encapsulated within.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 36.
- [Lecture 10 - part 1](https://youtu.be/SQz2CTpI-NM)