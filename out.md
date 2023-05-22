The Linux Programming Interface

> 5.1

-   Syscalls are atomic, uninterrupted.

    -   Avoids race conditions.

-   Open() call with O\_EXCL and O\_CREAT. The following is one ATOMIC
    operation:

    -   Checks if a file exists

        -   If file exists

            -   Returns error

        -   If file does not exist

            -   Creates it

-   Another example: appending data to file

    -   Lseek() to find EOF

    -   Write() to write data

    -   If a process is interrupted after lseek() and before write(),
        and another process does the same thing, data will be
        overwritten.

    -   Solution: open() with O\_APPEND flag.

>  
>
> 5.3

-   Retrieve/Modify status flags of an open file.

-   Checking access mode flags is more complex.

    -   Mask flags value with O\_ACCMODE constant.

-   Modify open file status flags with fcntl() F\_SETFL cmd.

    -   Used in the following scenarios:

        -   File was not opened by the calling program

        -   The file descriptor stems from a pipe() or socket() call,
            not open().

    -   To update flags:

        -   Get flags with GET\_FL

        -   Modify flags

        -   Update flags with SET\_FL

>  
>
> 5.4

-   Can have multiple file descriptors for the same file (e.g. different
    processes)

-   3 data structures maintained by kernel:

    -   Per-process file descriptor table

        -   List of open FD's for each process. Table entry consists of:

            -   a reference to the open FD

            -   A set of flags

    -   System-wide table of all open file descriptors

        -   Table entry consists of:

            -   Current file offset

            -   Status flags

            -   File access mode (read, write, read-write etc)

            -   Settings related to signals driven I/O

            -   A reference to the i-node object for this file

    -   File system i-node table

        -   Consists of:

            -   File type (regular file, socket, FIFO) and permissions

            -   Pointer to a list of locks held on the file

            -   File properties (size, timestamps of different file
                operations etc)

>  
>
>  
>
> 5.5

-   2&gt;&1 tells BASH we want standard error (fd2) redirected to same
    place as standard output (fd1).

    -   ./myscript &gt; results.log 2&gt;&1

        -   Sends std output and std error to the file results.log

-   Dup(), dup2(), dup3()

    -   Duplicate file descriptors (with same offset etc)

>  
>
>  
>
> 5.6

-   Ssize\_t Pread(int fd, void \*buf, size\_t count, off\_t offset);

    -   Returns \# bytes read, 0 on EOF, -1 on error

    -   Like read() but at a specified offset

-   Ssize\_t pwrite(int fd, const void \*buf, size\_t count, off\_t
    offset);

-   Useful in multithreaded applications

    -   Threads in a process all use the same file descriptor table

    -   Avoids race conditions

    -   Can also be useful for processes that have file descriptors
        referring to the same open file description

-   Better to use 1 system call than 2 for performance

    -   I/O bears much greater overhead than system calls though

>  
>
>  
>
> 5.7

-   Readv()/writev() = read multiple buffers at once. Buffers are
    defined in an array iov. Iovcnt specifies number of buffers (array
    element count).

>  

Thursday, 20 April 2023

11:15

 

**<span class="underline">CHAPTER 5: FILE I/O: FURTHER DETAILS</span>**

 

**5.1 Atomicity and Race Conditions**

-   Syscalls = atomic, uninterrupted

    -   Avoids race conditions

-   O\_EXCL and O\_CREAT flags with open() call

    -   Returns error if file already exists, otherwise creates it
        (checks if it exists & otherwise creates: does this in one
        atomic operation)

    -   Process can ensure it is the creator of a file

        -   We can understand why by analyzing bad\_exclusive\_open
            below

-   Fileio/bad\_exclusive\_open.c

    -   What this code does:

        -   Tries to open a file

            -   If file does not exist, file is opened.

            -   If file does exist, file is created.

                -   Prints PID of the process creating the file, inside
                    the loops.

    -   Bad\_exclusive\_open might evaluate the if-statement and notice
        the file does not exist, but then the scheduler might let
        another program run in the meantime. This other program could
        then create the file. Then the scheduler could go back to
        bad\_exclusive\_open. Now the bad\_exclusive\_open runs the
        open() command to create the file if it does not exist, thinking
        it created it.

    -   The above is a race condition example.

    -   The above could also happen in a system with multiple parallel
        processors (as is common today), not necessarily a kernel
        process scheduler thing.

    -   Testing the issue: Make one program sleep for 5 seconds, run two
        instances of the program simultaneously; both will claim they
        exclusively created the file.

 

-   Another example of atomicity being required:

    -   Multiple processes appending to global log file.

    -   First lseek() to end of file, then write() to end of file.

    -   If another process runs between lseek() and write() and appends
        some data to the global log file, when the first process
        resumes, it will overwrite that data because lseek() put them at
        the same end-of-file position (which changes while the second
        process ran and appended data).

    -   Problem solved with: open() with O\_APPEND flag.

<img src="media/image1.png" style="width:3.74167in;height:3.675in" />

 

**5.2. File control operations fcntl()**

 

Int fcntl(int fd, int cmd, …);

 

**5.3 Open file status flags**

 

-   Retrieve/Modify status flags of an open file.

    -   Int flags = fcntl(fd, F\_GETFL);

        -   F\_GETFL gets flags.

    -   We can check if file was opened with synchronized writes, for
        instance:

        -   If (flags & O\_SYNC)

            -   Printf("synced for writes");

-   Checking access mode flags is more complex

    -   Because O\_RDONLY, O\_WRONLY, O\_RDWR consist of multiple (i.e.
        not single) bits in the open file status flags

    -   Mask flags value with O\_ACCMODE constant:

        -   Int accessmode = flags & O\_ACCMODE

        -   If (accessmode == O\_WRONLY || accessmode == O\_RDWR)

            -   Printf("file is writable");

-   Modify open file status flags with fcntl() F\_SETFL cmd.

    -   Used in the following scenarios:

        -   File was not opened by the calling program

        -   The file descriptor stems from a pipe() or socket() call,
            not open().

    -   To update flags:

        -   Get flags with GET\_FL

        -   Modify flags

        -   Update flags with SET\_FL

    -   Can modify O\_APPEND, O\_NONBLOCK, O\_NOATIME, O\_ASYNC,
        O\_DIRECT.

    -   See the code on the right.

 

**5.4 relationship between file descriptors and open files**

 

-   Can have multiple file descriptors for the same file (e.g. different
    processes)

-   3 data structures maintained by kernel:

    -   Per-process file descriptor table

        -   List of open FD's for each process. Table entry consists of:

            -   a reference to the open FD

            -   A set of flags

    -   System-wide table of all open file descriptors

        -   Table entry consists of:

            -   Current file offset

            -   Status flags

            -   File access mode (read, write, read-write etc)

            -   Settings related to signals driven I/O

            -   A reference to the i-node object for this file

    -   File system i-node table

        -   Consists of:

            -   File type (regular file, socket, FIFO) and permissions

            -   Pointer to a list of locks held on the file

            -   File properties (size, timestamps of different file
                operations etc)

-   Reasons for multiple fd's for same files:

    -   Fork() could lead to process A and B pointing to same file
        descriptor (fd 2 process A, fd 2 proc B)

    -   Fd 0 in pA and fd3 in pB =&gt; but they point to same i-node
        (e.g. they each called open() on the same file)

    -   Implications:

        -   Because file offset is system-wide, if pA for fd2 changes
            file offset, pB's fd2 is also impacted.

        -   Similar impact on open file status flags (e.g. O\_APPEND,
            O\_ASYNC etc) using fcntl() F\_GETFL and F\_SETFL commands.

        -   File descriptor flags are private to the process (e.g.
            close-on-exec)

 

**5.5 Duplicating file descriptors**

 

-   2&gt;&1 tells BASH we want standard error (fd2) redirected to same
    place as standard output (fd1).

    -   ./myscript &gt; results.log 2&gt;&1

        -   Sends std output and std error to the file results.log

        -   The shell sets fd2 to point to the same open file as fd1 by
            duplicating fd1.

        -   Opening the results.log file twice is not sufficient: file
            offsets etc would not be synced i.e. stdoutput and stderror
            would overwrite each other.

        -   Dup() takes an oldfd, returns new fd that points to the same
            open fd.

        -   Dup2(int oldfd, int newfd);

            -   To specify exactly which newfd number you want, e.g.:
                dup2(1, 2);

                -   This returns the value of the 2nd file descriptor

                -   Returns with error EBADF

            -   Can also use fcntl(oldfd, F\_DUPED, startfd);

                -   startFD denotes the starting number of your desired
                    fd number, if not available, a higher number is
                    returned

        -   Dup can always be replicated using close() and fcntl()

        -   Dup3(int oldfd, int newfd, int flags) for optional flags
            such as O\_CLOEXEC (close-on-exec flag)

>  
>
>  

**5.6 file i/o at a specified offset: pread() and pwrite()**

 

-   Ssize\_t Pread(int fd, void \*buf, size\_t count, off\_t offset);

    -   Returns \# bytes read, 0 on EOF, -1 on error

    -   Like read() but at a specified offset

    -   Equivalent to atomically calling the following:

        -   Orig = lseek() //save current offset

        -   Lseek(offset) // set pointer to specified offset

        -   S = read(…) // read data

        -   Lseek(orig) // set pointer to original offset

-   Ssize\_t pwrite(int fd, const void \*buf, size\_t count, off\_t
    offset);

    -   Returns \# bytes written, -1 on error

-   Useful in multithreaded applications

    -   Threads in a process all use the same file descriptor table

    -   Avoids race conditions

    -   Can also be useful for processes that have file descriptors
        referring to the same open file description

-   Better to use 1 system call than 2 for performance

    -   I/O bears much greater overhead than system calls though

 

**5.7 Scatter-Gather I/O: readv() and writev()**

 

-   Readv()/writev() = read multiple buffers at once. Buffers are
    defined in an array iov. Iovcnt specifies number of buffers (array
    element count).

    -   Ssize\_t readv(int fd, const struct iovec \*iov, int iovcnt);

        -   Returns \# bytes read, 0 on EOF, -1 on error

    -   Ssize\_t writev(int fd, const struct iovec \*iov, int iovcnt);

        -   Returns \# bytes written, -1 on error

-   Each element of iov concists of:

    -   Struct iovec {

        -   Void \*iov\_base // start address of buffer

        -   Size\_t iov\_len // \# of bytes to transer to/from buffer

    -   }

>  

**SCATTER INPUT**

-   Readv() reads from an FD and stores buffers per iovec = "scattering"

-   Atomic - data is copied from user memory to kernel in whole, so
    tampering during operation cannot occur.

-   t\_readv.c

 

**GATHER OUTPUT**

 

Int flags = fcntl(fd, F\_GETFL);

If (flags == -1) errExit();

Flags |= O\_APPEND;

If fcntl(fd, SET\_FL, flags) == -1

errExit();

Printf("success");

 

<img src="media/image2.png" style="width:4.15833in;height:3.175in" />

 

<img src="media/image3.png" style="width:3.58333in;height:1.65833in" />
