---
layout: post
title: Chapter 2 Booting and System Management Daemon
categories: sysadmin
---

![Image](/docs/assets/images/sysadmin-handbook/ch2/1.png)

![Image](/docs/assets/images/sysadmin-handbook/ch2/2.png)

![Image](/docs/assets/images/sysadmin-handbook/ch2/3.png)

![Image](/docs/assets/images/sysadmin-handbook/ch2/4.png)

![Image](/docs/assets/images/sysadmin-handbook/ch2/5.png)

# 2.6 SYS MANAGEMENT DAEMONS

	• Kernel gets loaded into memory
		○ As part of the Kernel's implementation, "spontaneous" processes also get run
			§ These have low PIDs
		○ Systemd  has PID 1 - the initialization process for the kernel.
	• Init responsibilities:
		○ Init can run in 3 different modes:
			1. Single-user mode. Minimal setup. Basic filesystem and root shell.
			2. Multi-user mode. Full setup. Several users, GUI, network services, etc.
			3. Server mode. Also just shell, but loads services relevant for servers.
		○ Init does the following:
			§ Cleans up old files in /tmp.
			§ Configures packet filters, network services, etc.
			§ Starts/stops processes.
			§ Setting time zone, computer name.
			§ Mounting file systems, checking disks.
	• Implementations of Init:
		○ FreeBSD uses init
		○ Linux distros use systemd
		○ MacOS uses launchd
	• Traditional init:
		○ Has some shortcomings:
			§ Init is not powerful enough to run things on its own.
			§ It has a configuration file that points to shell scripts for selecting run level (i.e. which mode to run in; single-user, multi-user or server mode)
			§ These shell scripts in turn point to additional scripts for starting the relevant services and daemones for a given run level/mode.
	• Systemd vs the world
		○ Systemd is a much larger "monolithic" component than init.
		○ Systemd has control over things like the network daemons/services networkd, journald, and logind.
			§ Arguments against systemd include that it's bad for security that systemd has control over so many areas of the system.
			§ There are many more arguments at systemd.org. E.g. that it breeds complexity.
			§ It imposed many new standards and guidelines on the Linux kernel, which could also be a reason people dislike it.
	• Inits judged and assigned their proper punishments
		○ Systemd has mostly taken over, Red Hat, Ubuntu, Debian
		○ Init can still be useful for small footprint Linux installations

# 2.7 SYSTEMD IN DETAIL
	• It does advanced process management
	• It's a collection of many things; daemons, libraries, kernel components, etc.

	• Units and unit files
		○ Systemd manages "UNITS"
		○ A unit can be a: service, socket, mount point, disk, resource management slice, and many other things.
		○ A UNIT FILE tells systemd where the entity that systemd manages (the unit) has its executable file, how systemd can start/stop its services and which services it depends on.
		○ /etc/rsync.conf - unit file for rsync daemon which mirrors filesystems.
		○ Unit files typically go in /usr/lib/systemd/system/, but do not modify these.
		○ Your modified unit files can go into /etc/systemd/system/
		○ /etc unit files have highest prio.
		○ File suffixes: .service or .timer. Some declarations in the files can only appear for certain types of units, e.g. SERVICE for services etc.

	• Systemctl: manage systemd
		○ Systemctl --list-units --type=service
		○ Systemctl --list-unit-files [pattern]
		○ Various ways of working with systemd on the cmdline.

	• Unit statuses
		○ Sudo systemctl status <service-name>
		○ Use the -l option to get full log outputs and not condensed ones which are mostly unreadable.
		○ "linked" status
			§ The unit file is not part of systemd's system directory. Instead, it's a symlink to the unit file which lives elsewhere.
			§ Disabling a linked unit file deletes the symlink and all references to it.
		○ "masked" status
			§ Systemctl is aware of the unit but is not allowed to run it or act on any of its configuration.
	• Targets
		○ You can set which run-level/operating mode you want to run in.
		○ Sudo systemctl set-default graphical.target
			§ This will run a full GUI with networking and all other user features
			§ You can also choose single-user mode or recovery mode, etc.
		○ You can also run sudo systemctl isolate recovery.target
			§ Isolate deactivates all components not needed for recovery and only activates those required for recovery
		○ Sudo systemctl list-units --type=target
			§ See a list of all available targets/operating modes/run-levels
	• Dependencies among units
		○ D-Bus interprocess communication/network ports
			§ Systemd knows which services would use various network ports and IPC connection points
			§ IF a client shows up at that port/point, only then would systemd start the corresponding service.
				□ Otherwise, the service remains dormant.
		○ In the unit files, you can specify which other services a given unit might depend on, might "Wants" i.e. prefer to run in conjunction with, etc. See the list on the right.
		○ Systemctl add-wants multi-user.target my.local.service
			§ This ensures that when multi-user.target boots, my.local.service will be started as well.
			§ This adds a DEPENDENCY for multi-user.target on my.local.service.
	• Execution Order
		○ The execution order of services is handled by a separate systemd component and is not necessarily determined by the various dependency clauses such as "Wants", "Requires" etc.
	• A more complex unit file example
		○ Type forking means the service should run as a daemon in the background
		○ SIGQUIT is equivalent to:
			§ ExecStop=/bin/kill -s HUP $MAINPID
	• Local services and customizations
		○ Go to /usr/lib/systemd/system/
			§ Lists user-space services' unit files.
			§ Quite easy to setup a home-made service's unit file here.
		○ If you wish to modify existing configuration:
			§ Mkdir /usr/lib/systemd/system/<service-name>.d = e.g. nginx.d
			§ Inside nginx.d, you can put a file called overload.conf where you can add your new configuration.
				□ Systemd meshes the actual unit file with the .conf file. The .conf file takes priority if e.g. same values are defined.
			§ You can also do this with systemctl edit nginx.service. (then you still must run systemctl restart nginx.service for changes to take effect).
			§ To set the [INSTALL] section of the unit file, just use systemctl add-wants, systemctl add-requires.
	• Service and startup control caveats
        * Red Hat /etc/sysconfig
            * iptables-config, selinux, console/ directory for keymappings, and more.
            * network-scripts/ifcfg-eth0 contains configuration information for the eth0 network interface.
            * iptables-config is only for loading additional iptables modules; not to change iptables config. That's done elsewhere.
    * Systemd logging
        * With the advent of the cloud, diagnostic information was easily lost. Cannot just stare at the terminal to see error output!
        * journald daemon collects system messages from early boot time to final shutdown.
        * You can see the logs with journalctl.
        * rsyslog can forward logs to a remote syslog server or turn them into more ordinary log file formats.
        * /etc/systemd/journald.conf => here you can set configuration options 
            * For example, you might set Storage=persistent, which will save logs across reboots/sessions.

# 2.8 FreeBSD Init and startup scripts

* Startup process for FreeBSD:
    * init => /etc/rc (wrapper which triggers other startup scripts...) => config files: /etc/defaults/config, /etc/rc.conf, /etc/rc.conf.local => /usr/local/etc/rc.d and /etc/rc.d.
    * Startup scripts are run in an order that can be seen with rcorder command.

# 2.9 Reboot and shutdown procedures

* halt and reboot commands
    * Technically the same; reboot prepares AND shuts the system down; halt prepares but does not actually shut it down (unless you do halt -p).
    * shutdown does not differ at all technically from reboot and halt.

# 2.10 Stratagems for a non booting system

* 3 things to do:
    * Don't debug; restore to a previous good state.
    * Get a recovery mode shell, and investigate interactively.
    * Load a separate (live) OS image and debug the primary one from there.

* Single-user mode
    * Recovery mode, minimal startup, no network, just basic initialization.
    * Red Hat tries to mount file systems as well. This might be problematic if the file system is corrupt.
        * Control this with kernel parameters; systemd.unit=emergency.target (from GRUB) will boot without mounting file systems.
* Enter single-user mode on systems that use systemd/GRUB:
    * append: systemd.unit=rescue.target to the end of the kernel parameters (press e on the GRUB screen for the kernel you wish to boot).

* Recovery for cloud systems
    * Reset to known good state!
    * You can, if necessary, launch for example a separate VM/EC2 instance, unmount the disk from the problem VM/EC2 and attach it to the new VM/EC2. From the new VM/EC2, which runs another primary, good state OS, you can mount the problem file system and try to fix it from there.

