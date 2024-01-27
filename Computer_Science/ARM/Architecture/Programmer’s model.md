# Explanation 
- #### Operation modes and states
	- The [[Cortex-M3 & Cortex-M4 Processors]] have two operation states and two modes. 
	- The Cortex-M3 and Cortex-M4 processors can have privileged and unprivileged access levels. 
	- The privileged access level can access all resources in the processor, while unprivileged access level means some memory regions are inaccessible, and a few operations cannot be used.
	 ![[Operation states and modes.png]]
	- **Operation states**
		- Debug state: When the processor is halted, it enters debug state and stops executing instructions.
		- Thumb state: If the processor is running program code (Thumb instructions), it is in the Thumb state.
	- **Operation modes**
		- Handler mode: When executing an exception handler such as an Interrupt Service Routine (ISR). 
			- When in handler mode, the processor always has privileged access level.
		- Thread mode: When executing normal application code, the processor can be either in privileged access level or unprivileged access level. 
			- This is controlled by CONTROL register.
		- Software can switch the processor in privileged Thread mode to unprivileged Thread mode. 
		- It cannot switch itself back from unprivileged to privileged. If this is needed, the processor has to use the exception mechanism to handle the switch.
		- Thread mode and Handler mode have very similar programmer’s models. 
		- Thread mode can switch to using a separate shadowed Stack Pointer (SP),which allows the stack memory for application tasks to be separated from the stack used by the OS kernel, thus allowing better system reliability.
 
- #### Registers
	- Most of the registers are grouped in a unit called the register bank.
	- The register bank in the Cortex-M3 and Cortex-M4 processors has 16 registers.
	- Thirteen of the registers in register bank are general purpose 32-bit registers, and the other three have special uses.
	- **R0 - R12** ^512ca0
		- Registers R0 to R12 are general purpose registers. 
		- The first eight (R0 - R7) are also called low registers.  ^fdc67c
		- Due to the limited available space in the instruction set, many 16-bit instructions can only access the low registers. 
		- The high registers (R8 - R12) can be used with 32-bit instructions, and a few with 16-bit instructions, like [[Instruction set (Assembly)#^d4067d|MOV]] (move). 
		- The initial values of R0 to R12 are undefined.
	- **R13, stack pointer (SP)** ^574a6d
		- R13 is the Stack Pointer. ^093479
		- Physically there are two different Stack Pointers: the Main Stack Pointer (MSP, or SP_main in some ARM documentation) is the default Stack Pointer. 
		- It is selected after reset, or when the processor is in Handler Mode. 
		- The other Stack Pointer is called the Process Stack Pointer (PSP, or SP_process in some ARM documentation).  ^5b7e73
		- The PSP can only be used in Thread Mode. 
		- The selection of Stack Pointer is determined by a CONTROL register. 
		- In normal programs, only one of these Stack Pointers will be visible.
		- Both MSP and PSP are 32-bit, but the lowest two bits of the Stack Pointers (either MSP or PSP) are always zero, and writes to these two bits are ignored. 
		- In ARM Cortex-M processors, PUSH and POP are always 32-bit, and the addresses of the transfers in stack operations must be aligned to 32-bit word boundaries.
		- The initial value of PSP is undefined, and the initial value of MSP is taken from the first word of the memory during the reset sequence.
	- **R14, link register (LR)**
		- R14 is called the Link Register (LR). 
		- This is used for holding the return address when calling a function or subroutine.
		- At the end of the function or subroutine, the program control can return to the calling program and resume by loading the value of LR into the Program Counter (PC). 
		- If a function needs to call another function or subroutine, it needs to save the value of LR in the stack first. Otherwise, the current value in LR will be lost when the function call is made.
		- During exception handling, the LR is also updated automatically to a special EXC_RETURN (Exception Return) value.
		- Although the return address values in the Cortex-M processors are always even, bit 0 of LR is readable and writeable. Some of the branch/call operations require that bit zero of LR (or any register being used) be set to 1 to indicate Thumb state.
	- **R15, program counter (PC)**
		- R15 is the Program Counter (PC). 
		- It is readable and writeable: a read returns the current instruction address plus 4.
		- Writing to PC (e.g., using data transfer/processing instructions) causes a branch operation.
		- Since the instructions must be aligned to half-word or word addresses, the Least Significant Bit (LSB) of the PC is zero. However, when using some of the branch/ memory read instructions to update the PC, you need to set the LSB of the new PC value to 1 to indicate the Thumb state. 
		- Otherwise, a fault exception can be triggered, as it indicates an attempt to switch to use ARM instructions (i.e., 32-bit ARM instructions as in ARM7TDMI), which is not supported. In high-level programming languages (including C, C++), the setting of LSB in branch targets is handled by the compiler automatically.
	 ![[Registers in the register bank.png|450]]
- #### Special registers
	- Besides the registers in the register bank, there are a number of special registers. These registers contain the processor status and define the operation states and interrupt/exception masking.
	  ![[Special Registers.png|450]]
	- Special registers are not memory mapped, and can be accessed using special register access instructions such as MSR and MRS.
		`MRS reg, special_reg; Read special register into register.` 
		`MSR special_reg, reg; write to special register.`
	- **Program status registers**
		- The Program Status Register is composed of three status registers:
			- [[Application Program Status Register (APSR)]]
			- Execution PSR (EPSR)
			- Interrupt PSR (IPSR)
		- These three registers can be accessed as one combined register, referred to as xPSR in some documentation. 
		- In ARM assembler, when accessing xPSR, the symbol PSR is used. For example: MRS r0, PSR; Read the combined program status word. 
		  MSR PSR, r0; Write combined program state word.
		- You can also access an individual PSR (Figure 4.5). For example: `
			`MRS r0, APSR  ; Read Flag state into R0`
			`MRS r0, IPSR  ; Read Exception/Interrupt state`
			`MSR APSR, r0  ; Write Flag state`
		- The ERSR cannot be accessed by software code directly using MRS (read as zero) or MSR.
		- The IPSR is read only and can be read from combined PSR (xPSR).
		 ![[PSR.png|550]]
		 ![[Bit Fields in Program Status Registers.png|550]]
	- **PRIMASK, FAULTMASK, and BASEPRI registers**
		- The PRIMASK, FAULTMASK, and BASEPRI registers are all used for exception or interrupt masking.
		- Each exception (including interrupts) has a priority level where a smaller number is a higher priority and a larger number is a lower priority.
		- These special registers are used to mask exceptions based on priority levels. 
		- They can only be accessed in the privileged access level (in unprivileged state writes to these registers are ignored and reads return zero). 
		- By default, they are all zero, which means the masking (disabling of exception/interrupt) is not active.
		- The _PRIMASK_ register is a 1-bit wide interrupt mask register. 
		- When set, it blocks all exceptions (including interrupts) apart from the Non-Maskable Interrupt (NMI) and the HardFault exception. 
		- Effectively it raises the current exception priority level to 0, which is the highest level for a programmable exception/interrupt.
		- The _FAULTMASK_ register is very similar to PRIMASK, but it also blocks the HardFault exception, which effectively raises the current exception priority level to -1.
		- Unlike PRIMASK, FAULTMASK is cleared automatically at exception return (except return from NMI handler).
		- _BASEPRI_ register masks exceptions or interrupts based on priority level.
		- The width of the BASEPRI register depends on how many priority levels are implemented in the design, which is determined by the microcontroller vendors.
		- When BASEPRI is set to 0, it is disabled. 
		- When it is set to a non-zero value, it blocks exceptions (including interrupts) that have the same or lower priority level, while still allowing exceptions with a higher priority level to be accepted by the processor.
		 ![[PRIMASK, FAULTMASK, and BASEPRI registers.png|450]]
	- **Control register**
		- The CONTROL register defines:
			- The selection of stack pointer (Main Stack Point/Process Stack Pointer).
			- Access level in Thread mode (Privileged/Unprivileged).
		- One bit of the CONTROL register indicates if the currently executed code uses the floating point unit or not.
		 ![[CONTROL register.png|450]]
		- The CONTROL register can only be modified in the privileged access level and can be read in both privileged and unprivileged access levels.
		- After reset, the CONTROL register is 0. This means the Thread mode uses the Main Stack Pointer as Stack Pointer and Thread mode has privileged accesses.
		 ![[Bit Fields in CONTROL Register.png]]
		- If it is necessary to switch the processor back to using privileged access level in Thread mode, then the exception mechanism is needed. During exception handling, the exception handler can clear the nPRIV bit.
		 ![[Switching between privileged thread mode and unprivileged thread mode.png|550]]
		- After modifying the CONTROL register, architecturally an Instruction Synchronization Barrier (ISB) instruction (or `__ISB()` function in CMSIS compliant driver) should be used to ensure the effect of the change applies to subsequent code. 
		- Due to the simple nature of the Cortex-M3, Cortex-M4, Cortex-M0+, Cortex-M0, and Cortex-M1 pipeline, omission of the ISB instruction does not cause any problem.
		- To access the Control register in assembly, the MRS and MSR instructions are used:
			 `MRS r0, CONTROL ; Read CONTROL register into R0`
			 `MSR CONTROL, r0 ; Write R0 into CONTROL register`
	
- #### Floating point registers
	- The Cortex-M4 processor has an optional floating point unit. This provides additional registers for floating point data processing, as well as a Floating Point Status and Control Register (FPSCR).
	- **S0 to S31/D0 to D15**
		- Each of the 32-bit registers S0 to S31 (“S” for single precision) can be accessed using floating point instructions, or accessed as a pair, in the symbol of D0 to D15 (“D” for double-word/double-precision).
		- Although the floating point unit in the Cortex-M4 does not support double precision floating point calculations, you can still use floating point instructions for transferring double precision data.
		 ![[Registers in the floating point unit.png|400]]
	- **Floating point status and control register (FPSCR)**
		- The FPSCR contains various bit fields which:
			- Define some of the floating point operation behaviors
			- Provide status information about the floating point operation results
			 ![[Bit field in FPSCR.png]]
			 ![[Bit Fields in FPSCR.png]]
	- **Memory-mapped floating point unit control registers**
		- The Coprocessor Access Control Register (CPACR) is used to enable or disable the floating point unit. 
		- By default the floating point unit is disabled to reduce power consumption. Before using any floating point instructions, the floating point unit must be enabled by programming the CPACR register (Figure 4.15). 
		- In the C programming environment with a CMSIS-compliant device-driver:
			 `SCB->CPACR j= 0xF << 20; // Enable full access to the FPU`
		![[Bit field in CPACR.png]]
# Sources
- The Definitive Guide to ARM Cortex-M3 and Cortex-M4 Processors 4.1, 4.2, 4.3
- 05_Introduction To ARM Architecture Part5 video
- 06_Introduction To ARM Architecture Part6 video
- 07_Introduction To ARM Architecture Part7 video
- 09_Introduction To ARM Architecture Part9 video
- 10_Introduction To ARM Architecture Part10 video