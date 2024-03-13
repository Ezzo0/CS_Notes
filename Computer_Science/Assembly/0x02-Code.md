```arm-asm
.global _start
_start: ; these two lines indicates to the start of the program.
/* the _start is a label that indicate the assembler he can work from here
The .global directive tells the assembler that the symbol _start is 
accessible from other modules.*/

	MOV R0,#0x30 ; it write 0x30 in the R0 the accumulator register.

	MOV R7,#1  
	 /* moves the value 1 into the R7 register. The R7 register is the interrupt 
register, and it is used to control interrupts. 
The value 1 in the R7 register means
that the program is requesting to exit.*/

	SWI 0        /* it's a software interrupt that interrupt the program and let the 
 operating system takes over.*/

```

> [!NOTE] Note
> ASSEMBLY is not a case-sensitive language.


<span style="color:#92d050">Addressing modules</span>
```armasm
.global _start
_start:
	MOV R0,#0X5
	MOV R7,R0   ;This is called registered direct addressing.
	LDR R0,=list ; loads the address in R0
	LDR R1, [R0] /*loads the address of the list variable into the R0 register, 
loads the value of the first element of the array into the R1 register,
 and loads the value of the second element of the array 
into the R2 register.*/
	LDR R2, [R0,#4]!

.data
	list:
		  .word 4,5,-1,6,-5
```
### <span style="color:#92d050">explanation</span>
- The  #LDR instruction is used to load a value from memory into a register. The first operand of the instruction is the register `R0` that will store the value. The second operand is the address of the memory location that contains the value`[R0]`.
- The `R0` and `R1` are a general-purpose register that can be used to store any value.
- The `list` variable is a global variable that is declared in the `.data` section of the program. The `.word` directive tells the assembler to allocate 4 bytes of memory for the `list` variable. The values 4, 5, -1, 6, and -5 are stored in the `list` variable in that order.
- The `!` symbol in the `LDR R2, [R0,#4]!` instruction tells the assembler to increment the value of the R0 register by 4 after the value of the second element of the array has been loaded into the R2 register. This is done so that the next time the `LDR` instruction is executed, it will load the value of the third element of the array into the R2 register.
<span style="color:#92d050">Arithmetic and CPSR flags</span> :
```armasm
; Add two values and store the result in register R0.
add R0, R1, R2

; Subtract two values and store the result in register R0.
sub R0, R1, R2

; Multiply two values and store the result in register R0.
mul R0, R1, R2

; Divide two values and store the result in register R0.
div R0, R1, R2

; Compare two values and set the CPSR flags based on the result.
cmp R1, R2

; Branch to a different part of the program if the result of the previous operation is zero.
beq label
```
---
```
; Add two values and set the CPSR flags based on the result.
adds R0, R1, R2

; Subtract two values and set the CPSR flags based on the result.
subs R0, R1, R2

; Add two values and the carry flag and set the CPSR flags based on the result.
adcs R0, R1, R2

; Subtract two values and the carry flag and set the CPSR flags based on the result.
sbbs R0, R1, R2

; Compare two values and set the CPSR flags based on the result.
cmps R1, R2
```
- The condition flags are a set of flags that indicate the result of the previous arithmetic or logical operation. The condition flags can be used to control the flow of execution of a program. For example, the `subs` instruction can be used to set the `Z` flag if the result of the subtraction is zero. The `Z` flag can then be used to branch to a different part of the program if the result is zero.
## <span style="color:#92d050">Logical Operators:</span>
```armasm
; Perform a bitwise AND operation on two values and store the result in register R0.
and R0, R1, R2

; Perform a bitwise OR operation on two values and store the result in register R0.
or R0, R1, R2

; Perform a bitwise XOR operation on two values and store the result in register R0.
xor R0, R1, R2
```
<span style="color:#00b0f0">Logical operators can also be used to set and test bits.</span> 
```
; Set the least significant bit of register R0.
or R0, R0, #1
```
- The following code will test the least significant bit of register R0 and branch to a different part of the program if the bit is set:
```armasm
; Test the least significant bit of register R0.
tst R0, #1

; Branch to label if the least significant bit of register R0 is set.
beq label
```
## <span style="color:#92d050">Logical Shift</span>
```armasm
; Multiply r0 by 2
lsl r0, r0, #1

; Extract the most significant bit from r0
lsr r1, r0, #31

; Convert an 8-bit unsigned integer to a 16-bit unsigned integer
lsl r2, r2, #8
```

`MOV R1, R0 LSL #1`
	- moves the value of register R0 into register R1, after shifting the value to the left by one position.
- <span style="color:#92d050">Branching and loops</span> 
	

| B   | Unconditional branch                     |
| --- | ---------------------------------------- |
| BL  | Branch and link (used to call functions) |
| BAL | =                                        |
| BEQ | Branch on equal                          |
| BNE | Branch on not equal                      |
| BGT | Branch on greater than                   |
| BLT | Branch on less than                      |
| BGE | Branch on greater than or equal          |
| BLE | Branch on less than or equal             |

```armasm
; Implement a loop
loop:
    ; Do something
    B loop

; Implement a conditional statement
if_condition:
    CMP R0, #10
    BGT greater_than_10

; Do something if R0 is greater than 10
greater_than_10:
    ; Do something else

; Call a function
BL my_function

; Return from a function
MOV PC, LR
```
## Functions and #LR register
```armasm
; Declare a subroutine
my_subroutine:
    ; Do something
    BX LR

; Call the subroutine
BAL my_subroutine

; Continue executing the main program
```
# Control Hardware
```armsasm
.equ Switch, 0xff200040 
.equ led, 0xff200000
.equ se_seg, 0xff200020
.global _start
_start:
	
	ldr R0, =Switch
	ldr R1, [R0]
	
	ldr R0, =se_seg
	str R1,[R0]
```
this code declare variables `Switch` , `led` , `se_seg` and stores the address of the hardware

then load it <span style="color:#92d050">in </span>`R0` then it read the `Swith` as input, and store the value inside `R1` into the reference of the seven-segment.