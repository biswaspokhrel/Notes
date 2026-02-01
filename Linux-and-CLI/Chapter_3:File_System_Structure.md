# Chapter 3: File System Structure

## 🎯 Learning Objectives
By the end of this chapter, you will:
- Understand the Linux directory hierarchy and its purpose
- Navigate important system directories confidently
- Recognize different file types in Linux
- Interpret file permissions from `ls -l` output
- Understand inodes and file metadata
- Work with hidden files and special attributes

---

## 3.1 Linux Directory Hierarchy

### The Root Directory (/)

Everything in Linux starts from the root directory `/`. Unlike Windows with multiple drive letters (C:, D:), Linux has a single unified directory tree.

```bash
$ cd /
$ ls
bin   dev  home  lib    media  opt   root  sbin  sys  usr
boot  etc  init  lib64  mnt    proc  run   srv   tmp  var
```

**Key Concept**: In Linux, "/" means:
1. Root directory (when alone)
2. Path separator (in paths like `/home/user`)

---

### Important Directories Explained

#### /bin - Essential User Binaries
**Contains**: Essential command binaries (programs) needed for system boot and repair.

```bash
$ ls /bin
bash  cat  cp  ls  mkdir  mv  rm  sh  ...
```

**Examples**:
- `bash` - The shell itself
- `ls`, `cp`, `mv` - Basic file operations
- `cat`, `grep` - Text utilities

**Use Case**: Commands here work even in single-user/rescue mode.

---

#### /sbin - System Binaries
**Contains**: System administration binaries (root user tools).

```bash
$ ls /sbin
fdisk  fsck  halt  ifconfig  init  reboot  shutdown  ...
```

**Difference from /bin**:
- `/bin`: For all users (ls, cat, cp)
- `/sbin`: For system admin (fdisk, reboot)

---

#### /etc - Configuration Files
**Contains**: System-wide configuration files (plain text, no binaries).

```bash
$ ls /etc
passwd  shadow  hosts  hostname  fstab  ssh/  nginx/  ...
```

**Important Files**:
```bash
# User accounts
$ cat /etc/passwd  
root:x:0:0:root:/root:/bin/bash
john:x:1000:1000:John Doe:/home/john:/bin/bash

# System hostname
$ cat /etc/hostname
ubuntu-desktop

# Host file (local DNS)
$ cat /etc/hosts
127.0.0.1   localhost
127.0.1.1   ubuntu-desktop

# File system mounts
$ cat /etc/fstab
UUID=xxx  /  ext4  defaults  0  1
```

**Use Case**: Backup /etc before system changes:
```bash
$ sudo cp -r /etc /backup/etc_$(date +%Y%m%d)
```

---

#### /home - User Home Directories
**Contains**: Personal directories for each user.

```bash
$ ls -l /home
drwxr-xr-x 25 john  john  4096 Jan 30 10:00 john
drwxr-xr-x 18 jane  jane  4096 Jan 29 15:30 jane
drwxr-xr-x 12 bob   bob   4096 Jan 28 12:00 bob
```

**Your Home**: `~` or `$HOME` or `/home/username`

```bash
$ echo $HOME
/home/john

$ cd ~
$ pwd
/home/john

# Tilde expansion for other users
$ ls ~jane
# Lists Jane's home directory
```

**Typical Home Structure**:
```
/home/john/
├── Desktop
├── Documents
├── Downloads
├── Music
├── Pictures
├── Videos
├── .bashrc          (hidden config)
├── .bash_history    (command history)
└── .config/         (application configs)
```

---

#### /root - Root User's Home
**Contains**: Home directory for the root (superuser) account.

```bash
$ sudo ls /root
# Separate from /home
# Only accessible to root
```

**Why Separate?**
- Security: Regular users can't access
- Availability: Exists even if /home is unmounted
- Protection: Root's configs stay intact

---

#### /var - Variable Data
**Contains**: Files that change frequently (logs, caches, spools).

```bash
$ ls /var
log  cache  tmp  spool  mail  www  lib  ...
```

**Important Subdirectories**:

**/var/log - Log Files**:
```bash
$ ls /var/log
syslog  auth.log  kern.log  apache2/  nginx/  ...

# View system log
$ sudo tail -20 /var/log/syslog

# Check authentication attempts
$ sudo grep "Failed password" /var/log/auth.log
```

**/var/www - Web Server Files**:
```bash
$ ls /var/www
html/  mysite/  ...

# Default web root for Apache/Nginx
```

**/var/cache - Application Cache**:
```bash
$ ls /var/cache
apt/  fontconfig/  man/  ...
```

**Use Case - Cleaning Space**:
```bash
# Check log sizes
$ sudo du -sh /var/log/*

# Clear old logs (carefully!)
$ sudo journalctl --vacuum-time=7d
```

---

#### /tmp - Temporary Files
**Contains**: Temporary files, cleaned on reboot.

```bash
$ ls /tmp
systemd-private-xxx  tmpfiles-xxx  ...
```

**Characteristics**:
- World-writable (anyone can create files)
- Cleaned automatically (usually on reboot)
- Often mounted as tmpfs (RAM disk)
- Sticky bit set (users can't delete others' files)

**Usage**:
```bash
# Safe place for temporary work
$ cd /tmp
$ mkdir my_temp_work
$ # Do work here
$ # Files gone after reboot
```

---

#### /usr - User Programs
**Contains**: User applications and utilities (not "user data").

```bash
$ ls /usr
bin  sbin  lib  lib64  share  local  include  src  ...
```

**Subdirectories**:
- `/usr/bin`: User commands (gcc, python, vim)
- `/usr/sbin`: Non-essential system binaries
- `/usr/lib`: Libraries for /usr/bin
- `/usr/local`: Locally installed software
- `/usr/share`: Architecture-independent data (docs, icons)

**Historical Note**: Originally "UNIX System Resources"

---

#### /opt - Optional Software
**Contains**: Add-on application software packages.

```bash
$ ls /opt
google/  teams/  slack/  ...
```

**Purpose**: Self-contained 3rd party applications
**Example**: Google Chrome installs to `/opt/google/chrome/`

---

#### /mnt and /media - Mount Points
**/mnt**: Temporary mount point for administrators
```bash
$ sudo mount /dev/sdb1 /mnt
$ ls /mnt
```

**/media**: Automatic mount point for removable media
```bash
$ ls /media/$USER
USB_DRIVE  EXTERNAL_HD  ...
```

---

#### /dev - Device Files
**Contains**: Device files (everything is a file in Linux).

```bash
$ ls /dev
sda  sda1  sda2  tty  null  zero  random  ...
```

**Common Devices**:
```bash
/dev/sda        # First SATA drive
/dev/sda1       # First partition on sda
/dev/null       # Black hole (discard output)
/dev/zero       # Infinite zeros
/dev/random     # Random data
/dev/tty        # Current terminal
```

**Examples**:
```bash
# Discard output
$ command > /dev/null

# Generate random password
$ head -c 16 /dev/urandom | base64

# Create 1GB file of zeros
$ dd if=/dev/zero of=testfile bs=1M count=1024
```

---

#### /proc - Process Information
**Contains**: Virtual filesystem with process and system information.

```bash
$ ls /proc
1/  2/  1234/  cpuinfo  meminfo  version  ...
```

**Not Real Files**: Data generated on-the-fly by kernel.

**Examples**:
```bash
# CPU information
$ cat /proc/cpuinfo

# Memory information
$ cat /proc/meminfo

# Current processes
$ ls /proc
# Each number is a PID

# Info about process 1234
$ cat /proc/1234/cmdline
$ cat /proc/1234/status

# Kernel version
$ cat /proc/version
```

---

#### /sys - System Information
**Contains**: Information about devices, drivers, and kernel features.

```bash
$ ls /sys
block  bus  class  dev  devices  firmware  kernel  module  power  ...
```

**Examples**:
```bash
# List block devices
$ ls /sys/block

# Battery information (laptops)
$ cat /sys/class/power_supply/BAT0/capacity

# CPU frequency
$ cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq
```

---

### Directory Hierarchy Visualization

```
/                       (root - top of filesystem)
├── bin/               (essential binaries)
├── boot/              (bootloader files)
├── dev/               (device files)
├── etc/               (configuration files)
│   ├── passwd         (user accounts)
│   ├── shadow         (passwords)
│   └── hostname       (system name)
├── home/              (user home directories)
│   ├── john/
│   └── jane/
├── lib/               (system libraries)
├── media/             (removable media mount points)
├── mnt/               (temporary mount points)
├── opt/               (optional software)
├── proc/              (process information - virtual)
├── root/              (root user's home)
├── run/               (runtime data)
├── sbin/              (system binaries)
├── srv/               (service data)
├── sys/               (system information - virtual)
├── tmp/               (temporary files)
├── usr/               (user programs)
│   ├── bin/
│   ├── lib/
│   ├── local/
│   └── share/
└── var/               (variable data)
    ├── log/           (log files)
    ├── cache/         (cached data)
    └── www/           (web files)
```

---

## 3.2 File Types and Permissions

### File Types in Linux

Linux has seven file types:

| Symbol | Type | Description | Example |
|--------|------|-------------|---------|
| `-` | Regular file | Normal files (text, binary, data) | `file.txt` |
| `d` | Directory | Folder/directory | `Documents/` |
| `l` | Symbolic link | Shortcut to another file | `link -> target` |
| `c` | Character device | Serial devices (keyboard, mouse) | `/dev/tty` |
| `b` | Block device | Storage devices | `/dev/sda` |
| `p` | Named pipe (FIFO) | Inter-process communication | |
| `s` | Socket | Network communication | `/var/run/docker.sock` |

**Checking File Types**:
```bash
$ ls -l
-rw-r--r-- 1 john john    512 Jan 30 10:00 file.txt
drwxr-xr-x 2 john john   4096 Jan 30 10:00 folder
lrwxrwxrwx 1 john john      8 Jan 30 10:00 link -> file.txt
brw-rw---- 1 root disk  8, 0 Jan 30 09:00 /dev/sda
crw--w---- 1 root tty   4, 0 Jan 30 09:00 /dev/tty0
```

---

### Understanding ls -l Output

```bash
$ ls -l file.txt
-rw-r--r-- 1 john developers 1024 Jan 30 10:00 file.txt
```

Breaking it down:
```
-rw-r--r--  1  john  developers  1024  Jan 30 10:00  file.txt
│││││││││  │   │     │            │     │            └─ filename
│││││││││  │   │     │            │     └── modification time
│││││││││  │   │     │            └──────── size (bytes)
│││││││││  │   │     └───────────────────── group
│││││││││  │   └─────────────────────────── owner
│││││││││  └──────────────────────────────── hard links count
││││││││└───────────────────────────────── execute (others)
│││││││└────────────────────────────────── write (others)
││││││└─────────────────────────────────── read (others)
│││││└──────────────────────────────────── execute (group)
││││└───────────────────────────────────── write (group)
│││└────────────────────────────────────── read (group)
││└─────────────────────────────────────── execute (owner)
│└──────────────────────────────────────── write (owner)
└───────────────────────────────────────── read (owner) / file type
```

---

### File Permissions Basics

**Three Permission Types**:
1. **r (read)** - View file contents / list directory contents
2. **w (write)** - Modify file / create/delete files in directory  
3. **x (execute)** - Run file as program / enter directory

**Three Permission Groups**:
1. **Owner (user)** - File creator
2. **Group** - Users in the file's group
3. **Others** - Everyone else

**Symbolic Notation**:
```
rwxr-xr--
│││││││││
│││││││└└─ others: read only
│││└└└──── group: read + execute
└└└─────── owner: read + write + execute
```

**Numeric Notation**:
```
r = 4
w = 2
x = 1

rwx = 4+2+1 = 7
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4

rwxr-xr-- = 754
```

---

### Permission Examples

```bash
-rw-r--r--  (644)  # Regular file: owner can read/write, others read only
-rwxr-xr-x  (755)  # Executable: owner can do all, others can read/execute
drwxr-xr-x  (755)  # Directory: standard directory permissions
-rw-------  (600)  # Private file: only owner can read/write
drwx------  (700)  # Private directory: only owner can access
-rwxrwxrwx  (777)  # Dangerous: everyone can do everything
```

**Common Permission Patterns**:
```bash
# Files
644  rw-r--r--  # Documents (owner writes, others read)
600  rw-------  # Private files (SSH keys, passwords)
755  rwxr-xr-x  # Scripts/executables

# Directories
755  rwxr-xr-x  # Standard directories
700  rwx------  # Private directories
775  rwxrwxr-x  # Shared group directories
```

---

## 3.3 File Attributes

### Hidden Files (Dot Files)

Files starting with `.` are hidden.

```bash
# Normal ls doesn't show hidden files
$ ls
Documents  Pictures  Videos

# -a shows all files including hidden
$ ls -a
.  ..  .bashrc  .bash_history  .config  Documents  Pictures  Videos

# -A shows all except . and ..
$ ls -A
.bashrc  .bash_history  .config  Documents  Pictures  Videos
```

**Common Hidden Files**:
```bash
~/.bashrc           # Bash configuration
~/.bash_history     # Command history
~/.ssh/             # SSH keys and config
~/.config/          # Application configurations
~/.local/           # User-installed applications
~/.cache/           # Application caches
~/.gitconfig        # Git configuration
```

**Create Hidden File**:
```bash
$ touch .my_hidden_file
$ ls          # Won't show it
$ ls -a       # Will show it
```

---

### File Extensions

**Key Difference from Windows**:
- Windows: Extensions determine file type (.exe, .txt)
- Linux: Extensions are just part of the name

```bash
# All these are treated the same by the system
script
script.sh
script.txt

# File type determined by content/permissions, not name
$ file script
script: Bourne-Again shell script, ASCII text executable

# The shebang (#!/bin/bash) determines interpreter
```

**Common Extensions (Convention Only)**:
```bash
.txt    # Text file
.sh     # Shell script
.py     # Python script
.c      # C source code
.tar    # Tar archive
.gz     # Gzip compressed
.jpg    # JPEG image
.pdf    # PDF document
```

---

### Inodes and File Metadata

**Inode**: Data structure storing file metadata (not the name or data).

**What's in an Inode**:
- File type
- Permissions
- Owner and group
- File size
- Timestamps (access, modification, change)
- Link count
- Pointers to data blocks

**View Inode Number**:
```bash
$ ls -i
262234 file.txt

$ stat file.txt
  File: file.txt
  Size: 1024            Blocks: 8          IO Block: 4096   regular file
Device: 801h/2049d      Inode: 262234      Links: 1
Access: (0644/-rw-r--r--)  Uid: ( 1000/   john)   Gid: ( 1000/   john)
Access: 2026-01-30 10:00:00.000000000 +0545
Modify: 2026-01-30 10:00:00.000000000 +0545
Change: 2026-01-30 10:00:00.000000000 +0545
 Birth: -
```

**Hard Links Share Inodes**:
```bash
$ echo "Hello" > file1.txt
$ ln file1.txt file2.txt    # Hard link
$ ls -li
262234 -rw-r--r-- 2 john john 6 Jan 30 10:00 file1.txt
262234 -rw-r--r-- 2 john john 6 Jan 30 10:00 file2.txt
       └─ Same inode number!
```

---

### Timestamps

Linux tracks three timestamps:

```bash
$ stat file.txt
Access: 2026-01-30 10:00:00  # Last read
Modify: 2026-01-30 09:30:00  # Last content change
Change: 2026-01-30 09:30:00  # Last metadata change
```

1. **atime (Access Time)**: Last read
2. **mtime (Modification Time)**: Last content change
3. **ctime (Change Time)**: Last metadata change

```bash
# Find files modified in last 7 days
$ find . -mtime -7

# Find files accessed today
$ find . -atime 0

# Update access/modification time
$ touch file.txt
```

---

## 💡 Key Concepts Summary

1. Linux uses a **single unified directory tree** starting from `/`
2. **Important directories**: /home (users), /etc (config), /var (logs), /tmp (temporary)
3. **File types**: Regular (-), Directory (d), Link (l), Device (b/c)
4. **Permissions**: Read, Write, Execute for Owner, Group, Others
5. **Hidden files** start with `.` (dot)
6. **Inodes** store file metadata
7. **Extensions don't determine file type** in Linux

---

## 🔥 Real-World Use Cases

### Use Case 1: System Administrator Checking Logs

```bash
# Navigate to log directory
$ cd /var/log

# Check recent system errors
$ sudo tail -50 syslog | grep error

# See disk usage of logs
$ sudo du -sh *

# Find large log files
$ sudo find . -type f -size +100M

# Clean old logs
$ sudo find . -name "*.log" -mtime +30 -delete
```

### Use Case 2: Web Developer Setting Up Site

```bash
# Check web root
$ ls -l /var/www

# Create new site directory
$ sudo mkdir -p /var/www/mysite/public_html

# Set proper permissions
$ sudo chown -R www-data:www-data /var/www/mysite

# Create index file
$ sudo touch /var/www/mysite/public_html/index.html
```

### Use Case 3: Finding Configuration Files

```bash
# Find all SSH configs
$ find /etc -name "*ssh*" 2>/dev/null

# View current hostname
$ cat /etc/hostname

# Check DNS settings
$ cat /etc/resolv.conf

# Backup all configs
$ sudo tar -czf /backup/etc_backup_$(date +%Y%m%d).tar.gz /etc
```

### Use Case 4: Analyzing Disk Usage

```bash
# Check overall disk usage
$ df -h

# Find what's using space in /var
$ sudo du -sh /var/* | sort -rh | head -10

# Find files larger than 100MB
$ sudo find /var -type f -size +100M -exec ls -lh {} \;

# Check inode usage
$ df -i
```

---

## 🧪 Practice Problems

### Beginner Level

**Problem 1**: List all directories directly under the root (/) directory.

**Problem 2**: Display your current home directory using three different methods.

**Problem 3**: View the contents of /etc/hostname and /etc/hosts files.

**Problem 4**: Count how many files are in the /bin directory.

**Problem 5**: Create a hidden file called .practice in your home directory.

### Intermediate Level

**Problem 6**: Find the file type (using `ls -l`) of these locations: /dev/sda, /proc, /bin/bash

**Problem 7**: List all hidden files in your home directory with their permissions.

**Problem 8**: Find the five largest files in /var/log (you may need sudo).

**Problem 9**: Explain the permissions of this file: `-rwxr-x---` in both symbolic and numeric notation.

**Problem 10**: Use stat to view all metadata of a file you create.

### Advanced Level

**Problem 11**: Create two files with a hard link, verify they share the same inode, then delete one and check if the other still exists.

**Problem 12**: Find all files in /etc that have been modified in the last 24 hours.

**Problem 13**: Explain why `/dev/null` is useful and provide three practical examples.

**Problem 14**: Map out the directory structure of /usr and explain the purpose of each subdirectory.

**Problem 15**: Write a command to find all symbolic links in your home directory.

---

## 📝 Solutions

### Beginner Solutions

**Solution 1**:
```bash
$ ls -d /*/
/bin/  /boot/  /dev/  /etc/  /home/  /lib/  /media/  /mnt/  /opt/  /proc/  /root/  /run/  /sbin/  /srv/  /sys/  /tmp/  /usr/  /var/
```

**Solution 2**:
```bash
# Method 1
$ echo $HOME

# Method 2
$ cd ~; pwd

# Method 3
$ cd; pwd
```

**Solution 3**:
```bash
$ cat /etc/hostname
ubuntu-desktop

$ cat /etc/hosts
127.0.0.1   localhost
127.0.1.1   ubuntu-desktop
```

**Solution 4**:
```bash
$ ls /bin | wc -l
# Or
$ ls -1 /bin | wc -l
```

**Solution 5**:
```bash
$ touch ~/.practice
$ ls -a ~ | grep practice
.practice
```

### Intermediate Solutions

**Solution 6**:
```bash
$ ls -l /dev/sda /proc /bin/bash
brw-rw---- 1 root disk 8, 0 Jan 30 09:00 /dev/sda    # Block device
dr-xr-xr-x 1 root root     0 Jan 30 09:00 /proc      # Directory
-rwxr-xr-x 1 root root 1.2M Jan 15 10:00 /bin/bash  # Executable file
```

**Solution 7**:
```bash
$ ls -ald ~/.*
drwxr-xr-x 25 john john 4096 Jan 30 10:00 .
drwxr-xr-x  5 root root 4096 Jan 20 09:00 ..
-rw-------  1 john john 8234 Jan 30 09:30 .bash_history
-rw-r--r--  1 john john  220 Jan 10 10:00 .bash_logout
-rw-r--r--  1 john john 3771 Jan 10 10:00 .bashrc
drwx------ 12 john john 4096 Jan 29 15:00 .cache
drwx------ 18 john john 4096 Jan 30 08:00 .config
```

**Solution 8**:
```bash
$ sudo du -ah /var/log | sort -rh | head -5
1.2G    /var/log
450M    /var/log/journal
235M    /var/log/syslog
120M    /var/log/auth.log
85M     /var/log/kern.log
```

**Solution 9**:
```bash
-rwxr-x---

Symbolic: 
Owner: rwx (read, write, execute)
Group: r-x (read, execute)
Others: --- (no permissions)

Numeric:
Owner: 4+2+1 = 7
Group: 4+0+1 = 5
Others: 0+0+0 = 0
Result: 750
```

**Solution 10**:
```bash
$ touch testfile.txt
$ stat testfile.txt
  File: testfile.txt
  Size: 0               Blocks: 0          IO Block: 4096   regular empty file
Device: 801h/2049d      Inode: 262345      Links: 1
Access: (0664/-rw-rw-r--)  Uid: ( 1000/   john)   Gid: ( 1000/   john)
Access: 2026-01-30 10:30:00.000000000 +0545
Modify: 2026-01-30 10:30:00.000000000 +0545
Change: 2026-01-30 10:30:00.000000000 +0545
```

### Advanced Solutions

**Solution 11**:
```bash
# Create file
$ echo "Test content" > file1.txt

# Create hard link
$ ln file1.txt file2.txt

# Verify same inode
$ ls -li file*.txt
262345 -rw-r--r-- 2 john john 13 Jan 30 10:30 file1.txt
262345 -rw-r--r-- 2 john john 13 Jan 30 10:30 file2.txt

# Delete one file
$ rm file1.txt

# Check if other still exists and has content
$ cat file2.txt
Test content

# Explanation: Hard links share the same data. 
# Deleting one link doesn't delete the data until all links are removed.
```

**Solution 12**:
```bash
$ sudo find /etc -type f -mtime 0

# Or for exactly last 24 hours:
$ sudo find /etc -type f -mmin -1440
```

**Solution 13**:
```bash
# /dev/null is a special file that discards all data written to it
# and returns EOF when read from

# Example 1: Discard output
$ command > /dev/null 2>&1
# Sends both stdout and stderr to /dev/null (silence command)

# Example 2: Empty a file
$ cat /dev/null > logfile.txt
# or
$ > logfile.txt

# Example 3: Test command without seeing output
$ grep "pattern" large_file.txt > /dev/null
$ echo $?
0  # Success - pattern found, but output was discarded
```

**Solution 14**:
```bash
$ tree -L 1 /usr
/usr
├── bin/        # User commands and applications
├── include/    # Header files for C programming
├── lib/        # Libraries for /usr/bin programs
├── lib64/      # 64-bit libraries
├── libexec/    # Programs started by other programs
├── local/      # Locally compiled/installed software
│   ├── bin/
│   ├── lib/
│   └── share/
├── sbin/       # Non-essential system binaries
├── share/      # Architecture-independent data
│   ├── doc/    # Documentation
│   ├── man/    # Manual pages
│   └── icons/  # Icons and themes
└── src/        # Source code (if installed)
```

**Solution 15**:
```bash
# Method 1: Using find
$ find ~ -type l

# Method 2: With details
$ find ~ -type l -ls

# Method 3: Using ls
$ ls -la ~ | grep ^l
```

---

## 🎓 Learning Tips

### Mnemonics for Directories
- **/bin** = **Bin**aries (programs)
- **/etc** = **Et C**etera (configs - though actually "Editable Text Configuration")
- **/var** = **Var**iable (changes frequently)
- **/tmp** = **T**e**mp**orary
- **/usr** = **U**nix **S**ystem **R**esources
- **/opt** = **Opt**ional

### Memory Aid for Permissions
```
User  Group  Others
rwx   rwx    rwx
421   421    421
```

**Remember**: **4-2-1** = **R-W-X**

### Best Practices
1. **Never modify files in /proc or /sys** - they're virtual
2. **Backup /etc** before system changes
3. **Check /var/log** when troubleshooting
4. **Use /tmp** for temporary work
5. **Don't store data in /root** - use /home
6. **Respect permissions** - they exist for security

### Common Mistakes to Avoid
1. Confusing `/` (root) with `/root` (root's home)
2. Assuming file extensions matter
3. Forgetting `-a` flag when looking for hidden files
4. Not using `sudo` when accessing system directories
5. Modifying critical files in /etc without backups

---

## 🔗 Connections to Future Chapters

- **Chapter 4**: Detailed permissions management (chmod, chown)
- **Chapter 5**: Processing files in /var/log
- **Chapter 6**: Redirecting to /dev/null
- **Chapter 7**: Understanding /proc for processes
- **Chapter 13**: Monitoring /var for system resources
- **Chapter 14**: Managing services that use /etc configs

---

## 📚 Additional Resources

### Man Pages to Read
```bash
man hier        # File system hierarchy
man file        # Determine file type
man stat        # File status
```

### Further Reading
- File System Hierarchy Standard: https://refspecs.linuxfoundation.org/fhs.shtml
- Linux Documentation Project: https://www.tldp.org/

### Practice Commands
```bash
# Explore the file system
$ cd /
$ ls -laR | less

# Understand your system
$ df -h           # Disk space
$ du -sh /*       # Size of top-level directories
$ mount           # Mounted filesystems
$ findmnt         # Better mount display
```

---

## ✅ Self-Assessment Checklist

Before moving to Chapter 4, ensure you can:
- [ ] Explain the purpose of /, /home, /etc, /var, /tmp, /usr
- [ ] Navigate to any system directory confidently
- [ ] Identify file types from `ls -l` output
- [ ] Interpret file permissions (both symbolic and numeric)
- [ ] Find and work with hidden files
- [ ] Understand what inodes are
- [ ] Explain why Linux doesn't rely on file extensions
- [ ] Use stat command to view file metadata
- [ ] Describe the difference between /dev, /proc, and /sys
- [ ] Know where to find logs, configs, and user data

---

## 🚀 Next Steps

Great job! You now understand the Linux filesystem structure and file basics.

In **Chapter 4**, you'll learn:
- How to change file permissions with chmod
- Managing file ownership with chown and chgrp
- Special permissions (SUID, SGID, sticky bit)
- Using sudo safely
- Access Control Lists (ACLs)

**Challenge**: Create a "cheat sheet" document mapping common tasks to the directories where they happen (e.g., "check logs" → /var/log, "edit config" → /etc).

---

*"Understanding the filesystem is understanding the system. Everything else builds on this foundation."*