#### What is an OS
- An operating system is a software that:
	- Manages hardware and resources.
	- Allows interaction with hardware to perform useful task.
#### Unix
- Unix is a family of operating systems
- Popular Unix-based OSs include:
	- Oracle Solaris
	- FreeBSD
	- HP-UX
	- IBM AIX
	- Apple macOS
- **Unix beginnings:**
	- Original Unix OS was created, in the 1960s, for a specific hardware system which is PDP-7 computer.
	- In the 1970s, the Unix OS was rewritten in C, distinguishing itself from other systems and making it portable to many hardware architectures.
	- In the late 1970s, UC Berkley developed Berkley Software Distribution (or BSD), an add-on to Unix providing additional software and capabilities. 
	- The famous macOS was later derived from BSD.
#### What is Linux
- Linux is a family of Unix-like operating systems.
- Linux was developed as an effort to create a free, open source version of the Unix OS.
- **Linux Features are:**
	- Free and open source.
	- Secure.
	- Multi-user.
	- Multitasking.
	- Portability.
- **Linux beginnings:**
	- In the 1980s, GNU was developed at MIT. GNU stands for “GNU’s not Unix” and was made as a free, open source set of the existing Unix system tools.
	- In 1991, Linus Torvalds developed a free, open source version of the Unix kernel called  Linux. 
	- The kernel is the core component of an OS that enables components to communicate  with the machine’s hardware.
	- In 1992, the potential of unifying GNU and the Linux kernel was realized as popular Linux operating systems began to appear. 
	- In 1996, a computer scientist named Larry Ewing created ”Tux” the penguin, which was later adopted by Linus Torvalds as the official Linux mascot.
#### Linux Distribution (Linux Distro)
- Linux distribution is a specific flavor of the Linux OS.
- All Linux distribution must use the Linux kernel.
- **Linux distro differences:**
	- Each Linux distro includes a unique set of default utilities that are part of the OS such as commands and apps that come prepacked with the distribution.
	- Each distro has its own GUI.
	- Each distro supports a specific set of commands that can be used in a shell, a window for entering and receiving output from commands.
	- Different levels of support:
		- It could be a Long-term support (LTS) or a rolling release, where stable package updates are released on a rolling schedule.
		- It could be developed and managed as a community-backed project or maintained by a commercial enterprise.
- **Linux Distros:**
	1. Debian:
		- Debian is one of the earliest-rooted distros.
		- It is stable, reliable, and fully open source.
		- It supports many computer architectures.
		- Debian is the largest community-run distro currently available.
	2. Ubuntu:
		- It is Debian-based.
		- It has 3 editions:
			- Ubuntu Desktop
			- Ubuntu Server
			- Ubuntu Core for IoT.
	3. Red Hat Linux:
		- Red Hat Linux, like Debian, is a core Linux distro, meaning that isn't derived from another Linux distro.
		- It is stable, reliable, and fully open source.
		- It’s managed by Red Hat, an IBM subsidiary. 
		- Today it ships as Red Hat Enterprise Linux (or RHEL), an edition focused entirely on enterprise customers.
	4. Fedora:
		- It is stable and supports many architectures.
		- It is very reliable and secure, offering unique firewall and security features.
		- Actively developed with a large and growing community.
		- Sponsored by Red Hat.
	5. SUSE:
		- SUSE Linux Enterprise is available in two editions: 
			- Server, or SLES. 
			- Desktop, or SLED. 
		- It supports many architectures, such as ARM for Raspberry Pi.
		- Uses the SUSE Package Hub, which enables users to install packages that aren’t officially part of SLE. 
	6. Arch Linux:
		- The unique, do-it-yourself approach of Arch Linux allows users to customize every part of their system. 
		- Need a strong understanding of Linux and system tools.
		- Arch isn’t focused on stability in the same way as other distros, it has easy access to the newest software, which has little guarantee of being completely stable. 
#### Linux Layers
- The outermost layer of the Linux architecture is the UI, or user interface, which allows users to interact with the system using a keyboard or mouse.
- The application layer includes system daemons, shells, user apps, and tools used to perform tasks in a Linux system. The applications communicate with the operating system to perform tasks. 
- The OS is responsible for jobs that are vital for system stability such as job scheduling and keeping track of time. 
- All Linux operating systems are built on top of the Linux kernel, which performs the most vital lower-level jobs. The kernel is the core component of the operating system and is responsible for managing memory, processing, and security. 
- The kernel interacts with the hardware layer, which includes all the physical or electronic devices in the computer such as processors, memory modules, input devices, and storage.
	 ![[LinuxLayers.png]]
#### Linux Filesystem
- The Linux filesystem is the collection of files on your machine.
- The top level of the filesystem is the root directory, symbolized by a forward slash (/). 
- Below this is a tree-like structure of the directories and files in the system.
- Filesystem assigns appropriate access rights to the directories and files.
- The very top of the Linux filesystem is the root directory, which contains many other directories and files:
	1. Binary files ( /bin) : 
		- Binary files contain the code your machine reads to run programs and execute commands. 
	2. Other key directories include /usr, which contains user programs.
	3. /home, which is your personal working directory where you should store all your personal files.
	4. /boot, which contains your system boot files, the instructions vital for system startup.
	5. /media, which contains files related to temporary media such as CD or USB drives that are connected to the system.
#### Linux Shell
- The Linux shell is an OS-level application that interprets commands.
- Shell commands can be used to perform tasks such as:
	- moving and copying files.
	- writing to and reading from files.
	- extracting and filtering data, and searching for data.
- There are many shell versions, but the base functionality of most is the same. Some popular examples include: Bash And Zsh.
- Interaction with the Linux shell through a Linux terminal. 
- A terminal is an application, or user interface, where you enter the commands you want to run and receive any output from those commands.
	 ![[terminal and shell.png]]
#### Paths in Linux Filesystem
- There are also special paths: 
	- A single tilde symbol (~) refers to the user’s home directory. 
	- A single slash (/) at the beginning of a path refers to the root directory. 
	- Two periods (..) refer to the parent of the current directory. 
	- A single period (.) refers to the current directory.
- The `cd` command stands for change directory.
- Enter `cd /` to go to the root directory.
- To change the current directory back to the directory you were in previously use  `cd -`.
- The `ls` command is used to display in the terminal window the names of all files and directories within the current directory.
- The `pwd` command prints the path name for our present working directory on the next line.
#### Packages & Package mangers
- Both software updates and software installation files for Linux OSs are distributed in files known as packages.
- Packages are archive files containing the required components for either installing new software or updating existing software.
- Package managers are used to manage the download and installation of packages. 
- Different Linux distros provide different package managers—some are GUI-based and some are command line tools.
- **Deb and RPM packages:**
	- Deb and RPM packages are used by package managers in Linux operating systems. 
	- They are distinct file types containing software or updates for different Linux operating systems. 
	- .deb (stands for Debian.) files are used for Debian-based distributions such as Debian, Ubuntu, and Mint. 
	- .rpm (stands for Red Hat Package Manager.) files are used for Red Hat-based distributions such as CentOS/RHEL, Fedora, and openSUSE.
	- Deb and RPM formats are equivalent, so the contents of the file can be used on other types of Linux OSs. 
	- If you find that a package that you want to use is only available in the other format you can convert it using the alien tool. 
	- To convert packages from RPM format to deb `alien <package name>.rpm`.
	- To convert packages from deb format to RPM `alien -r <package name>.deb`.
- Package managers can automatically resolve dependencies between packages.
#### Updating & Installation
- The `sudo`, "super-user do", command is used to activate the powerful `apt` command.
- **_For deb-based Linux:_**
	- `apt` (_Advanced Packaging Tool_) is used to perform system administration operations such as installing software packages, upgrading existing packages, and updating your system's _package list index_.
	- To find available packages for your distro use `sudo apt update` command.
		- The output of this command lists each available package, builds a dependency tree, and lets you know how many packages can be upgraded.
	- To install the packages, use the `sudo apt upgrade` command.
	- If you want to only install a specific package, you can use `sudo apt upgrade <package-name>`.
	- To install a software use `sudo apt install <package-name>`.
- **_For RPM-based Linux:_**
	- yum, stands for Yellow dog Updater, Modified, is a command-line tool for updating RPM-based systems.
	- To update all packages in your system, type `sudo yum update`. After you enter your password, Yum fetches all available package updates.
	- If you want to only install a specific package, you can use `sudo yum upgrade <package-name>`.
	- To install a software use `sudo yum install <package-name>`.
#### Shell
- A shell is a powerful user interface for Unix-like operating systems. 
- It can interpret commands and run other programs. 
- A shell, which enables access to files, utilities, and applications, is also an interactive language.
- A shell is also a scripting language. And it can be used to automate tasks.
- The default shell on Linux systems is usually Bash, which stands for ‘Bourne again shell.’
- To find out what the default shell is, enter `printenv SHELL` on the command line. This returns the path to the default shell program.
- If your default shell is not Bash, you can always switch to it, simply by entering `bash` on the command line.
- **_Applications of shell commands include:​_** 
	1. Getting information. 
		- _Some common shell commands for getting information include:_
		- `whoami` – which returns the user's username. 
		- `id` – which returns the current user, and group IDs. 
			- `id -u` – return the numerical ID of the user.
			- `id -u -n` – the name corresponding to the numerical user ID.
		- `uname` – returns the operating system name. 
			- `uname -s -r` – return both the OS name and its version.
			- `uname -v` – to view more detailed version information about OS.
			- `uname -a` – to prints all the system information.
		- `ps` – displays running processes and their IDs. 
			- `ps -e` – display all of the processes running on the system. The includes processes owned by other users.
		- `top` – displays running processes and resource usage including memory, CPU, and IO. 
			- `top -n 3` –will show a table of running processes and their resource usage. 
			- The number “3” to display the top three running tasks.
			- By default, tasks are sorted by CPU usage.
		- `df` – shows information about mounted file systems. 
			- `df -h ~` –displays the following table, which is specific to your home directory, represented by the tilde symbol.
				 ![[df.png]]
			- In Linux, you can “mount” a disk onto a directory, which means that the file system of that disk becomes accessible through that directory.
			- `df -h` –To view disk usage on all filesystems.
				 ![[df-h.png]]
		- `man` – fetches the reference manual for any shell command. 
			- `man command_name` – To see the "man" page for a command.
		- `date` – prints today's date. 
	2. Navigating and working with files and directories. 
		- _Some common shell commands for working with files include:_ 
		- `cp /source/file /dest/file` – copy file. 
			- `cp -r /source/dir/ /dest/dir/` – To copy entire directories.
		- `mv /source/file /dest/file` – change file name or path. 
		- `rm file_name` – remove file. 
		- `touch file_name` – create empty file, update file timestamp. 
		- `chmod file_name` – change or modify file permissions. 
		- `wc file_name` – get count of lines, words, characters in file. 
			- `wc -l file_name` – To view only line count.
			- `wc -w file_name` – To view only word count.
			- `wc -c file_name` – To view only character count.
		- `grep pattern file_name` – which stands for “global regular expression print”, returns lines of a file matching a specified pattern, such as a regular expression.
			- `grep -i pattern file_name` – `-i` option expands the pattern search by making it case-insensitive.
	1. Printing file and string contents​. 
		- _Very common shell commands for navigating and working with directories include:_
		- `ls` – lists the files and directories in the current directory. 
			- `ls -l` –listed all the child files, directories, and additional details, such as permissions, last-modified date, and owner.
		- `find` – used to find files matching a pattern in the current directory tree.
			- It return the path to every file that matches a user-specified criterion.
			- `find . -name "file_name.txt"` – The first  `.` argument means “search within here,” so the command will only search within your current working directory.
			- `find . -iname "file_name.txt"` – To perform a case-insensitive version of the search.
		- `pwd` – get the present working directory. 
		- `mkdir dir_name` – makes a new directory, 
		- `cd` – changes the current directory to another directory. 
		- `rmdir dir_name` – removes an entire directory. 
	2. File compression and archiving​. 
		- _For printing file contents or strings, common commands include:_
		- `cat file_name` – which prints the entire contents of a file. 
		- `sort file_name` – sorts the lines of a file alpha-numerically and prints the sorted result to standard output.
			- `sort -r file_name` – sorted in the reverse order.
		- `more file_name` – view a file's content in a page by page format. 
		- `uniq file_name` – filter out the repeated lines. Note that the `uniq` command only removes duplicated lines if they are consecutive.
		- `cut` – command to extract specific sections from each line in your file.
			- `cut -C 2-9 file_name` – extract the second to ninth characters from every line.
			- `cut -d 'delimiter' -f2 file_name` – specify that the field delimiter, or the character that indicates the break between fields. `-f2` option to return the second field from each line.
		- `paste file_name file_name` – merge lines from multiple files
			- `paste -d "delimiter" file_name file_name`– specify a delimiter other than Tab (the default delimiter of `paste` command).
		- `head file_name` – for printing just the first 10 lines of a file. 
			- `head -n desired_number file_name` – specify the number of lines you would like head to return.
		- `tail file_name` – for printing the last 10 lines of a file. 
			- `tail -n desired_number file_name` – specify the number of lines you would like tail to return.
		- `echo` – which 'echoes' an input string by printing it. It can also ‘echo’ the value of a variable.
		- _Shell commands related to file compression and archiving applications include:_ 
		- `tar -cf archieved_file_name.tar file_or_dir_name` – which is used to archive and de-archive files and directories.
			- `-c` option means create new achieve.
			- `-f` flag tells tar to interpret its input from the file rather than from the default, which is standard input.
			- `tar -czf archieved_file_name.tar.gz file_or_dir_name` – archive to be compressed.
				- `-z` option, which filters the archive file through a GNU compression program called g-zip.
				- Adding the suffix `.gz` to the output name ensures that Windows-based programs will correctly recognize the file type.
			- `tar -tf archieved_file.tar` – check the contents of the archived file.
			- `tar -xf archieved_file_name.tar optional_dest_name` – de-archive the archived files.
				- `-x` option tells tar to extract file and directory objects from the archive.
			- `tar -xzf archieved_file_name.tar.gz optinal_dest_name` – decompress the `.tar.gz` file.
		- `zip -r compressed_file_name.zip file_or_dir_name` – which is used to to compress files and directories.
		- `unzip compressed_file_name.zip` – which extracts files from a compressed or zipped archive.
	3. Performing network operations. 
		- _Networking applications include the following:_
		- `hostname` – prints the host name. 
		- `ping` – sends packets to a URL and prints the response. 
		- `ifconfig` – displays or configures network interfaces on the system. 
		- `curl` – displays the contents of a file located at a URL. 
		- `wget` –  can be used to download a file from a URL. 
	4. Monitoring performance and status of the system, its components and applications. 
	5. Running batch jobs, such as ETL operations.
#### Shell Scripting
- A script is a list of commands that can be interpreted and run by a program called scripting language. 
- Commands can be entered interactively at the command line, or listed line by line in a text file.
- Scripting languages are usually not compiled. They are interpreted at runtime. 
- Scripts are generally slower to run than compiled languages, but they are also much easier and faster to develop. 
- Scripts are widely used to automate processes, such as ETL jobs, file backups and archiving, and general system administration tasks.
- A shell script is an executable text file in which the first line usually has the form of an interpreter directive.
- The interpreter directive is also known as a ‘shebang’ directive, and has the following form:
	- `#!interpreter [option-arg]` 
		- Interpreter is an absolute path to an executable program.
		- the optional argument is a string representing a single argument.
- Shell scripts are scripts that invoke a shell program. For example:
	- `#!/bin/sh` invokes the Bourne shell or other compatible shell program, from the bin directory.
	- `#!/bin/bash` "shebang" invokes the Bash shell.
- **_Create Simple Shell Script_**
       ![[simpleShellScript.png]]
- Shell variables offer a powerful way to store and later access or modify information such as numbers, character strings, and other data structures by name.
	 ```
	 $ firstname=Jeff
	 $ echo $firstname
	 Jeff
	 ```
	- The first line assigns the value `Jeff` to a new variable called `firstname`. 
	- The next line accesses and displays the value of the variable, using the `echo` command along with the special character `$` in front of the variable name to extract its value, which is the string `Jeff`.
- Reading user input into a shell variable at the command line:
	```
	$ read lastname
	Grossman
	$ echo $lastname
	Grossman
	``` 
	- The value `Grossman` has just been stored in the variable `lastname` by the `read` command.
- You can add comments to the script using `#`. 
- **_Filters & Pipe:_**
	- Filters or shell commands or programs which take their input from standard input normally the keyboard and return their output to standard output, which is normally the terminal.
	- There are many examples, including `wc`, `cat`, `more`, `head`, `sort`, `grep`, and so on.
	- The value of filters is that they can be chained together which brings us to the pipe command.
	- The pipe command, denoted by a vertical slash `|`, immensely extends the functionality of shells.
	- It allows you to chain together sequences of filter commands.
	- The use pattern looks like this: `command1 | command2 `
		- The output of command 1 becomes the input of command 2, and so on 
			- (`[command 1] | [command 2] | [command 3] ... | [command n]`).
		- For example: `ls | sort -r`.
- **_Variables:_**
	- Shell variables are variables that are limited in scope to the shell in which they were created.
	- shells cannot see each other's shell variables.
	- `set` – list all variables and their definitions that are visible to the current shell.
	- `var_name=value` – define a new shell variable.
	- `unset var_name` – delete the variable.
	- Environmental variables are just like shell variables, except they have extended scope.
	- `export var_name` – extend shell variable to be an environment variable.
	- `env` – list all environment variables.
- **_Metacharacters:_**
	- Metacharacters are special characters that have meaning to the shell.
	- `#` – is used to include comments that the shell ignores.
	- `;` – separates commands typed on the same line.
	- `*` – represents any number of consecutive characters within a filename pattern.
	- `?` – acts as a single-character version of `*` metacharacter.
    ![[metacharacters.png|400]]
- **_Quoting:_**
	- Quoting specifies whether the shell should interpret special characters as metacharacters or ‘escape’ them.
	- `\` – escape the interpretation of a single character as a metacharacter.
		- The backslash tells Bash to interpret the dollar sign, in the figure, as text rather than the default variable name, so the output is literally "dollar one each."
	- `"Any_text"` – interpret the text literally, except for any metacharacters, which will be interpreted according to their special meanings.
		- The "dollar one" expression, without the preceding backslash, is interpreted as a variable, which in this case is empty, as we see in the figure.
	- `'Any_text'` – interpret all contents as literal characters.
		- The expression in the figure returns the same result as the first example above.
       ![[Quoting.png|400]]
- **_I/O redirection:_**
	- Refers to a set of features used for redirecting either the standard input, the keyboard, or the standard output, the terminal.
	- `>` – is used to redirect the standard output of a command to a file.
		- It also creates the file if it doesn’t exist and overwrites its contents if it already exists.
	- `>>` – append output to existing content.
	- `2>` – redirects an error message to a file.
	- `2>>` – append the error message file.
	- `<` – is used to pass file contents as input to the standard input.
- **_Command substitution:_**
	- `$(command)` or `` `command` `` – replace a command with its output.
       ![[commandSub.png]]
- **_Command Line Arguments:_**
	- Command line arguments are arguments used by a program specified on the command line.
	- In particular, they provide a way to pass arguments to a shell script, a program.
	- Example: `./MyBashScript.sh arg1 arg2`.
- **_Batch V.S Concurrent modes:_**
	- Batch mode, which is the usual mode, runs commands sequentially.
		- `command1; command2`
		- Command 2 only runs after command 1 is completed.
	- In concurrent mode, commands run in parallel.
		- `command1 & command2`
		- The ampersand operator, after command 1, directs command 1 to operate in the background and passes control to command 2 in the foreground.
#### Job Scheduling
- The _cron_ utility on Linux and Unix-like operating systems allows you to run scheduled jobs.
- _Cron_ is the general name of the tool that runs scheduled jobs consisting of shell commands or shell scripts.
- _Crond_ is the daemon or service that interprets “crontab files” every minute and submits the corresponding jobs to cron at scheduled times.
-  _Crontab_, short for “cron table,” is a file containing jobs and schedule data.
	- Crontab is also a command that invokes a text editor to allow you to edit a crontab file.
	- Entering `crontab -e` on the command line opens the default text editor.
	- Using the editor, you can specify a new schedule and a command, which has the following syntax `m h dom mon dow command`
		- "command" can be any shell command, including a call to a shell script.
		- The symbols `m h dom mon dow` stand for minute, hour, day of month, month, and day of week.
		- All five positions must have either a numeric entry or `*`, which is a wildcard symbol that means any.
	- `crontab -l` – returns a list of all cron jobs and their schedules.
#### Conditionals
- `if` statements, are a way of telling a script to do something only under a specific condition.
- Bash script conditionals use the following `if`-`then`-`else` syntax:
	```
	if [ condition ]
	then
		statement_block_1
	else
		statement_block_2
	fi
	```
- Every `if` condition block must be paired with a `fi` to tell Bash where the condition block ends.
- In the following example, the condition is checking whether the number of command-line arguments read by some Bash script, `$#`, is equal to 2.
	```
	if [[ $# == 2 ]]
	then
		echo "number of arguments is equal to 2"
	else
		echo "number of arguments is not equal to 2"
	fi
	```
- Notice the use of the double square brackets, which is the syntax required for making integer comparisons in the condition `[[ $# == 2 ]]`.
- You can also make string comparisons. 
	- For example, assume you have a variable called `string_var` that has the value `"Yes"` assigned to it. Then the following statement evaluates to `true`: 
		- `` `[ $string_var == "Yes" ]` ``
- You can also include multiple conditions to be satified by using the "and" operator `&&` or the "or" operator `||`. For example:
	- `if [ condition1 ] && [ condition2 ]`
	- `if [ condition1 ] || [ condition2 ]`
- The following logical operators can be used to compare integers within a condition in an `if` condition block.
	- `==`: is equal to
	- `!=`: is not equal to
	- `<=`: is less than or equal to. You can use the equivalent notation `-le` in place of `<=`:
#### Arithmetic calculations
- You can perform integer addition, subtraction, multiplication, and division using the notation `$(())`.
	```
	a=3
	b=2
	c=$(($a+$b))
	echo $c
	```
- Bash natively handles integer arithmetic but does not handle floating-point arithmetic. As a result, it will always truncate the decimal portion of a calculation result.
#### Arrays
- To create an array, declare its name and contents:
	- `my_array=(1 2 "three" "four" 5)`
	- This statement creates and populates the array `my_array` with the items in the parentheses: `1`, `2`, `"three"`, `"four"`, and `5`.
- You can also create an empty array by using: `declare -a empty_array` 
- If you want to add items to your array after creating it, you can add to your array by appending one element at a time: `my_array+=(element)`
- By using indexing, you can access individual or multiple elements of an array:
	```
	# print the first item of the array:
	echo ${my_array[0]}
	
	# print all array elements:
	echo ${my_array[@]}
	``` 
#### loops
- For loop examples:
	```
	for i in ${!my_array[@]}; do
	echo ${my_array[$i]}
	done
	```
	```
	for i in ${!my_array[@]}; do
	echo $i
	done
	```
	```
	N=6
	for (( i=0; i<=$N; i++ )) ; do
		echo $i
	done
	```
	- The `for` loop requires a `; do` component in order to cycle through the loop.
	- Additionally, you need to terminate the `for` loop block with a `done` statement.
