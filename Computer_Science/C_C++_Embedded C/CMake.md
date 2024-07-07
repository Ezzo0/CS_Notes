# Explanation
- CMake is a tool which generate [[Makefile]].
- #### CMake basic commands:
	- `cmake_minimum_required(VERSION X.Y.Z)` - determine the required version for CMake tool. It will generate error if the version of the tool is less than X.Y.Z.
	- `project(project_name)` - specify a name for the project.
	- `add_executable(exeFile_name file_1.c file_2.c ....file_n.c)` - generate an exe file from all .c files.
	- `target_include_directories(exeFile_name access_permision directory_path/)` -  include the paths of your different directories.
	- `#` - used to make a comment.
	- `set (variable_name values)` - set a value or many values to variable.
		- `add_executable(exeFile_name ${variable_name})` - an example to store the required files for the executable file in a variable.
	- `message("message_text")` - to print a value on the screen
		- `message("message text ${variable_name}")` - to print the value of a variable.
	- `cmake --help` - is used to get reference manual.
	- `CMAKE_SOURCE_DIR` - is a built-in variable used to get the path of your CMake file.
		- `message("CMake path is ${CMAKE_SOURCE_DIR}")` - print the path of your CMake file.
	- **_Conditions:_**
		```CMake
		if(condition)
			command1
		elseif()
			command2
		else()
			command3
		endif()
		```
		- Note that `if` must end with `endif`.
	- `add_library(<name> [STATIC | SHARED | MODULE] <sourceFiles>...)` - Adds a library target called `<name>` to be built from the source files listed.
	- `add_subdirectory(source_directory [binary_dir] [EXCLUDE_FROM_ALL] [SYSTEM])` - adds a subdirectory to the build (Parent directory). The `source_directory` specifies the directory in which the source `CMakeLists.txt` and code files are located.
	- `target_link_libraries(<exeFile> library)` - link library with target.
	- you can send a value to a variable in CMake by using: 
		- `cmake -Dvariable_name=value`.
	- **_Header File generation:_**
		- `configure_file(file_name.h.in generated_file_name.h)` - used to generate header file.
		- **Example of definitions in header file:**
			- `#define productCompany "value"` - constant value.
			- `#define productYear "${var_name}"` - var_name is a variable in CMake file.
			- `project(project_name VERSION 2.1)` | `#define project_name_VERSION_MAJOR @project_name_VERSION_MAJOR@`.
	- **_Loops:_**
		```CMake
		# Foreach
		set(lst A b d e l r a h m a n E z z)
		foreach(f in lists lst)
			message("${f}")
		endforeach()
		
		# While
		set(var 10)
		while(var)
			message("${var}")
			math(EXPR var "${var} - 1")
		endwhile()
		# break command
		break()
		#continue command
		continue()
		```
	- **_Functions:_**
		```CMake
		function(function_name arg1)
			message("arg1: ${arg1}") #print name of the arg
			message("arg1: ${${arg1}}") # print the value of the arg
			message("argN: ${ARGN}") # print extra args
			message("arg_count: ${ARGC}") # print the number of passed arguments
		endfunction()
		set(var 2 3 4 5)
		function_name(var jan)
		```
	- To store a user defined variable with its value in CMakeCache use:
		- `option(var "comment above the line in cache file" val)`
		- `set(var value CACHE var_dataType "comment above the line in cache file")`
	- `target_compile_definitions(exe_file access_permision "var=val")` - used to generate definitions without generating config file.
	- `target_compile_options(exe_file access_permision target_option)` - used to specify compilation options.
	```CMake
	install(FILES "${PROJECT_BINARY_DIR}/hellobin.exe"
	    DESTINATION "${PROJECT_BINARY_DIR}/executable"
	)
	```
	- used to copy specified files to new destination.
		- Execute of the `install` command is by using `cmake --install .`
	- **_Read from files:_**
		- `file(READ "file_name" var)` - read txt in a file and store it in variable `var` in the same format as the file.
		- `file(STRINGS "file_name" var REGEX "val")` - read string in a file and store it in variable `var` and search for `"val"` using regular expressions.
		- `file(GLOB_RECURSE var "*.cpp (any input)")` - read string in a file or files and store it in variable `var`.
		```CMake
		file(READ "main.cpp" mainTxt)
		string(REGEX REPLACE "int" "void" OUT ${mainTxt})
		```
		- read the content of the file and replace every int word with void, then store the result in `OUT` variable.
	- **_Macro:_**
	```CMake
		macro(function_name parameters)
			commands
		endmacro()
	``` 
	-  `execute_process(COMMAND "command")` - used to run a command, which is used in bash, in CMake.
	- 
# Sources
- [CMake - MoatasemElsayed](https://www.youtube.com/playlist?list=PLkH1REggdbJpG8fHZvivt-5Hlg3UZcJrK)
- [CMake Reference Documentation](https://cmake.org/cmake/help/latest/index.html)
