# Explanation
- #### Cross compilers & Native compilers
	1. **Native compiler:**
		-  Generate code for the same platform on which it runs.
		- Convert the high level language into computer's executable format (for example .exe).
		- Code generation and running the executable happens on the same platform.
	2. **Cross compiler:**
		- Generate executable code for a platform other than the one on which it is running.
- #### Compilation process consists of 3 main stages
	1. **Compilation**
		- The purpose of this stage is to convert source files into object files (Relocatable objects).
		- The compiler woks with one translation unit (.c file) at a time.
		- Compiler is responsible for allocating memory for definitions.
		- Compiler is responsible for generating opcodes from program statements.
		- A librarian tool may be used to take the object files and combine them into a library file.
		- Some compilers convert translation units into assembly files (.s file), then Assembler convert them into object files.
		 ![[archive utility.png|700]]
		- To convert for example main.c to main.o use:
			- `gcc -wall -g -c main.c -o main.o`
			- `-wall` : is a compiler flag that enable all warning during the compilation process.
			- `-g` : is a compiler flag that generate debugging information.
			- `-c` : is a command to only compile or assemble the source file.
			- `-o` : is a command to change the name of the generated object file, if it isn't used, the default name will be "a.o".
			- [Compilation Flags](https://gcc.gnu.org/onlinedocs/gcc-9.4.0/gcc/)
		- **_Compilation stage is broken into 3 parts:_**
			1. Front End: which is responsible for parsing the code.
			2. Middle End: which is responsible for code optimization.
			3. Back End: which is responsible for code generation.
			- ###### Front End
				- Front End consists of 5 steps:
					1. Pre-Processing
						 ![[pre_processing_1.png|600]]
						 ![[pre_processing_2.png|600]]
					2. Whitespace Removal
						- Compiler in this step strip out all whitespaces.
					3. Tokenizing (Lexical Analysis)
						- C program is made up of tokens which may be:
							- Keywords 
							- Constants
							- Identifiers 
							- Strings
							- Operators
							- Special symbols
					4. Syntax Analysis
						- Syntax analysis ensures that the tokens are organized in the correct way, according to the rules of the language.
						- If tokens are not organized in a right way, the compiler produce syntax error.
						- The output of syntax analysis is a data structure known as **Parse Tree** or **Syntax Tree**.
						  ![Engelbart|500](https://ruslanspivak.com/lsbasi-part7/lsbasi_part7_tree_terminology.png)
					5. Intermediate Representation
						- This step isn't mandatory for all compilers. 
						- Intermediate Representations are in the form of **Abstract Syntax Tree or Pseudo code**.
			- ###### Middle End
				- Middle End consists of 2 steps:
					1. Semantic Analysis
						- Semantic Analyzer will check actual meaning of the statement parsed in parse tree.
						- Semantic analysis adds further information to the **IR AST** and performs some checks on the logical structure of the program.
						- The type and amount of semantic analysis performed varies from compiler to another.
						- Semantic analysis can compare information in one part of a **Parse tree** to that in another part.
							- For example: Check if reference to a variable with its declaration or Check if parameters to a function call match the function definition.
						- Semantic Analyzer insert any debug information if the debug information flag was activated.
						- Semantic Analyzer construct program symbol table.
						 ![[symbol_table.png|600]]
					2. Optimization
						-  Optimization transform the code into smaller or faster version.
						- Optimization is a multilevel process.
			- ###### Back End
				- Converts the optimized **IR code structure** into **native opcodes for target platform**.
- #### GNU ARM cross compiler
	- **_arm-none-eabi-gcc.exe_**
		- This is the master driver for entire toolchain.
		- It compiles the code and call linker to link separate object files. 
		- To get flag instructions of compiler use `arm-none-eabi-gcc --help` command.
		- To determine the destination architecture use `arm-none-eabi-gcc processFlag -mcpu=cortex-m4 -mthumb fileName -o newName` command.
	- **_arm-none-eabi-objdump.exe_**
		- It is used to analysis relocatable object file.
		- To see options of this tool use `arm-none-eabi-objdump` command.
		- To display the content of symbol tables use `arm-none-eabi-objdump -t fileName` command.
- #### Relocatable object file
	- The output of compilation stage is Relocatable object file pre one translation unit.
	- Object file contains:
		- File size, data size, source file name
		- Machine architecture specific binary instructions (Opcodes) and data.
		- Symbol table.
		- Debug information, which is used by debugger.
- #### Memory sections
	- C compilers allocate memory for code and data in different sections, each one contains different types of information.
	- Code section is stored in FLASH memory.
	- Data sections are stored in FLASH memory or RAM memory depending on the nature of data.
	- **.text/.code** section contains code that was developed & Literals.
	- **.rodata** section contains read only data (Global constants).
	- **.data** sections in RAM and ROM contains any Global initialized data & any Static initialized variable.
		- Each variable in .data section has 2 addresses, which are LMA & VMA, startup code copies data from LMA to VMA.
			- Load Memory Address is in ROM.
			- Virtual Memory Address is in RAM.
	- **.bss or Block Started by Symbol** segment contains Global uninitialized data & any Static uninitialized variable.
		- This space is initialized to a value with the help of startup code.
	- **.user defined** segment is created by user, can contain data or code, and could be created in RAM or ROM.
	- **.special compiler** section is added by the compiler in ROM memory.
	 ![[ROM_&_RAM_sections.png|700]]
- #### Symbol Tables
	- It is a data structure created and maintained by compilers to store information about entities like variable names, function names, objects, classes, etc. 
	 ![[symboltableRowelements.png|800]]
	 ![[flag char.png|1000]]
	 - The generated address for each symbol is **_relative_** to the file being compiled. 
	 - These addresses are called relative addresses and are generated with respect to offset 0.
	  ![[symboltableandcode.png|1000]]
	 - Global variables and non-static functions are commonly referred as global symbols.
	 - 
- #### Linking Stage
	- The Linker combines object files into a single executable program.
	- Inputs to linker are:
		- Linker command file (Linker Script File).
		- Relocatable object files.
		- Library files:
			- Outputs from Archive utility.
			- C standard library.
	- Outputs from Linker are:
		- Relocatable file.
		- Executable image.
		- Shared object.
		- Map file.
	 ![[Linker generated output files.png]]
	- Linker combines input sections, having the same name, from different object files into a single output section with that name.
	- Linking process involve:
		1. **_Symbol resolution_**
			- It is an iterative process in which linker goes through each object file and determine, for the object file, in which other file or files the external symbols are defined.
			- Sometimes the linker must process the list of object files multiple times trying to resolve all of the external symbols.
			- When external symbols are defined in a static library, the linker copies object files from the library and writes them into the final image. This process is called static linking.
			- Linker always ensures that each symbol defined by the program must has a unique address.
			- If the linker cannot resolve a symbol it will report Unresolved Reference error.
			- If the linker find the same symbol defined in two object files it will report Redefinition error.
		2. **_Symbol relocation_**
			- After completing symbol resolution step, linker associate each symbol reference in the code with exactly one symbol definition.
			- At this point, linker knows the exact sizes of the code and data sections. 
			- After that, linker merges input object files and assign runtime address to each symbol.
	- To call linker use `arm-none-eabi-ld fileName1 fileName2 -o OutputFileName` command.
- #### Linker Script File
	- Linker script file include linker commands that controls the combination and allocation of segments into the target system.
	- The format of linker command file **vary from linker to linker**.
	- The most two common directives among linkers are:
		1. MEMORY:
			- It is used to describe the target system's memory map.
			- To translate memory map into MEMORY directive:
			 ```c
			MEMORY
			{
				RAM(xrw)    : ORGIN = address, LENGTH = its length
				SRAM(xrw)   : ORGIN = address, LENGTH = its length
				FLASH(rx)   : ORGIN = address, LENGTH = its length
			}
		   ```
			- x letter means that section can contain executable code.
			- r letter means that section is Read-only section.
			- w letter mean that section is readable and writable section.
		2. SECTION:
			- This directive describe to linker which input sections are to be merged into which output section.
			- A general notation of SECTIONS command:
			```c
			/* Highest address of the user mode stack */
			_estack = ORIGIN(RAM) + LENGTH(RAM);/* End of "RAM" Ram type memory */
			
			_Min_Heap_Size = 0x200 ;  /* Required amount of heap */
			_Min_Stack_Size = 0x400 ; /* Required amount of stack */
			
			SECTIONS
			{
                /* .text section, The program code and other data into"FLASH"
                    Rom type memory */
				.text :
				{
					. = ALIGN(4);     /*Force aligment to 4 bytes*/
					*(.text)          /* .text sections (code) */
					*(.text*)         /* .text* sections (code) */
					. = ALIGN(4);
					_etext = .;       /* Define a global symbols at end of code */
				} >FLASH
				
				/* .rodata section, Constant data into "FLASH" Rom type memory */
				.rodata :
				{
					. = ALIGN(4);
					*(.rodata)   /* .rodata sections (constants, strings, etc.) */
					*(.rodata*) /* .rodata* sections (constants, strings, etc.) */
					. = ALIGN(4);
				} >FLASH

				/* .data section, Initialized data sections into "RAM" Ram type
					memory */
				.data :
				{
					_sdata = .;       /* Create a global symbol at data start */
					. = ALIGN(4);
					*(.data)          /* .data sections */
					*(.data*)         /* .data* sections */
					. = ALIGN(4);
					_edata = .;       /* Define a global symbol at data end */
				} >RAM AT> FLASH

				/* .bss section, Uninitialized data section into "RAM" Ram type
					memory */
				.bss :
				{
					_sbss = .;         /* Define a global symbol at bss start */
					. = ALIGN(4);
					*(.bss)            /* .bss sections */
					*(.bss*)           /* .bss* sections */
					. = ALIGN(4);
					_ebss = .;         /* Define a global symbol at bss end */
				} >RAM

				/* .User_heap_stack section, used to check that there is enough
					"RAM" Ram  type memory left */
				._user_heap_stack :
				{
					. = ALIGN(8);
					. = . + _Min_Heap_Size;
					. = . + _Min_Stack_Size;
					. = ALIGN(8);
				} >RAM
			}
		   ```
			- `*(.text)` means that linker will combine input sections which names are "text" together.
			- `*(.text*)` means that linker will combine input sections which names are "text" or "text" followed by anything together.
			- `.=ALIGN(4)` means that linker will force alignment into 4 bytes alignment.
			- To define a section for functions or variables use `__attribute__ ((section(".NAME")))` command before variables or function definition.
			- Linker variable `.` always contains the current output location counter.
			- The usage of location counter is inside the sections.
			- Assigning a value to `.` will cause location counter to be removed.
				- This may be used to create holes in output section.
			- _etext, _edata, _sbss & _ebss are linker symbols, which used to store location counter.
			- In addition to linking we can generate map file, which contains all symbols, using `arm-none-eabi-ld -Map MapFileName.map -T LinkerScriptFileName.ld FileName.o FileName.o -o OutputFileName.elf`
			- To change format of the output file use `arm-none-eabi-objcopy -O binary InputFileName.format OutputFileName.newFormat` command.
# Sources
- **Compilation process**
	- 01_Compilation Process (GNU Arm Embedded Toolchain) Part1
	- 02_Compilation Process (GNU Arm Embedded Toolchain) Part2
	- 03_Compilation Process (GNU Arm Embedded Toolchain) Part3
___
- **GNU ARM cross compiler**
	- 04_Compilation Process (GNU Arm Embedded Toolchain) Part4
	- 05_Compilation Process (GNU Arm Embedded Toolchain) Part5
___
- **Memory sections & Relocatable object file**
	- 06_Compilation Process (GNU Arm Embedded Toolchain) Part6
	- 07_Compilation Process (GNU Arm Embedded Toolchain) Part7
___
- **Linking Stage**
	- 09_Compilation Process (GNU Arm Embedded Toolchain) Part9
	- 10_Compilation Process (GNU Arm Embedded Toolchain) Part10
___
- **Linker Script File**
	- 11_Compilation Process (GNU Arm Embedded Toolchain) Part11
	- 12_Compilation Process (GNU Arm Embedded Toolchain) Part12