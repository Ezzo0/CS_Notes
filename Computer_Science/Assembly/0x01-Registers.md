# Registers

## <span style="color:#92d050">SP - Stack Pointer Register</span>

- #SP register is a stack pointer register that stores the address of the top of the stack.
    - The stack is a region of memory that is used to store data temporarily.
    - The SP register is decremented when data is pushed onto the stack.
    - It allows quick and efficient access to stack data.
    - Located in the general-purpose registers (e.g., R13 in ARM architecture).
    - Operations:
	    - - **Pushing data** onto the stack: When data is pushed onto the stack, the value of the SP register is decremented by the size of the data. The data is then stored at the address that is now pointed to by the SP register.
		- Popping data off the stack: When data is popped off the stack, the value of the SP register is incremented by the size of the data. The data is then removed from the memory address that is now pointed to by the SP register.
		- Calling a function: When a function is called, the return address is pushed onto the stack. The return address is the address of the instruction that should be executed after the function returns.
		- Returning from a function: When a function returns, the return address is popped off the stack and the processor resumes execution at the address that is now pointed to by the SP register.
## <span style="color:#92d050">L<span style="color:#92d050">R - Link Register</span></span>

- #LR register is a link register that stores the return address when a subroutine call is made. This is the address of the instruction that should be executed after the subroutine returns.

> A subroutine is a self-contained sequence of instructions that performs a specific task.

- The <span style="color:#00b0f0">LR registe</span>r is used to quickly and efficiently return from a subroutine. When a subroutine is called, the return address is pushed onto the stack. The return address is then loaded into the LR register. When the subroutine returns, the value of the LR register is loaded into the program counter, which causes the processor to resume execution at the address that is now pointed to by the LR register.
- The LR register is typically located in the general-purpose registers of the processor. In the x86 architecture, the LR register is register 14 (EIP). In the ARM architecture, the LR register is register 14 (R14).
- <span style="color:#00b0f0">Some Operations:</span>
	- Calling a subroutine: When a subroutine is called, the return address is pushed onto the stack and the value of the LR register is loaded into the program counter.
	- Returning from a subroutine: When a subroutine returns, the value of the LR register is loaded into the program counter.
	- It can also used to store temp data.

## <span style="color:#92d050">P<span style="color:#92d050">C - Program Counter</span></span>

- The program counter #PC register is a register that stores the address of the next instruction to be executed.
- The PC register is incremented by 1 after each instruction is executed. This allows the processor to keep track of the current instruction in the program.
- The PC register is also used to jump to other instructions in the program. This can be done by loading a new value into the PC register. For example, when a subroutine is called, the return address is pushed onto the stack and the value of the PC register is loaded with the address of the subroutine. When the subroutine returns, the value of the PC register is popped off the stack and the processor resumes execution at the address that is now pointed to by the PC register.
- <span style="color:#00b0f0">Some Operations</span>
	- - It stores the address of the next instruction to be executed.
	- It is incremented by 1 after each instruction is executed.
	- It is used to jump to other instructions in the program.
	- It is used to store the return address when a subroutine is called.
	- It is used to resume execution after a subroutine returns.
## <span style="color:#92d050">CPSR - Current Program Status Register</span>

- The Current Program Status Register #CPSR is a register that holds the processor status and control information. It is a 32-bit register that is divided into four fields:
    - **Flags**: These bits are set or cleared by arithmetic and logical operations. They indicate the status of the result, such as whether it is negative, zero, or overflowed.
    - **Status:** These bits indicate the processor mode and the state of the interrupt system.
    - **Extension:** These bits are used to extend the functionality of the CPSR.
    - **Control:** These bits control the operation of the processor, such as the enabling or disabling of interrupts.
- ### <span style="color:#00b0f0">Some Operations</span>
	- monitor the status of the processor, such as whether it is in user mode or kernel mode, and whether interrupts are enabled or disabled.
	- To control the execution of instructions, such as by enabling or disabling conditional execution.
	- To control the interrupt system, such as by enabling or disabling interrupts.
	- To implement system calls, which are used to interact with the operating system.
## <span style="color:#92d050">SPSR - Saved Program Status Register</span>

- The Saved Program Status Register #SPSR is a register that stores the processor status and control information when an exception is taken.
- some of the uses of the SPSR:
	- - To save the state of the processor when an exception is taken.
	- To restore the state of the processor after an exception has been handled.
	- To implement exception handling, which is used to handle unexpected events, such as division by zero or invalid memory access.