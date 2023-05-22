---
layout: post
title: The Linux Programming Interface Chapter 5 File I/O Further Details
categories: tlpi
---

[5.1 Atomicity and Race Conditions](#5-1-atomicity-and-race-conditions)

[5.3 Open File Status Flags](#5-3-open-file-status-flags)

[5.4 Relationship Between File Descriptors and Open Files](#5-4-relationship-between-file-descriptors-and-open-files)

[5.5 Duplicating File Descriptors](#5-5-duplicating-file-descriptors)

[5.6 File IO at Specified Offset with pread and pwrite](#5-6-file-io-at-specified-offset-with-pread-and-pwrite)

[5.7 Scatter Gather IO with readv and writev](#5-7-scatter-gather-io-with-readv-and-writev)


# 5 1 Atomicity and Race Conditions
![Image](/docs/assets/images/5.1-atomicity.png)
* Syscalls = atomic, uninterrupted
    * Avoids race conditions
* O_EXCL and O_CREAT flags with open() call
    * Returns error if file already exists, otherwise creates it (checks if it exists & otherwise creates: does this in one atomic operation)
    * Process can ensure it is the creator of a file
        * We can understand why by analyzing bad_exclusive_open below
* Fileio/bad_exclusive_open.c
    * What this code does:
        * Tries to open a file
            * If file does not exist, file is opened.
            * If file does exist, file is created.
                * Prints PID of the process creating the file, inside the loops.
    * Bad_exclusive_open might evaluate the if-statement and notice the file does not exist, but then the scheduler might let another program run in the meantime. This other program could then create the file. Then the scheduler could go back to bad_exclusive_open. Now the bad_exclusive_open runs the open() command to create the file if it does not exist, thinking it created it.
    * The above is a race condition example.
    * The above could also happen in a system with multiple parallel processors (as is common today), not necessarily a kernel process scheduler thing.
    * Testing the issue: Make one program sleep for 5 seconds, run two instances of the program simultaneously; both will claim they exclusively created the file.

* Another example of atomicity being required:
    * Multiple processes appending to global log file.
    * First lseek() to end of file, then write() to end of file.
    * If another process runs between lseek() and write() and appends some data to the global log file, when the first process resumes, it will overwrite that data because lseek() put them at the same end-of-file position (which changes while the second process ran and appended data).
Problem solved with: open() with O_APPEND flag.

# 5 3 Open File Status Flags

	• Retrieve/Modify status flags of an open file.
		○ Int flags = fcntl(fd, F_GETFL);
			§ F_GETFL gets flags.
		○ We can check if file was opened with synchronized writes, for instance:
			§ If (flags & O_SYNC)
				□ Printf("synced for writes");
	• Checking access mode flags is more complex
		○ Because O_RDONLY, O_WRONLY, O_RDWR consist of multiple (i.e. not single) bits in the open file status flags
		○ Mask flags value with O_ACCMODE constant:
			§ Int accessmode = flags & O_ACCMODE
			§ If (accessmode == O_WRONLY || accessmode == O_RDWR)
				□ Printf("file is writable");
	• Modify open file status flags with fcntl() F_SETFL cmd.
		○ Used in the following scenarios:
			§ File was not opened by the calling program
			§ The file descriptor stems from a pipe() or socket() call, not open().
		○ To update flags:
			1. Get flags with GET_FL
			2. Modify flags
			3. Update flags with SET_FL
		○ Can modify O_APPEND, O_NONBLOCK, O_NOATIME, O_ASYNC, O_DIRECT.
        * See the code on the right.

```c
int flags = fcntl(fd, F_GETFL);
if (flags == -1) errExit();
flags |= O_APPEND;
if fcntl(fd, SET_FL, flags) == -1
  errExit();
printf("success");
```

# 5 4 Relationship Between File Descriptors and Open Files

![Image](/docs/assets/images/tlpi-5.4-fd.png)

	• Can have multiple file descriptors for the same file (e.g. different processes)
	• 3 data structures maintained by kernel:
		○ Per-process file descriptor table
			§ List of open FD's for each process. Table entry consists of:
				□ a reference to the open FD
				□ A set of flags
		○ System-wide table of all open file descriptors
			§ Table entry consists of:
				□ Current file offset
				□ Status flags
				□ File access mode (read, write, read-write etc)
				□ Settings related to signals driven I/O
				□ A reference to the i-node object for this file
		○ File system i-node table
			§ Consists of:
				□ File type (regular file, socket, FIFO) and permissions
				□ Pointer to a list of locks held on the file
				□ File properties (size, timestamps of different file operations etc)
	• Reasons for multiple fd's for same files:
		○ Fork() could lead to process A and B pointing to same file descriptor (fd 2 process A, fd 2 proc B)
		○ Fd 0 in pA and fd3 in pB => but they point to same i-node (e.g. they each called open() on the same file)
		○ Implications:
			§ Because file offset is system-wide, if pA for fd2 changes file offset, pB's fd2 is also impacted.
			§ Similar impact on open file status flags (e.g. O_APPEND, O_ASYNC etc) using fcntl() F_GETFL and F_SETFL commands.
            * File descriptor flags are private to the process (e.g. close-on-exec)

# 5 5 Duplicating File Descriptors

	• 2>&1 tells BASH we want standard error (fd2) redirected to same place as standard output (fd1).
		○ ./myscript > results.log 2>&1
			§ Sends std output and std error to the file results.log
			§ The shell sets fd2 to point to the same open file as fd1 by duplicating fd1.
			§ Opening the results.log file twice is not sufficient: file offsets etc would not be synced i.e. stdoutput and stderror would overwrite each other.
			§ Dup() takes an oldfd, returns new fd that points to the same open fd.
			§ Dup2(int oldfd, int newfd);
				□ To specify exactly which newfd number you want, e.g.: dup2(1, 2);
					® This returns the value of the 2nd file descriptor
					® Returns with error EBADF
				□ Can also use fcntl(oldfd, F_DUPED, startfd);
					® startFD denotes the starting number of your desired fd number, if not available, a higher number is returned
			§ Dup can always be replicated using close() and fcntl()
			§ Dup3(int oldfd, int newfd, int flags) for optional flags such as O_CLOEXEC (close-on-exec flag)

# 5 6 File IO at Specified Offset with pread and pwrite

	• Ssize_t Pread(int fd, void *buf, size_t count, off_t offset);
		○ Returns # bytes read, 0 on EOF, -1 on error
		○ Like read() but at a specified offset
		○ Equivalent to atomically calling the following:
			§ Orig = lseek() //save current offset
			§ Lseek(offset) // set pointer to specified offset
			§ S = read(…) // read data
			§ Lseek(orig) // set pointer to original offset
	• Ssize_t pwrite(int fd, const void *buf, size_t count, off_t offset);
		○ Returns # bytes written, -1 on error
	• Useful in multithreaded applications
		○ Threads in a process all use the same file descriptor table
		○ Avoids race conditions
		○ Can also be useful for processes that have file descriptors referring to the same open file description
	• Better to use 1 system call than 2 for performance
		○ I/O bears much greater overhead than system calls though

# 5 7 Scatter Gather IO with readv and writev

	• Readv()/writev() = read multiple buffers at once. Buffers are defined in an array iov. Iovcnt specifies number of buffers (array element count).
		○ Ssize_t readv(int fd, const struct iovec *iov, int iovcnt);
			§ Returns # bytes read, 0 on EOF, -1 on error
		○ Ssize_t writev(int fd, const struct iovec *iov, int iovcnt);
			§ Returns # bytes written, -1 on error
	• Each element of iov concists of:
```c
struct iovec {
			§ void *iov_base // start address of buffer
			§ size_t iov_len // # of bytes to transer to/from buffer
}
```
