# Explanation
- #### Moving data within the processor ^d4067d
	- The most basic operation in a microprocessor is to move data around inside the processor.
	 ![[Instructions for Transferring Data within the Processor.png]]
	 ![[Instructions for Transferring Data between the Floating Point Unit and Core.png]]
	- For setting a register in the [[Programmer’s model#^512ca0|general purpose register bank]] to an 8-bit immediate value, the MOVS instruction is sufficient and is used with a 16-bit Thumb instruction if the destination register is a [[Programmer’s model#^fdc67c|low register]].
	- For moving an immediate value into a high register, or if the APSR must not be updated, the 32-bit version of the `MOV`/ `MOVS` instructions would be used.
	- To set a register to a larger immediate value (between 9-bit and 16-bit), the `MOVW` instruction can be used.
___
This File will be completed after watching Assembly video
___
