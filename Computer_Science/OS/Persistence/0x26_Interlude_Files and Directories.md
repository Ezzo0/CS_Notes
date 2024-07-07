# Explanation
- A persistent-storage device, such as a classic **[[0x24_Hard Disk Drives|hard disk drive]]** or a more modern **solid-state storage device**, stores information permanently (or at least, for a long time).
##### Files And Directories
- Two key abstractions have developed over time in the virtualization of storage.
	1. The first is the **file**.
		- A file is simply a linear array of bytes, each of which you can read or write.
		- Each file has some kind of **low-level name**, usually a number of some kind.
		- The low-level name of a file is often referred to as its **inode number (i-number)**.
	2. The second abstraction is that of a **directory**. 
		- A directory, like a file, also has a low-level name (i.e., an inode number), but its contents are quite specific.
		- Directory contains a list of entries(user-readable name, low-level name).
		- Each entry in a directory refers to either files or other directories.
		- By placing directories within other directories, users are able to build an arbitrary **directory tree** (or **directory hierarchy**), under which all files and directories are stored.
		- The directory hierarchy starts at a **root directory** and uses some kind of **separator** to name subsequent **sub-directories** until the desired file or directory is named.
			 ![[Directory Tree.png|500]]
		- Directories and files can have the same name as long as they are in different locations in the file-system tree.
##### Creating Files
- This can be accomplished with the `open` system call; by calling `open()` and passing it the `O_CREAT` flag, a program can create a new file.
```C
int fd = open("foo", O_CREAT|O_WRONLY|O_TRUNC, S_IRUSR|S_IWUSR);
```
- The routine `open()` takes a number of different flags. 
- In this example, the second parameter creates the file (`O_CREAT`) if it does not exist, ensures that the file can only be written to (`O_WRONLY`), and, if the file already exists, truncates it to a size of zero bytes thus removing any existing content (`O_TRUNC`).
- The third parameter specifies permissions, in this case making the file readable and writable by the owner.
- The older way of creating a file is to call `creat()`, as follows:
```C
// option: add second flag to set permissions
int fd = creat("foo");
```
- One important aspect of `open()` is what it returns: a **file descriptor**.
- A file descriptor is just an integer, private per process, and is used in UNIX systems to access files; thus, once a file is opened, you use the file descriptor to read or write the file, assuming you have permission to do so.
- Another way to think of a file descriptor is as a pointer to an object of type file.
- File descriptors are managed by the operating system on a per-process basis. 
- This means some kind of simple structure (e.g., an array) is kept in the `proc` structure on UNIX systems. Here is the relevant piece from the xv6 kernel:
```C
struct proc {
	...
	struct file *ofile[NOFILE]; // Open files
	...
};
```
- A simple array (with a maximum of `NOFILE` open files), indexed by the file descriptor, tracks which files are opened on a per-process basis.
- Each entry of the array is actually just a pointer to a `struct file`.
##### Reading And Writing Files
```C
prompt> strace cat foo
...
open("foo", O_RDONLY|O_LARGEFILE)      = 3
read(3, "hello\n", 4096)               = 6
write(1, "hello\n", 6)                 = 6
hello
read(3, "", 4096)                      = 0
close(3)                               = 0
...
prompt>
```
- Why does the first call to `open()` return 3, not 0 or perhaps 1 as you might expect?
- As it turns out, each running process already has three files open, **standard input** (which the process can read to receive input), **standard output** (which the process can write to in order to dump information to the screen), and **standard error** (which the process can write error messages to).
- These are represented by file descriptors 0, 1, and 2, respectively. Thus, when you first open another file, it will almost certainly be file descriptor 3.
- After the open succeeds, `cat` uses the `read()` system call to repeatedly read some bytes from a file. 
- The first argument to `read()` is the file descriptor, thus telling the file system which file to read; a process can of course have multiple files open at once, and thus the descriptor enables the operating system to know which file a particular read refers to.
- The second argument points to a buffer where the result of the `read()` will be placed; in the system-call trace above, `strace` shows the results of the read in this spot.
- The third argument is the size of the buffer, which in this case is 4 KB.
- The call to `read()` returns successfully as well, here returning the number of bytes it read (6, which includes 5 for the letters in the word `hello` and one for an end-of-line marker).
- The `cat` program then tries to read more from the file, but since there are no bytes left in the file, the `read()` returns 0 and the program knows that this means it has read the entire file.
- Thus, the program calls `close()` to indicate that it is done with the file “foo”, passing in the corresponding file descriptor.
##### Reading And Writing, But Not Sequentially
- Sometimes, however, it is useful to be able to read or write to a specific offset within a file. To do so, we will use the `lseek()` system call. Here is the function prototype: `off_t lseek(int fildes, off_t offset, int whence);`
- The first argument is a file descriptor. 
- The second argument is the offset, which positions the **file offset** to a particular location within the file.
- The third argument, called `whence`, determines exactly how the seek is performed.
- The offset is kept in that `struct file` we saw earlier, as referenced from the `struct proc`. Here is a (simplified) xv6 definition of the structure:
```C
struct file {
	int ref;
	char readable;
	char writable;
	struct inode *ip;
	uint off;
};
```
- These file structures represent all of the currently opened files in the system; together, they are sometimes referred to as the **open file table**.
- The xv6 kernel just keeps these as an array, with one lock for the entire table:
```C
struct {
	struct spinlock lock;
	struct file file[NFILE];
} ftable;
```
##### Shared File Table Entries: `fork()` And `dup()`
- The mapping of file descriptor to an entry in the open file table is a one-to-one mapping. 
- For example, when a process runs, it might decide to open a file, read it, and then close it; in this example, the file will have a unique entry in the open file table. Even if some other process reads the same file at the same time, each will have its own entry in the open file table.
- In this way, each logical reading or writing of a file is independent, and each has its own current offset while it accesses the given file.
- However, there are a few interesting cases where an entry in the open file table is shared. One of those cases occurs when a parent process creates a child process with `fork()`.
```C
int main(int argc, char *argv[]) {
	int fd = open("file.txt", O_RDONLY);
	assert(fd >= 0);
	int rc = fork();
	if (rc == 0) {
		rc = lseek(fd, 10, SEEK_SET);
		printf("child: offset %d\n", rc);
	} else if (rc > 0) {
		(void) wait(NULL);
		printf("parent: offset %d\n",
				(int) lseek(fd, 0, SEEK_CUR));
	}
	return 0;
}
```
- When we run this program, we see the following output:
```c
prompt> ./fork-seek
child: offset 10
parent: offset 10
prompt>
```
- Figure 39.3 shows the relationships that connect each process’s private descriptor array, the shared open file table entry, and the reference from it to the underlying file-system inode.
	 ![[Open File Table Entry.png|500]] 
- Note that we finally make use of the **reference count** here. When a file table entry is shared, its reference count is incremented; only when both processes close the file (or exit) will the entry be removed.
- The `dup()` call allows a process to create a new file descriptor that refers to the same underlying open file as an existing descriptor.
```c
int main(int argc, char *argv[]) {
	int fd = open("README", O_RDONLY);
	assert(fd >= 0);
	int fd2 = dup(fd);
	// now fd and fd2 can be used interchangeably
	return 0;
}
```
##### Writing Immediately With `fsync()`
- Most times when a program calls write(), it is just telling the file system: please write this data to persistent storage, at some point in the future. 
- The file system, for performance reasons, will **buffer** such writes in memory for some time at that later point in time, the write(s) will actually be issued to the storage device.
- When a process calls `fsync()` for a particular file descriptor, the file system responds by forcing all dirty (i.e., not yet written) data to disk, for the file referred to by the specified file descriptor. The `fsync()` routine returns once all of these writes are complete.
```C
int fd = open("foo", O_CREAT|O_WRONLY|O_TRUNC, S_IRUSR|S_IWUSR);
assert(fd > -1);
int rc = write(fd, buffer, size);
assert(rc == size);
rc = fsync(fd);
assert(rc == 0);
```
- Interestingly, this sequence does not guarantee everything that you might expect; in some cases, you also need to `fsync()` the directory that contains the file `foo`. 
- Adding this step ensures not only that the file itself is on disk, but that the file, if newly created, also is durably a part of the directory.
##### Renaming Files
- To give a file a different name, `mv` command is used.
- Using strace, we can see that `mv` uses the system call `rename(char *old, char *new)`, which takes precisely two arguments: 
	1. The original name of the file (`old`). 
	2. The new name (`new`).
- One interesting guarantee provided by the `rename()` call is that it is (usually) implemented as an **atomic** call with respect to system crashes.
##### Getting Information About Files
- Beyond file access, we expect the file system to keep a fair amount of information about each file it is storing. We generally call such data about files **metadata**.
- To see the metadata for a certain file, we can use the `stat()` or `fstat()` system calls. These calls take a pathname (or file descriptor) to a file and fill in a stat structure.
```C
struct stat {
dev_t       st_dev;        // ID of device containing file
ino_t       st_ino;        // inode number
mode_t      st_mode;       // protection
nlink_t     st_nlink;      // number of hard links
uid_t       st_uid;        // user ID of owner
gid_t       st_gid;        // group ID of owner
dev_t       st_rdev;       // device ID (if special file)
off_t       st_size;       // total size, in bytes
blksize_t   st_blksize;    // blocksize for filesystem I/O
blkcnt_t    st_blocks;     // number of blocks allocated
time_t      st_atime;      // time of last access
time_t      st_mtime;      // time of last modification
time_t      st_ctime;      // time of last status change
};
```
- Each file system usually keeps this type of information in a structure called an **inode**(Some file systems call these structures similar, but slightly different, names, such as dnodes)
##### Removing Files
- How do you delete files? just run the program `rm`. But what system call does `rm` use to remove a file?
- `unlink()` just takes the name of the file to be removed, and returns zero upon success.
##### Making Directories
- You can never write to a directory directly. Because the format of the directory is considered file system metadata, the file system considers itself responsible for the integrity of directory data; thus, you can only update a directory indirectly by, for example, creating files, directories, or other object types within it.
- To create a directory, a single system call, `mkdir()`, is available.
- When such a directory is created, it is considered “empty”, although it does have a bare minimum of contents. 
- Specifically, an empty directory has two entries: one entry that refers to itself, and one entry that refers to its parent. The former is referred to as the “.” (dot) directory, and the latter as “..” (dot-dot).
##### Reading Directories
- Now that we’ve created a directory, we might wish to read one too.
- Instead of just opening a directory as if it were a file, we instead use a new set of calls. Below is an example program that prints the contents of a directory. The program uses three calls, `opendir()`, `readdir()`, and `closedir()`.
```C
int main(int argc, char *argv[]) {
	DIR *dp = opendir(".");
	assert(dp != NULL);
	struct dirent *d;
	while ((d = readdir(dp)) != NULL) {
		printf("%lu %s\n", (unsigned long) d->d_ino, d->d_name);
	}
	closedir(dp);
	return 0;
}
```
- The declaration below shows the information available within each directory entry in the `struct dirent` data structure:
```C
struct dirent {
char               d_name[256];     // filename
ino_t              d_ino;           // inode number
off_t              d_off;           // offset to the next dirent
unsigned short     d_reclen;        // length of this record
unsigned char      d_type;          // type of file
};
```
##### Deleting Directories
- You can delete a directory with a call to `rmdir()` (which is used by the program of the same name, `rmdir`). 
- Unlike file deletion, however, removing directories is more dangerous, as you could potentially delete a large amount of data with a single command. 
- Thus, `rmdir()` has the requirement that the directory be empty (i.e., only has “.” and “..” entries) before it is deleted. If you try to delete a non-empty directory, the call to `rmdir()` simply will fail.
##### Hard Links
- The `link()` system call takes two arguments, an old pathname and a new one; when you “link” a new file name to an old one, you essentially create another way to refer to the same file.
- The command-line program `ln` is used to do this, as we see in this example:
```C
prompt> echo hello > file
prompt> cat file
hello
prompt> ln file file2
prompt> cat file2
hello
```
- The way `link()` works is that it simply creates another name in the directory you are creating the link to, and refers it to the same inode number (i.e., low-level name) of the original file.
- The file is not copied in any way; rather, you now just have two human-readable names that both refer to the same file.
- When you create a file, you are really doing two things. 
	1. First, you are making a structure (the inode) that will track virtually all relevant information about the file, including its size, where its blocks are on disk, and so forth. 
	2. Second, you are linking a human-readable name to that file, and putting that link into a directory.
- When the file system unlinks file, it checks a **reference count** within the inode number.
- This reference count (sometimes called the **link count**) allows the file system to track how many different file names have been linked to this particular inode. 
- When `unlink()` is called, it removes the “link” between the human-readable name (the file that is being deleted) to the given inode number, and decrements the reference count. 
- Only when the reference count reaches zero does the file system also free the inode and related data blocks, and thus truly “delete” the file.
##### Symbolic Links
- There is one other type of link that is really useful, and it is called a **symbolic link** or sometimes a **soft link**.
- Hard links are somewhat limited: you can’t create one to a directory (for fear that you will create a cycle in the directory tree); you can’t hard link to files in other disk partitions (because inode numbers are only unique within a particular file system, not across file systems); etc.
- To create such a link, you can use the same program `ln`, but with the `-s` flag.
- Symbolic links are actually quite different from hard links. 
	1. The first difference is that a symbolic link is actually a file itself, of a different type.
	2. Because of the way symbolic links are created, they leave the possibility for what is known as a **dangling reference**
		- Quite unlike hard links, removing the original file named file causes the link to point to a pathname that no longer exists. 
##### Permission Bits And Access Control Lists
- The file system presents a virtual view of a disk, transforming it from a bunch of raw blocks into much more user-friendly files and directories.
- However, the abstraction is notably different from that of the CPU and memory, in that files are commonly shared among different users and processes and are not (always) private.
- Thus, a more comprehensive set of mechanisms for enabling various degrees of sharing are usually present within file systems.
- The first form of such mechanisms is the classic UNIX **permission bits**.
- To see permissions for a file foo.txt, just type:
```C
prompt> ls -l foo.txt
-rw-r--r-- 1 remzi wheel  0 Aug 24 16:29 foo.txt
```
- We’ll just pay attention to the first part of this output, namely the `-rw-r--r--`.
- The first character here just shows the type of the file: `-` for a regular file (which foo.txt is), `d` for a directory, `l` for a symbolic link, and so forth.
- We are interested in the permission bits, which are represented by the next nine characters. These bits determine, for each regular file, directory, and other entities, exactly who can access it and how.
- The permissions consist of three groupings: what the **owner** of the file can do to it, what someone in a **group** can do to the file, and finally, what anyone (sometimes referred to as **other**) can do.
- The abilities the owner, group member, or others can have include the ability to read the file, write it, or execute it.
- In the example above, the first three characters of the output of `ls` show that the file is both readable and writable by the owner (`rw-`), and only readable by members of the group wheel and also by anyone else in the system (`r--` followed by `r--`).
- The owner of the file can readily change these permissions, for example by using the `chmod` command (to change the **file mode**).
- The execute bit is particularly interesting. For regular files, its presence determines whether a program can be run or not.
- For directories, the execute bit behaves a bit differently. Specifically, it enables a user (or group, or everyone) to do things like change directories into the given directory, and, in combination with the writable bit, create files therein.
- Beyond permissions bits, some file systems, such as the distributed file system known as AFS, include more sophisticated controls.
- AFS, for example, does this in the form of an **access control list (ACL)** per directory. Access control lists are a more general and powerful way to represent exactly who can access a given resource.
- In a file system, this enables a user to create a very specific list of who can and cannot read a set of files, in contrast to the somewhat limited owner/group/everyone model of permissions bits described above.
##### Making And Mounting A File System
- There is one more topic we should discuss: how to assemble a full directory tree from many underlying file systems. 
- This task is accomplished via first making file systems, and then mounting them to make their contents accessible.
- To make a file system, most file systems provide a tool, usually referred to as `mkfs`, that performs exactly this task.
- The idea is as follows: give the tool, as input, a device (such as a disk partition, e.g., `/dev/sda1`) and a file system type (e.g., `ext3`), and it simply writes an empty file system, starting with a root directory, onto that disk partition.
- However, once such a file system is created, it needs to be made accessible within the uniform file-system tree. This task is achieved via the `mount()` system call.
- What mount does, quite simply is take an existing directory as a target **mount point** and essentially paste a new file system onto the directory tree at that point.
# Sources
- Operating Systems: Three Easy Pieces - Chapter 39.
- [Lecture 11 - part 4]()