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
