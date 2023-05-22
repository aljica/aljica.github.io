---
layout: post
title: Chapter 3 Access Control and Rootly Powers
categories: sysadmin
---

# 3.1 Standard UNIX access controls

* Users/Objects
    * Users own objects, but do not have unrestricted privileges over them.
    * Objects may be files and processes.
    * Root is the ultimate owner of all objects on the system.
    * System calls like settimeofday can only be called by root.
    * Others, like "kill", have specific calculations they perform to check permissions depending on user group, etc.
    * File systems implement their own access controls, but do make use of the kernel's VFS layer (and they use UNIX groups for the AC).

* Communication with devices occurs via files found under /dev.
* These files are objects subject to the file system's access controls.
    * Thus, access to devices is restricted by file system AC.
