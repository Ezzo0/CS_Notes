# Explanation
- In [[Cortex-M3 & Cortex-M4 Processors]], there can be three types of reset:
	1. Power on Reset
		- Reset everything in microcontroller including processor and its debug support and peripherals
	2. System Reset
		-  Reset the processor and peripherals without resetting debug support components of the processor. 
	3. Processor Reset
		- Reset only the processor.
	- Debug host can generate system reset and processor reset
- The duration of power on reset and system reset depends on the microcontroller design, through a register in System Control Block (SCB).
- Bootloader is the first application to run after power on reset.
- Bootloader is an optional app. If it doesn't exist the compiled program is the first app to run after resetting.
- One of Bootloader tasks is to jump to the application code, if it is existed, by checking the application valid bytes.
- The first instruction in application code is reset handler function.
- Reset Handler is part of startup code, which is executed before `main()` function.
- Startup code is responsible initializing microcontroller by doing some steps:
	1. Disable global interrupt.
	2. Initialize stack pointer.
	3. Initialize .bss segment to zeros.
	4. Copy content of .data segment in Flash to .data in SRAM.
	5. Initialize interrupt vector table by ISR addresses.
	6. Enable global interrupt.
	7. Call `main()` function.
	 ![[reset_sequence.png|800]]
- After reset and before the processor starts executing the program, the cortex-M processors read the first two words from the beginning of the memory space, which contains the [[Exceptions and interrupts#^c2826c|Vector Table]].
- First two words in vector table are:
	1. Initial value of [[Programmer’s model#^093479|Main Stack Pointer]].
	2. Reset vector, which is the starting address of reset handler.
- The setup of the MSP is necessary because some exceptions such as NMI and HardFault handlers could potentially occur shortly after reset, and the stack pointer will be needed to push some registers to stack before exception handling.
- In classic ARM processors, vector table holds instruction code instead of address values.
# Sources
- 01_Reset Mechanism and Reset Sequence Part1
- 02_Reset Mechanism and Reset Sequence Part2