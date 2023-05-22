---
layout: post
title: The Linux Programming Interface Chapter 5 File I/O Further Details
categories: tlpi
---

[5.1 Atomicity and Race Conditions](#5-1-atomicity-and-race-conditions)

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