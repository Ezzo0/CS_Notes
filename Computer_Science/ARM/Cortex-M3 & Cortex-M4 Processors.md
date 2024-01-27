# Explanation 
- The Cortex-M3 and Cortex-M4 processors use a 32-bit architecture. 
- Internal registers in the register bank, the data path, and the bus interfaces are all 32 bits wide. 
- **Instruction set**
	- The Instruction Set Architecture (ISA) in the Cortex-M processors is called the Thumb ISA and is based on Thumb-2 Technology.
	- In classic ARM processors, for example, the ARM7TDM, the processor has two operation states: a 32-bit ARM state and a 16-bit Thumb state.
	- However, There is overhead (in terms of both execution time and instruction count) to switch between the states, and the separation of two states can increase the complexity of the software compilation process (by specifying ARM state or Thumb state in source files). 
	 ![[Switching between ARM code and Thumb code in classic ARM processors.png|500]]
	- Thumb-2 instruction set has been extended to support both 16-bit and 32-bit instruction encoding. 
	- It is now possible to handle all processing requirements without switching between the two different operation states. 
	- The Cortex-M processors do not support 32-bit ARM instructions at all. 
	- Interrupt processing is handled entirely in Thumb state, whereas in classic ARM processors interrupt handlers are entered in ARM state.
	 ![[Instruction set comparison between Cortex-M processors and ARM7TDMI.png|500]]
	 
- **The Cortex-M3 and Cortex-M4 processors have:**
	- Three-stage pipeline design.
	- Harvard bus architecture with unified memory space:
		- It maintains separate instruction and data caches and uses separate buses for accessing instruction memory and data memory, similar to the Harvard architecture. 
		- However, it uses a unified memory space where instructions and data are stored together in a single physical memory.
		-  This design simplifies the memory management and reduces the complexity of the system.
	- 32-bit addressing, supporting 4GB of memory space.
	- On-chip bus interfaces based on ARM AMBA (Advanced Microcontroller Bus Architecture) Technology, which allow pipelined bus operations for higher throughput.
	- An interrupt controller called NVIC supporting up to 240 interrupt requests and from 8 to 256 interrupt priority levels.
	- Support for various features for OS implementation such as a system tick timer, shadowed stack pointer.
	- Sleep mode support and various low power features.
	- Support for an optional MPU (Memory Protection Unit) to provide memory protection features like programmable memory, or access permission control.
	- Support for bit-data accesses in two specific memory regions using a feature called Bit Band.
	- The option of being used in single processor or multi-processor designs.

- **The ISA used in Cortex-M3 and Cortex-M4 processors provides a wide range of instructions:**
	- General data processing, including hardware divide instructions.
	- Memory access instructions supporting 8-bit, 16-bit, 32-bit, and 64-bit data, as well as instructions for transferring multiple 32-bit data.
	- Instructions for bit field processing.
	- Multiply Accumulate (MAC) and saturate instructions.
	- Instructions for branches, conditional branches and function calls.
	- Instructions for system control, OS support, etc.

- **The Cortex-M4 processor also supports:**
	- Single Instruction Multiple Data (SIMD) operations.
	- Additional fast MAC and multiply instructions.
	- Saturating arithmetic instructions.
	- Optional floating point instructions (single precision).
- **Processor architecture**
	- ARM Cortex-M processors are regarded as RISC processors.
	- The Cortex-M3 and Cortex-M4 processors are based on ARMv7-M architecture.  ^554ea7
	- In ARM processors, the term architecture can refer to two areas:
		- Architecture: Instruction Set Architecture (ISA), programmer’s model, and debug methodology.
		- Micro-architecture: Implementation-specific details such as interface signals, instruction execution timing, pipeline stages. 
		- Micro-architecture is processor design-specific.
	- An ISA can be implemented with various implementations of micro-architecture.
- **Advantages of the Cortex-M processors**
	1. Low Power
		- Compared to other 32-bit processor designs, Cortex-M processors are relatively small. 
		- The Cortex-M processor designs are also optimized for low power consumption.
		- Cortex-M processors also include support for sleep mode features.
	2. Performance
		- The Cortex-M3 and Cortex-M4 processors can deliver over 3 CoreMark/MHz.
	3. Energy efficiency
		- We can still do a lot of processing, with a limited supply of energy.
	4. Code density
		- The Thumb ISA provides excellent code density, which means achieving the same tasks with smaller program size.
	5. Debug features
		- Besides standard design features, which you can find in most microcontrollers like halting and single stepping, you can also generate a trace to capture program flow, data changes, profiling information, and so on.
	6. OS support
- **Block diagram**
	 ![[Block diagram of the Cortex-M3 and Cortex-M4 processor.png]]
# Sources
- The Definitive Guide to ARM Cortex-M3 and Cortex-M4 Processors -Chapter 1-
- The Definitive Guide to ARM Cortex-M3 and Cortex-M4 Processors 3.1
- 01_Introduction To ARM Architecture Part1 video
- 02_Introduction To ARM Architecture Part2 video
- 03_Introduction To ARM Architecture Part3 video
- 04_Introduction To ARM Architecture Part4 video