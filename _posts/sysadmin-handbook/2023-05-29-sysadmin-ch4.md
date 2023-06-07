---
layout: post
title: Chapter 4 Process Control
categories: sysadmin
---

### 4.1 Components of a Process

* A process has a virtual address space, which maps into pages in physical memory.
    * A page is a sub-section of physical memory
        * Physical memory is equally divided into pages
    * These pages are randomly distributed in physical memory.
    * A process consists of code, libraries, variables, stack, heap, etc.
* Kernel internal data structures maintains:
    * A table that maps virtual address space to physical memory pages
    * CPU, memory usage information
    * Status
    * Etc

* All processes have one main thread
* They can have multiple threads, which can run simultaneously on multicore computer systems
* Threads of one process share the same memory

# PID: Process ID Number

* Each process gets an ID.
* Namespaces can further segregate/isolate processes from each other.
    * With namespaces, a process can have different PIDs depending on which other process is checking for it.

# PPID: Parent PID

* Parent & Child processes.
* If the parent dies, init/systemd becomes the new parent.
* Misbehaving applications can be traced back to their parents; this can be helpful.

# UID and EUID: real/effective user ID

* Process has UID value; this is the UID of the user who created the parent process.
* Effective UID: the files/programs a process has access to.
* UID/EUID different only for SETUID programs.
* Saved UID: A SETUID program can place its root privileges here for most of execution, and only change the EUID for the SID for when it needs root.

# GID and EGID: real and effective group ID

* Same concept as for UID and EUID, but for groups.
* GID is really only relevant when a process creates new files; depending on file system permission settings, new files might set the GID of its creator as owner.

# Niceness

* A process gets scheduled based on 2 main factors, derived from an algorithm:
    1. How much CPU time it has used, and
    2. how long it has waited to execute.

# Control terminal

* For nondaemon processes, a control terminal determines that process's STDIN/STDOUT/STDERR channels, sends the process keyboard events (CTRL+C) etc.
* When you use your terminal to run a command/process, your terminal acts as a control terminal.

### 4.2 The Lifecycle of a Process

* fork() system call copies a parent process's memory, code, variables, stack, heap, etc, into a child process.
    * clone() is actually used behind the scenes of fork() - clone() includes threads and additional features.
* fork() returns 2 different values - the child process gets the value 0, the parent gets the PID of the newly created child.
    * This way, in the code, you can have an if statement that checks if(fork() == 0) - then you know you are executing inside the child process. 
    * If fork() equals PID of the child, then you are executing in the parent process.
* The child can use an exec() command to run another command.
    * Exec() also resets various memory segments so that the newly executing command becomes its own process entirely; the fork() command helps it setup the core things needed for a process - e.g. code, variables, stack, heap, etc. by simply copying them from the parent - but then it can change those and run as an entirely independent process.


* Upon system startup, kernel creates the init / systemd process.
    * All subsequent processes are children of init / systemd with PID 1.
* A completed process calls _exit routine to notify kernel it's ready to die.
    * Process sends exit code.
* Parent uses wait() to acknowledge death of child.
    * Parent can examine exit code & other stats.
    * If parent dies first, orphans are adopted by init / systemd which handles wait() and removal of children.

# Signals

* Can be sent among processes as a means of communication
* CTRL+C / CTRL+Z
* kill command used by administrator
* Used by kernel to:
    * Notify a process of e.g. death of child
    * Kill processes that commit infractions (e.g. division by zero).
    * Much more.

* A process can have a signal handler function that gets called when a particular signal is received.
* Otherwise, kernel handles it.
* Results can be:
    * Termination of process.
    * Core dump (memory image of process is logged) - debugging.

* Processes can block / ignore signals.
    * Blocked:
        * Process must explicitly unblock the signal; it is queued by the kernel.
            * Handler function called only once, even if multiple signals in queue.
        * Ignored: signals are dropped.

![Image](/docs/assets/images/sysadmin-handbook/ch4/signals.jpeg)

* KILL and STOP cannot be caught/blocked/ignored.
* STOP suspends process execution until CONT is received.
    * CONT may be ignored/caught, but not blocked.

* CTRL+Z => soft "STOP" => "request to stop".
    * Programs must not have a handler function for this - i.e. they can ignore it.

* Differentiation between different signals:
    * KILL - 
        * Unblockable.
        * Terminates process.
    * INT - 
        * CTRL+C
        * Request to terminate current operation.
            * Process may catch & react to signal. 
            * Default behavior if not handler routine is for process to be killed.