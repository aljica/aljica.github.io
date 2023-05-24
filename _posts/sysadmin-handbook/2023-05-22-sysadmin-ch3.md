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

* Security: Make a habit of calling the full path: /bin/su or /usr/bin/su.
* Also, don't include the current directory "." as a search path for your terminal.
    * Reason: Attacks can keep fake versions of commonly used commands such as su which could collect passwords.

# sudo: limited su

* sudo takes a command line as argument.
* It checks /etc/sudoers to see if the user requesting the privileged operation is allowed to use sudo.
* Re-typing password after one sudo usage is configurable; time-out period of always require password entry.
* sudo logs who used it, when, etc. Can be logged by syslog.
    * Ideally forward from syslog to a central log server.

#### Example Configuration

![Image](/docs/assets/images/sysadmin-handbook/ch3/sudoers-config.png)

* First two lines define groups of hosts
* The permissions section:
    * First are the user(s) the permissions apply to
    * Then the machine hosts
    * The commands the users can run
    * The users as whom the commands can be executed

* First permission line:
    * Applies to mark and ed
    * On the machines in the PHYSICS group
    * Built-in command ALL means they can run ANY command

* Second line:
    * herb can run tcpdump on CS group machines, and DUMP-related commands on PHYSICS machines.
    * The DUMP-commands can only be run as operator, so the actual command herb would run would be:
    * `ubuntu$ sudo -u operator /usr/sbin/dump 0u /dev/sda1`

* Third line:
    * Lynda can run commands on any machine as any user, except shell.
    * She can still get a shell, though:
    `ubuntu$ cp -r /bin/sh /tmp/sh`
    `ubuntu$ sudo /tmp/sh`
    * Technically speaking, whenever you say "everything except...", there's usually a way around it.
        * But, you can still set your sudoers file as such to denote that root shells are discouraged.

* Fourth and final line:
    * Users in group wheel can run:
        * watchdog command as root on all machines except PHYSICS machines.
        * No password required to run the command.


* Commands in sudoers file contain full path names, to prevent execution of user-created files with the same name.
* Edit sudoers file with visudo.
    * It checks that no one is already editing the file.
    * Verifies syntax of the edited file.
        * Otherwise, you may not be able to sudo back into it if the syntax is off and breaks something.

#### Sudo pros and cons

* Advantages
    * Limit user privileges
    * Logging
    * Faster than su/root login
    * Protect root password
    * List of users with root privileges
    * Revoke privileges
    * Unattended root shell is a non-issue

* Disadvantages
    * Breach of user account = breach of sudo/root
        * Encourage good security/password practices
    * Logging can be subverted
        * At least you'll see an attacker did this

#### Sudo vs advanced access control

* Sudo gives you more control over who has privileges when
* Simple configuration
* One configuration file across all devices on your network
* Advanced logging
* Sudo is portable across all UNIX/LINUX systems


* Negative: Attack surface expansion (includes all user accounts with sudo)

* Typical setup can be simple and straightforward:

```
User_Alias  ADMINS = alice, bob, charles
ADMINS      ALL = (ALL) ALL
```

#### Environment management

* Environment variables can modify behavior of commands
    * Both convenient but also source of attacks
* E.g. EDITOR environment variable to spawn VS Code
    * If this environment variable points to a hacker's program instead; you will execute that.
* That's why sudo environment variables are kept in sudoers file:

```
Defaults    env_keep += "SSH_AUTH_SOCK"
```

* The above line preserves environment variable used by SSH key forwarding.
* You can also preserve a variable as such:

`$ sudo EDITOR=emacs vipw`

#### sudo without passwords

* Avoid:

```
ansible     ALL = (ALL) NOPASSWD : ALL # Don't
```

* Sudo execution without password.
* Common mistake when setting up Ansible for configuration automation etc.

* Replace manual password entry with forwarded SSH keys for authentication
* Can do this through ssh-agent
* Requires PAM module on the server: pam_ssh_agent_package

#### Precedence

```
User_Alias      ADMINS = alice, bob, charles
User_Alias      MYSQL_ADMINS = alice, bob

%wheel          ALL = (ALL) ALL
MYSQL_ADMINS    ALL = (mysql) NOPASSWD : ALL
ADMINS          ALL = (ALL) NOPASSWD : /usr/sbin/logrotate
```

* The LAST matching line takes precedence.
* A matching line must match in the 4-tuple of (user, host, target host, command) for a given user.
    * Otherwise, it is ignored.
* If the %wheel line was last, alice would have to type a password for all commands.

#### sudo without a control terminal

* Allow/Disallow cron/background processes from running privileged/sudo commands
* In sudoers file, add: `Defaults   requiretty`
* Generally, disallow by adding '!' in front of requiretty. Best practice.

#### Site-wide sudo configuration

* Use one sudoers file and distribute it across hosts all over your network.
    * = Scalability!
* This is because you can keep distinct user and group information in the sudoers file.
    * Think machine host names, user names, etc.
* Avoid %wheel, i.e. group-based definitions in your sudoers config file.
    * Group membership is LOCAL and does not scale.
    * You'd have to check the groups on that host to see who is affected by that line.


* Configuration management system.
    * Great for distributing updated sudoers file to hosts across your network!
* If no config mgmt system:
    1. `scp` on host to fetch sudoers file from host. (Checks remote server's key to ensure integrity/no spoofing).
    2. Validate with `visudo -c -f newsudoersfile` on the local host.


* Hostname of the host must match exactly in the sudoers file.
* This might be a problem if:
    * You've specified hostname `anchor` in sudoers file, but the machines have hosts based on FQDN's like `anchor.cs.colorado.edu` (i.e. when you run the `hostname` command on the host, that's the output you get).
    * Common issue in cloud where hostnames are randomly generated.
    * Sudo has pattern-matching and can normalize based on FQDN's. Turn the fqdn option on in the sudoers file.

# Disabling the root account


