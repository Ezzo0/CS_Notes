# Explanation 
- A boot-loader is an application that allow a system software to be updated without using a specialized hardware like JTAG programmer
- Communication with boot-loader can be done via different protocols like USART, I2C, CAN, Ethernet, USB...etc.
- Firmware upgrade can be risky, especially when the system power loss occurs during the update process.
- To solve the problem of power loss or other failures during update process, dual bank Flash memories was designed to respond to the above needs.
- The dual bank Flash memory allows a code to be executed in one bank, while another bank is being erased or programmed.
- It avoids a CPU stalling during programming operations and protects the system from power failures or other errors.
- The main features of the Flash dual bank mode are:
	- Read-while-write.
	- Dual boot.
	- Performance and consumption.
	 ![[1 Mbyte Flash memory organization.png]]
- Boot-loader receive new firmware and store it in a RAM buffer and do some operation on this data.
# Sources
- 01 What is the BootLoader and Why do we a BootLoader Part 1
- 02 What is the BootLoader and Why do we a BootLoader Part 2
- 03 What is the BootLoader and Why do we a BootLoader Part 3