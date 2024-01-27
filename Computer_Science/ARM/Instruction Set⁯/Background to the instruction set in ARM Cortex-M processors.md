# Explanation
- Early ARM processors (prior to the ARM7TDMI processor) supported a 32-bit instruction set called the ARM instruction set. 
- ARM instruction set evolved for a few years, progressing from ARM architecture version 1 to version 4.
- In 1995, a new 16-bit instruction set called “Thumb” was introduced in ARM7TDMI processor.
	- The ARM7TDMI can operate in the ARM state, the default state, and also in the Thumb state. 
	- During operation, ARM7TDMI switches between ARM state and Thumb state under software control. 
	- Parts of the application program are compiled with ARM instructions for higher performance, and the remaining parts are compiled as Thumb instructions for better code density. 
	- By providing this two state mechanism, the applications can be squeezed into a smaller program size, while maintaining high performance when needed. 
	- In some cases, the Thumb code provides a code size reduction of 30% compared to the equivalent ARM code.
	- In the ARM7TDMI processor design, a mapping function is used to translate Thumb instructions into ARM instructions for decoding so that only one instruction decoder is needed.
- Although the Thumb instruction set can provide most of the same commonly used functionality as the ARM instructions, it have some limitations, such as restrictions on the register choices for operations, available addressing modes, or a reduced range of immediate values for data or addresses.
- The instruction set design of the Cortex-M processors is upward compatible from Cortex-M0, to Cortex-M4.
- When an operation can be carried out in 16-bit, the compiler will normally choose the 16-bit version to give a smaller code size. 
- The 32-bit version in ARMv6-M might support a greater choice of registers (e.g., high registers), larger immediate data, longer address range, or a larger choice of addressing modes. However, for the same operation, the 16-bit version and the 32-bit version of an instruction will take the same amount of time to execute.
# Sources
- The Definitive Guide to ARM Cortex-M3 and Cortex-M4 Processors 5.1
- The Definitive Guide to ARM Cortex-M3 and Cortex-M4 Processors 5.2