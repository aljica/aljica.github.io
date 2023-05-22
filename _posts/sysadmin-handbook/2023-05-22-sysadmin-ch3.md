---
layout: post
title: Chapter 3 Access Control and Rootly Powers
categories: sysadmin
---

### 3.1 Standard UNIX access controls

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

# Filesystem access control

* Owner can set file permissions.
    * Even such that no one else can access the file.
* Groups can also own a file; any user part of that group then also owns the file.
    * Group info in /etc/group.
    * Nowadays, groups are commonly stored in networked DB's like LDAP.

```bash
ls -l
```
* Yields the permissions on a file.
* The file owner can determine the group's privileges on it.
* UIDs are mapped to /etc/passwd file, GIDs to /etc/group.

# Process Ownership

* The owner of a process can reduce the process's scheduling priority, and can send it signals.
* Processes have real, effective, saved and file system ID's associated with them.
    * Effective ID is used for access control. 
        * File system ID's are too, but they are more of an artifact of Linux systems and are not really used.
    * Saved ID's can be used to change between privileged modes of operation, enhancing security.

# The root account

* UID 0.
* Remember that root cannot do *everything*:
    * If a file lacks the executable bit, then root cannot execute it!
    * But, root can perform any VALID privileged operations.
    * This includes:
        * Setting system hostname
        * Setting time of day
        * Configuring network interfaces
        * System shutdown
        * Creating device files
        * and more

# Setuid and setgid execution

* A process can run a setuid/setgid program that is privileged, even though the process lacks that level of privilege.
* The process's privilege level is temporarily raised, just for running that particular program.
* Example might be for a user to change their password: they need to modify the /etc/passwd or /etc/shadow file.
    * So there needs to be a setuid passwd command/program that users can call, which temporarily elevates the privileges of the terminal process the user used to call passwd and set the new password, before returning to its original privilege level.
        * Note that the USER's privilege level does NOT change.
* Be very careful with setuid/setgid programs. Preferably do not use them unless absolutely necessary.
* Specify nosuid option when using the mount command to disable setuid.

### 3.2 Management of the root account

# Root account login

* Root logins are not monitored/audited/recorded/logged!!
    * You won't know what changes were made to your system.
    * You won't know who did what changes, if multiple users can access the root account.
    * Attacker can cover their tracks more easily (and do a ton of damage of course...).
* Disable root logins, ideally.

# su: substitute user identity

* su records who became root and when; but does not record activity.
* For clarity: run su in a terminal and you will have root access to the system via that terminal process instance.
* You can also use `su - username`
    * This prompts you for the password for the user 'username', and then you get access to that account.

<details>
  <summary>Spoiler Placeholder</summary>
  
  Placeholder
</details>
