# linux interview questions 1

Basic

### What is Linux?

Linus Torvalds developed Linux, a Unix-like, free, open-source kernel-based operating system. It is designed for systems, servers, embedded devices, mobile devices, and mainframes and is supported on major platforms such as ARM, x86, and SPARC.

### Explain the basic features of the Linux OS

Some basic features of Linux are:

* Linux is free and easily available.
* It is more secure than other operating systems because it uses security auditing and password authentication features.
* Linux has its personal software repositories.
* It includes support for multiple languages and different language keyboards.
* It offers both CLI and GUI to run commands and applications (e.g., Firefox, VLC).

### Name some Linux distros

Commonly used distributions:

* Ubuntu
* Debian
* CentOS
* Fedora
* RedHat

### Major differences between Linux and Windows

| Comparison Factor |                  Linux |                        Windows |
| ----------------- | ---------------------: | -----------------------------: |
| Free/Paid         |  Free and open-source. |  Not open-source (commercial). |
| Security          |         Highly secure. | Less secure compared to Linux. |
| Path separator    | Uses forward slash (/) |              Uses backslash () |
| Efficiency        |         More efficient |                 Less efficient |
| Kernel type       |      Monolithic kernel |                    Microkernel |
| File system       |         Case-sensitive |               Case-insensitive |

### Define the basic components of Linux

Major components:

* Kernel: Core part bridging hardware and software.
* Shell: Interface between kernel and user.
* GUI: Graphical user interface.
* Application programs: Software that performs tasks.
* System utilities: Tools to manage the system.

### File permissions in Linux

Three basic file permissions:

* Read: Open and read files.
* Write: Modify files.
* Execute: Run the file.

### What is the Linux Kernel? Is it legal to edit it?

The kernel is low-level system software that manages resources and provides system interfaces. Linux is released under the GPL (General Public License), so it is legal to edit the kernel in accordance with the license.

### Explain LILO

LILO (Linux Loader) is a boot loader that loads the Linux OS into memory and starts execution. It is one of several boot loaders available for Linux.

### What is Shell in Linux?

Common shells:

* csh (C Shell) — C-like syntax, job control.
* ksh (Korn Shell) — advanced scripting features.
* zsh (Z Shell) — advanced features like globbing, startup files.
* bash (Bourne Again Shell) — default shell on many systems.
* fish (Friendly Interactive Shell) — autosuggestions, web-based config.

### What is a root account?

The root account is the system administrator account with complete system control. Ordinary users do not have root privileges.

### Describe CLI and GUI in Linux

* CLI (Command Line Interface): Accepts commands as text input.
* GUI (Graphical User Interface): Uses icons, menus, and windows manipulated with a mouse.

### What is Swap Space?

Swap space is disk space used to extend RAM. The system moves inactive pages from RAM to swap to free physical memory.

### Difference between hard links and soft links

| Hard Links                                                | Soft Links                                                                 |
| --------------------------------------------------------- | -------------------------------------------------------------------------- |
| Include the original content (direct reference to inode). | Include the original file location (path reference).                       |
| Faster.                                                   | Slower.                                                                    |
| Share the same inode number.                              | Have different inode numbers.                                              |
| No relative path concept (they reference the same inode). | Use relative or absolute paths.                                            |
| Cannot link directories (typically).                      | Can link directories.                                                      |
| Changes reflect across hard links.                        | Changes reflect between soft link and target; breaking target breaks link. |
| Use less disk metadata.                                   | Use more metadata.                                                         |

### How do users create a symbolic link in Linux?

Use the ln command with -s for symbolic links:

```bash
ln -s <target> <link_name>
```

### What are the standard streams?

Standard streams are:

* stdin (standard input)
* stdout (standard output)
* stderr (standard error)

They channel I/O between programs and their environment.

## Intermediate-Level Linux Interview Questions

### How do you mount and unmount filesystems in Linux?

Use `mount` and `umount`.

Mounting steps:

* Identify partition: `fdisk -l` or `lsblk`
* Create mount point: `mkdir /mnt/mountpnt`
* Mount: `sudo mount <partition> <mount_point_directory>`

Unmount:

```bash
sudo umount <mount_point_directory>
```

### How do you troubleshoot network connectivity issues in Linux?

Steps include:

* Check physical connectivity (cables, interface up).
* Verify network configuration: `ip addr` or `ifconfig`, `ip route`, check `/etc/resolv.conf`.
* Check firewall rules (`ufw`, `iptables`).
* Restart network interfaces (`ifdown`/`ifup`) and reboot if needed.
* Use tools like `ping`, `traceroute`, `nslookup`/`dig` to further diagnose.

### How do you list all the processes running in Linux?

* `ps` (e.g., `ps -ef`, `ps auxf`) for snapshots.
* `top` and `htop` for real-time interactive views (`htop` has color, sorting, filtering).

### What is the chmod command and how do you use it?

`chmod` changes file permissions.

Example (add write and execute for user):

```bash
chmod u+wx ABC.sh
```

Supports symbolic modes and numeric modes.

### How do you check disk space usage?

* `df` (e.g., `df -h`) to show filesystem disk space.
* `du` (e.g., `du -sh ~/directory`) to estimate directory usage.
* `ncdu` for interactive disk usage browsing.

### How do you find the PID of a running process?

* `pgrep <process_name>`
* `ps -e | grep -i <process_name>`

### What is rsync and how do you use it?

`rsync` synchronizes files between directories or systems.

Basic syntax:

```bash
rsync <options> <source> <destination>
```

Example:

```bash
rsync -av ~/Documents ~/Downloads
rsync -avz --delete ~/Documents/ ~/Downloads/
```

Options:

* `-a` preserve attributes
* `-v` verbose
* `-z` compress
* `--delete` remove destination files not in source

### How do you create a user account?

Use `useradd` or `adduser`.

Example with `useradd` (set password separately):

```bash
sudo useradd -m ron
sudo passwd ron
```

Example with `adduser` (interactive):

```bash
sudo adduser shawn
```

### How do you format a disk in Linux?

* List partitions: `lsblk`
* Unmount if mounted: `umount <partition>`
* Identify desired filesystem type (ext4, xfs, ntfs) and run one of:

```bash
mkfs.ext4 <partition>
mkfs.xfs <partition>
mkfs.ntfs <partition>
```

Then mount the partition again. Always backup data before formatting.

### How do you change the password for a user account?

```bash
passwd <username>
# e.g.
passwd Ron
```

You'll be prompted to enter and confirm the new password.

### Difference between a process and a thread

| Factor           | Process                       | Thread                   |
| ---------------- | ----------------------------- | ------------------------ |
| Creation time    | Higher                        | Lower                    |
| Dependency       | Independent (separate memory) | Dependent (share memory) |
| Resource usage   | Higher                        | Lower                    |
| Termination time | Higher                        | Lower                    |

### What is the ulimit command and how do you use it?

`ulimit` controls resource limits for user processes.

Example: set max number of processes to 50

```bash
ulimit -u 50
```

### What is the find command and how do you use it?

Searches for files based on criteria.

Example:

```bash
find ~/Downloads -name Linux.txt
```

### What is RAID in Linux?

RAID (Redundant Array of Independent Disks) combines physical disks into logical units for performance and redundancy.

Common levels:

* RAID 0 — striping (no redundancy)
* RAID 1 — mirroring (full copy)
* RAID 5 — distributed parity
* RAID 6 — dual parity (higher redundancy)
* RAID 10 — combination of RAID 1 and RAID 0

### What are the challenges of using Linux?

Common challenges:

* Hardware compatibility issues for some devices.
* Steeper learning curve for configuration and commands.
* Limited game compatibility compared to Windows.
* Occasional driver/firmware issues.

## Advanced-Level Linux Interview Questions

### What is the /proc filesystem?

`/proc` is a virtual filesystem exposing kernel and process information. It can be used to inspect system state and tune kernel parameters at runtime.

### How do you secure a Linux server?

Common practices:

* Use strong passwords.
* Keep system updated and patched.
* Use SSH with key-based authentication.
* Deploy intrusion detection systems (IDS).
* Configure firewalls and restrict services.
* Disable unused services.
* Regular backups and log audits.
* Encrypt network traffic and enable monitoring.

### What is strace?

`strace` traces system calls made by a process. Example:

```bash
strace ls
```

It helps debug how a program interacts with the kernel.

### How do you optimize Linux system performance?

Strategies:

* Keep system updated.
* Optimize disk usage and enable caching.
* Manage memory and CPU usage.
* Disable unnecessary services; use lightweight alternatives.
* Monitor resources regularly.
* Tune kernel parameters where appropriate.
* Use monitoring tools (e.g., Performance Co-Pilot).

### How to administer Linux servers?

Key tasks:

* Manage users and permissions.
* Configure system/network for performance and security.
* Implement backups and recovery plans.
* Set up monitoring for resource usage and system health.
* Configure firewall and intrusion detection.
* Document configurations and test recovery procedures.

### What is the Linux virtual memory system?

Virtual memory allows the system to use disk space as an extension of RAM, swapping pages between RAM and disk to handle workloads that exceed physical memory.

### What is process scheduling in Linux?

Process scheduling determines the order and CPU time allocation for processes. Linux uses priority-based, preemptive scheduling with scheduling policies that can change process ordering based on behavior and resources.

### Most important Linux commands

Commonly used commands:

* ls, mkdir, pwd
* top, htop
* grep, cat
* tar, wget
* free, df
* man

### What is iptables and how is it used for filtering?

`iptables` manages Netfilter firewall rules for packet filtering and NAT.

* Show rules: `iptables -L`
* Add rule example:

```bash
iptables -A <chain> <options> -j <target>
```

* Make rules persistent:

```bash
iptables-save > /etc/iptables/rules.v4
```

### How do you troubleshoot a Linux OS that fails to boot?

Steps:

* Check boot messages and logs.
* Inspect GRUB boot options; try older kernel.
* Check hardware (cables, RAM).
* Identify recent changes that might have caused the issue.
* Boot into recovery or live environment to repair.

### What is the init process?

`init` (PID 1) is the first process started at boot, responsible for initializing the system. Modern systems commonly use `systemd` instead of SysV init.

### What is SMTP?

SMTP (Simple Mail Transfer Protocol) is used to transmit email between servers. Models:

* End-to-end (connects different organizations)
* Store-and-forward (used within organizations)

### What is LVM?

LVM (Logical Volume Manager) provides flexible disk management: create, resize, snapshot, and mirror logical volumes for dynamic storage allocation.

### Difference between UDP and TCP

* UDP: connectionless, low overhead, no reliability (used for real-time apps like streaming, DNS).
* TCP: connection-oriented, reliable (retransmission, ordering) used for file transfers, web, email.

### What is /etc/resolv.conf?

Config file for DNS resolver settings (nameserver entries, search domains, resolver options).

### Absolute vs relative paths

* Absolute path: starts at root `/`, e.g., `/home/user/geeksforgeeks.txt`
* Relative path: relative to current working directory, e.g., `documents/file.txt`

### What is grep used for?

`grep` searches for patterns in files or streams.

Example:

```bash
grep "test" file.txt
```

### How to check status of a service/daemon?

With systemd:

```bash
systemctl status apache2
```

### Difference between /etc/passwd and /etc/shadow

* `/etc/passwd`: user account info (usernames, UIDs, home dirs, shells) — world-readable.
* `/etc/shadow`: encrypted passwords and security info — readable only by root.

### How to compress and decompress files?

Create gzipped tarball:

```bash
tar -czvf jayesh.tar.gz files
```

Extract:

```bash
tar -xzvf jayesh.tar.gz
```

### Difference between a process and a daemon

* Process: executing program instance (foreground or background).
* Daemon: background service typically started at boot; runs without interactive user session.

### How to schedule recurring tasks?

Use `crontab -e` and add an entry.

Example — run script daily at 03:30:

```
30 3 * * * /path/to/geeks.sh
```

### What is sed used for?

`sed` performs stream editing, e.g., find-and-replace:

```bash
sed 's/foo/bar/g' file.txt
```

### What are runlevels?

Runlevels define system states (single-user, multi-user, with/without GUI). Commonly: runlevel 3 (multi-user without GUI), runlevel 5 (with GUI). Modern systems use `systemd` targets.

## Bonus Linux Interview Questions

### What is sudo?

`sudo` (superuser do) runs commands with elevated privileges. It typically requires the user's password for authorization.

### What is umask?

`umask` sets default permission restrictions for newly created files and directories.

### How to find and kill a process?

Find PID:

```bash
ps aux | grep <process>
```

Kill by PID:

```bash
kill <PID>
```

Kill by name:

```bash
pkill <process>
```

`pkill` sends SIGTERM by default.

### What is network bonding?

Bonding combines multiple network interfaces into a single logical interface for redundancy and increased throughput.

### What is SELinux?

Security-Enhanced Linux (SELinux) is a mandatory access control framework that provides an additional layer of access control and security policies.

### Purpose of SSH and how to connect securely

SSH (Secure Shell) provides encrypted remote access. Example:

```bash
ssh username@remote_ip
```

Use key-based authentication for stronger security.

### How to check file contents without opening an editor?

Use `cat`, `less`, or `more`.

Example:

```bash
cat geeks.txt
```

### Purpose of the crontab file

Crontab schedules recurring tasks (cron jobs). Edit with `crontab -e` and add timing and command lines.

Example — run `jayesh.sh` daily at 05:00:

```
0 5 * * * /path/to/jayesh.sh
```

### Find and replace text with sed

Example — replace "true" with "False" in a file:

```bash
sed 's/true/False/g' file_name
```

### Purpose of the sudoers file and configuring sudo

The `sudoers` file controls which users can run commands as root. Edit safely with:

```bash
sudo visudo
```

To grant full sudo access to a user:

```
user_name ALL=(ALL) ALL
```

### Change ownership with chown

```bash
chown new_owner:new_group filename
# e.g.
chown jayesh:users file_name
```

### Purpose of ping and testing connectivity

`ping` sends ICMP echo requests to test reachability:

```bash
ping <remote_host_ip>
```

### Recursively copy files and directories with cp

```bash
cp -R source_directory destination_directory
```

### Purpose of netstat and viewing connections

`netstat` shows active connections and listening ports.

Example — list listening TCP/UDP ports:

```bash
netstat -tuln
```

### Set up a static IP via CLI

Location of config varies by distro (e.g., `/etc/network/interfaces`).

Example stanza (for Debian-style `/etc/network/interfaces`):

```
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
```

Save and restart networking service or reboot.

### Copy a file to multiple directories

Methods include using `xargs`, `find`, `tee`, or a shell loop. Example using a loop:

```bash
for d in dir1 dir2 dir3; do
  cp file.txt "$d"/
done
```

## Linux Admin Interview Questions

### How are files organized in Linux?

Hierarchical filesystem rooted at `/`. Files and directories are organized under `/`.

### How can you find the IP address of a Linux system?

Use `ifconfig` or `ip addr show`.

### Distinction between hard link and symbolic link

* Hard link: direct reference to file inode.
* Symbolic link: path reference. Deleting the target breaks a symlink.

### How to check disk space usage?

Use `df` (e.g., `df -h`) to show total/used/available space.

### How to start and stop a service?

With systemd:

```bash
systemctl start <service>
systemctl stop <service>
```

### Common causes of file permission issues

* Incorrect ownership
* Improper permission bits
* Conflicting user/group permissions

### Troubleshoot system can't connect to a remote server

Check connectivity (`ping`), firewall rules, DNS, and relevant logs.

## Linux Troubleshooting Interview Questions

### Steps to fix a network connectivity issue

* Verify physical connections.
* Verify IP configuration and routes.
* Check firewall and DNS.
* Use `ping`, `traceroute`, `tcpdump` for deeper inspection.

### How to check system logs

View logs in `/var/log` with `tail`, `less`, or `journalctl` (systemd):

```bash
tail -f /var/log/syslog
# or
journalctl -xe
```

### Possible reasons for running out of memory

* Memory leaks in applications
* Excessive memory usage by processes
* Insufficient physical memory
* Large dataset processing

### Troubleshoot a slow-performing server

* Inspect `top`/`htop` for CPU/memory usage
* Monitor disk I/O
* Analyze network traffic
* Check application logs for bottlenecks

### Common causes of running out of disk space

* Large or growing log files
* Accumulation of temporary files
* Uncleaned old backups
* Runaway processes generating output

### Identify and terminate high-CPU process

Use `top`/`htop` to find high CPU processes, then `kill <PID>` to terminate.

### Troubleshoot a system that cannot boot

* Check hardware and BIOS/UEFI settings
* Boot into recovery or live environment
* Inspect boot logs and filesystem integrity

## Linux Networking Interview Questions

### What does ifconfig do?

Configures or displays network interfaces (addresses, netmasks). (`ip` is modern replacement.)

### How do you set up a fixed IP address?

Edit network configuration files depending on distribution (e.g., `/etc/network/interfaces` or `/etc/sysconfig/network-scripts/ifcfg-<interface>`), set static address, netmask, gateway, DNS, then restart networking.

### How to configure a DNS server?

Configure BIND or similar — edit `named.conf` and zone files, set forwarders/root hints as required.

### What is a firewall and how do you set it up?

Firewall filters and controls network traffic. Use `iptables`, `nftables`, or higher-level tools (`ufw`, `firewalld`) to define rules and zones.

### How to check network connectivity between two systems?

Use `ping` and `traceroute` to test reachability and path.

### Purpose of the route command

View or modify the IP routing table.

### How to configure a Linux system as a router?

Enable IP forwarding:

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
# or set in /etc/sysctl.conf: net.ipv4.ip_forward=1
```

Then configure interfaces and routing tables appropriately.
