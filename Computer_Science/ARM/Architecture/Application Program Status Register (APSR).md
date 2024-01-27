# Explanation
- The APSR contains several groups of status flags:
	- Status flags for integer operations (N-Z-C-V bits).
	- Status flags for saturation arithmetic (Q bit).
	- Status flags for SIMD operations (GE bits).
- #### Integer status flags
	- The integer status flags are very similar to ALU status flags in many other processor architectures.
	- In the [[Cortex-M3 & Cortex-M4 Processors#^554ea7|ARMv7-M]] and ARMv7E-M architecture, most of the 16-bit instructions affect these four ALU flags. 
	- In most of the 32-bit instructions one of the bits in the instruction encoding defines if the APSR flags should be updated or not.
	- Some of these instructions do not update the V flag or the C flag. For example, the MULS (multiply) instruction only changes the N flag and the Z flag.
	 ![[ALU Flags on the Cortex-M Processors.png]]
	- In addition to conditional branch or conditional execution code, the Carry bit of APSR can also be used to extend add and subtract operations to over 32 bits. For example, when adding two 64-bit integers together, we can use the carry bit from the lower 32-bit add operation as an extra input for the upper 32-bit add operation.
- #### Q status flag
	- The Q is used to indicate an occurrence of saturation during saturation arithmetic operations or saturation adjustment operations.
	- After this bit is set, it remains set until a software write to the APSR clears the Q bit.
- #### GE bits
	- The “Greater-Equal” (GE) is a 4-bit wide field in the APSR in the Cortex-M4, and is not available in the Cortex-M3 processor. 
	- It is updated by a number of SIMD instructions where, in most cases, each bit represents positive or overflow of SIMD operations for each byte.
	- For SIMD instructions with 16-bit data, bit 0 and bit 1 are controlled by the result or lower half-word, and bit 2 and bit 3 are controlled by the result of upper half-word.
# Sources
- The Definitive Guide to ARM Cortex-M3 and Cortex-M4 Processors 4.3
- 07_Introduction To ARM Architecture Part7 video