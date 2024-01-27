# Explanation
- CMake is a tool which generate [[Makefile]].
- #### CMake basic commands:
	- `cmake_minimum_required(VERSION X.Y.Z)` - determine the required version for CMake tool. It will generate error if the version of the tool is less than X.Y.Z.
	- `project(project_name)` - specify a name for the project.
	- `add_executable(exe_name file_1.c file_2.c ....file_n.c)` - generate an exe file from all .c files.
	- `target_include_directories(exe_name access_permision directory_path/)` -  include the paths of your different directories.
	- `#` - used to make a comment.
	- `set (variable_name value)` - set a value to variable.
		- `add_executable(exe_name ${variable_name})` - an example to store the required files for the executable file in a variable.
	- `message("message_text")` - to print a value on the screen
		- `message("message text ${variable_name}")` - to print the value of a variable.
	- `cmake --help` - is used to get reference manual.
	- `CMAKE_SOURCE_DIR` - is a built-in variable used to get the path of your CMake file.
		- `message("CMake path is ${CMAKE_SOURCE_DIR}")` - print the path of your CMake file.
	- **_Conditions:_**
		```
		if(condition)
			command1
		else()
			command2
		endif()
		```
		- Notice that `if` must end with `endif`.
	- `add_library(<name> [STATIC | SHARED | MODULE] [EXCLUDE_FROM_ALL [<source>...])` - Adds a library target called `<name>` to be built from the source files listed.
	- `add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL] [SYSTEM])` - adds a subdirectory to the build. The `source_dir` specifies the directory in which the source `CMakeLists.txt` and code files are located.
	- `target_link_libraries(<target> ... <item>... ...)` - link library with target.
	- you can send a value to a variable in CMake by using: 
		- `cmake -Dvariable_name=value`.
	- **_Header FIle generation:_**
		- `configure_file(file_name.h.in generated_file_name.h)` - used to generate header file.
		- **Example of definitions in header file:**
			- `#define productCompany "value"` - constant value.
			- `#define productYear "${var_name}"` - var_name is a variable in CMake file.
			- `project(project_name VERSION 2.1)` | `#define project_name_VERSION_MAJOR @project_name_VERSION_MAJOR@`.
	- 
# Sources
- [CMake - MoatasemElsayed](https://www.youtube.com/playlist?list=PLkH1REggdbJpG8fHZvivt-5Hlg3UZcJrK)
- [CMake Reference Documentation](https://cmake.org/cmake/help/latest/index.html)