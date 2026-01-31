# Chapter 8: User and Group Management

## 🎯 Learning Objectives
By the end of this chapter, you will:
- Understand Linux user and group system
- Create and manage user accounts
- Work with /etc/passwd, /etc/shadow, and /etc/group
- Manage passwords and account security
- Create and manage groups
- Switch users with su and sudo
- Configure sudo access properly
- Understand user types and UIDs
- Manage user login history

---

## 8.1 Understanding User Accounts

### What is a User Account?

In Linux, every person or service that accesses the system needs a user account. User accounts:
- Identify who is using the system
- Control access to files and resources
- Track actions for security and auditing
- Isolate users from each other

**Three Types of Users:**

1. **Root User (UID 0)**
   - Superuser with unlimited privileges
   - Can do anything on the system
   - Username: `root`

2. **System Users (UID 1-999)**
   - Used by services and daemons
   - Not for human login
   - Examples: www-data, mysql, sshd

3. **Regular Users (UID 1000+)**
   - Human users
   - Limited privileges
   - Your daily login account

```bash
# Check your user info
$ id
uid=1000(john) gid=1000(john) groups=1000(john),27(sudo),1001(developers)

# Check root
$ id root
uid=0(root) gid=0(root) groups=0(root)

# Check system user
$ id www-data
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

---

### The /etc/passwd File

**Location:** `/etc/passwd`
**Purpose:** Stores user account information
**Permissions:** 644 (readable by all, writable by root)

**Format:** One line per user, colon-separated fields
```
username:password:UID:GID:comment:home_directory:shell
```

**Example:**
```bash
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
john:x:1000:1000:John Doe:/home/john:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

**Field Breakdown:**
```
john:x:1000:1000:John Doe:/home/john:/bin/bash
│    │ │    │    │        │          └─ Login shell
│    │ │    │    │        └──────────── Home directory
│    │ │    │    └───────────────────── Comment/Full name
│    │ │    └────────────────────────── Primary Group ID
│    │ └─────────────────────────────── User ID
│    └───────────────────────────────── Password (x = in /etc/shadow)
└────────────────────────────────────── Username
```

**View passwd file:**
```bash
# Full file
$ cat /etc/passwd

# Specific user
$ grep john /etc/passwd
john:x:1000:1000:John Doe:/home/john:/bin/bash

# Extract usernames only
$ cut -d: -f1 /etc/passwd
root
daemon
john
www-data

# Count users
$ wc -l /etc/passwd
42 /etc/passwd
```

---

### The /etc/shadow File

**Location:** `/etc/shadow`
**Purpose:** Stores encrypted passwords
**Permissions:** 640 (readable only by root)

**Why separate?** Security - passwords are encrypted and not world-readable.

**Format:**
```
username:encrypted_password:last_change:min:max:warn:inactive:expire:reserved
```

**Example:**
```bash
$ sudo cat /etc/shadow
root:!:19000:0:99999:7:::
john:$6$xyz...:19350:0:99999:7:::
www-data:*:19000:0:99999:7:::
```

**Field Breakdown:**
```
john:$6$xyz...:19350:0:99999:7:14:19500:
│    │         │     │  │     │  │  │
│    │         │     │  │     │  │  └─ Account expiration (days since 1970)
│    │         │     │  │     │  └──── Inactivity period
│    │         │     │  │     └─────── Warning period
│    │         │     │  └───────────── Maximum password age
│    │         │     └──────────────── Minimum password age
│    │         └────────────────────── Last password change (days since 1970)
│    └──────────────────────────────── Encrypted password
└───────────────────────────────────── Username
```

**Password Field Values:**
- `$6$...` : Encrypted password (SHA-512)
- `*` : Account locked (no password login)
- `!` : Account locked
- Empty : No password required (dangerous!)

**Check password aging:**
```bash
$ sudo chage -l john
Last password change                                    : Jan 15, 2026
Password expires                                        : never
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 0
Maximum number of days between password change          : 99999
Number of days of warning before password expires       : 7
```

---

### The /etc/group File

**Location:** `/etc/group`
**Purpose:** Stores group information
**Permissions:** 644

**Format:**
```
group_name:password:GID:user_list
```

**Example:**
```bash
$ cat /etc/group
root:x:0:
sudo:x:27:john,alice
developers:x:1001:john,bob,alice
www-data:x:33:
```

**Field Breakdown:**
```
developers:x:1001:john,bob,alice
│          │ │    └─ Group members (comma-separated)
│          │ └────── Group ID
│          └──────── Password (rarely used, x = none)
└─────────────────── Group name
```

**View groups:**
```bash
# All groups
$ cat /etc/group

# Your groups
$ groups
john sudo developers

# Another user's groups
$ groups alice
alice sudo developers designers

# Group details
$ getent group developers
developers:x:1001:john,bob,alice
```

---

## 8.2 User Management Commands

### useradd - Add New User

**Basic Syntax:**
```bash
sudo useradd [options] username
```

**Common Options:**
```bash
-m          # Create home directory
-d dir      # Specify home directory
-s shell    # Specify login shell
-g group    # Primary group
-G groups   # Additional groups (comma-separated)
-c comment  # Full name or comment
-u UID      # Specific UID
-e date     # Expiration date
```

**Examples:**
```bash
# Basic user creation
$ sudo useradd john
# Creates user, but NO home directory by default!

# Better: Create with home directory
$ sudo useradd -m alice
# Creates /home/alice

# Full user creation
$ sudo useradd -m -s /bin/bash -c "Bob Smith" -G sudo,developers bob

# Specific UID and home
$ sudo useradd -m -u 1500 -d /home/special/charlie charlie

# With expiration date
$ sudo useradd -m -e 2026-12-31 tempuser

# System user (for services)
$ sudo useradd -r -s /usr/sbin/nologin appuser
```

**Default Settings:**
```bash
# View defaults
$ useradd -D
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/sh
SKEL=/etc/skel
CREATE_MAIL_SPOOL=no

# Modify defaults
$ sudo useradd -D -s /bin/bash
# New users will get bash by default
```

**What happens when you create a user:**
1. Entry added to /etc/passwd
2. Entry added to /etc/shadow
3. Entry added to /etc/group (if creating new group)
4. Home directory created (if -m used)
5. Files from /etc/skel copied to home directory

```bash
# Check /etc/skel
$ ls -la /etc/skel
.bash_logout
.bashrc
.profile
```

---

### usermod - Modify User

**Syntax:**
```bash
sudo usermod [options] username
```

**Common Operations:**
```bash
# Change login shell
$ sudo usermod -s /bin/zsh john

# Change home directory (and move files)
$ sudo usermod -d /home/newlocation -m john

# Change username
$ sudo usermod -l newname oldname

# Lock account
$ sudo usermod -L john
# Password login disabled, other auth methods still work

# Unlock account
$ sudo usermod -U john

# Add to group (append)
$ sudo usermod -aG sudo john
# Without -a, replaces all secondary groups!

# Set expiration date
$ sudo usermod -e 2026-12-31 john

# Change UID
$ sudo usermod -u 1500 john

# Change comment/full name
$ sudo usermod -c "John Smith Jr." john
```

**Important usermod Flags:**
```bash
# WRONG - Removes user from all other groups!
$ sudo usermod -G developers john

# CORRECT - Adds to group while keeping others
$ sudo usermod -aG developers john
# -a = append
# -G = supplementary groups
```

---

### userdel - Delete User

**Syntax:**
```bash
sudo userdel [options] username
```

**Examples:**
```bash
# Delete user (keeps home directory)
$ sudo userdel john

# Delete user AND home directory
$ sudo userdel -r john

# Force delete (even if logged in)
$ sudo userdel -f john
# Dangerous! Can cause issues.
```

**Safe Deletion Procedure:**
```bash
# 1. Check if user is logged in
$ who | grep john

# 2. Check running processes
$ ps -u john

# 3. Kill user's processes if needed
$ sudo pkill -u john

# 4. Lock account first (optional)
$ sudo usermod -L john

# 5. Delete with home directory
$ sudo userdel -r john

# 6. Verify
$ id john
id: 'john': no such user
```

**Finding leftover files:**
```bash
# Find files owned by deleted user (by UID)
$ sudo find / -uid 1000 2>/dev/null

# Fix ownership if needed
$ sudo chown newuser:newgroup /path/to/file
```

---

### passwd - Change Password

**Syntax:**
```bash
passwd [options] [username]
```

**Examples:**
```bash
# Change your own password
$ passwd
Changing password for john.
Current password: 
New password: 
Retype new password: 
passwd: password updated successfully

# Change another user's password (root only)
$ sudo passwd alice
New password: 
Retype new password: 
passwd: password updated successfully

# Lock account
$ sudo passwd -l john
passwd: password expiry information changed.

# Unlock account
$ sudo passwd -u john

# Force password change on next login
$ sudo passwd -e john

# Delete password (make account passwordless - dangerous!)
$ sudo passwd -d john

# Check password status
$ sudo passwd -S john
john P 01/15/2026 0 99999 7 -1
#    │ └─ Last change
#    └─ P=usable password, L=locked, NP=no password
```

**Password Policy:**
```bash
# Set password aging
$ sudo chage john
# Interactive editor

# Or with flags
$ sudo chage -M 90 john       # Max 90 days
$ sudo chage -m 7 john        # Min 7 days
$ sudo chage -W 14 john       # Warn 14 days before
$ sudo chage -I 30 john       # Inactive after 30 days
$ sudo chage -E 2026-12-31 john  # Account expires

# View settings
$ sudo chage -l john
```

---

## 8.3 Group Management

### groupadd - Create Group

```bash
# Create group
$ sudo groupadd developers

# With specific GID
$ sudo groupadd -g 1500 designers

# System group (GID < 1000)
$ sudo groupadd -r appgroup

# Verify
$ getent group developers
developers:x:1001:
```

---

### groupmod - Modify Group

```bash
# Rename group
$ sudo groupmod -n newname oldname

# Change GID
$ sudo groupmod -g 1600 developers
```

---

### groupdel - Delete Group

```bash
# Delete group
$ sudo groupdel developers

# Can't delete if it's a user's primary group
$ sudo groupdel john
groupdel: cannot remove the primary group of user 'john'

# Fix: Change user's primary group first
$ sudo usermod -g users john
$ sudo groupdel john
```

---

### gpasswd - Administer Groups

```bash
# Add user to group
$ sudo gpasswd -a john developers
Adding user john to group developers

# Remove user from group
$ sudo gpasswd -d john developers
Removing user john from group developers

# Set group administrators
$ sudo gpasswd -A alice,bob developers

# Add multiple users
$ sudo gpasswd -M john,alice,bob developers
```

---

### Working with Groups

```bash
# List your groups
$ groups
john sudo developers

# List specific user's groups
$ groups alice
alice sudo developers designers

# Primary vs secondary groups
$ id john
uid=1000(john) gid=1000(john) groups=1000(john),27(sudo),1001(developers)
#                    └─ Primary group    └─ Secondary groups

# Change primary group temporarily
$ newgrp developers
$ id
uid=1000(john) gid=1001(developers) groups=1001(developers),1000(john),27(sudo)
#                    └─ Now primary (for this session)

# Exit temporary group
$ exit
```

**Use Cases:**
```bash
# Create shared project group
$ sudo groupadd project-team
$ sudo usermod -aG project-team john
$ sudo usermod -aG project-team alice
$ sudo usermod -aG project-team bob

# Create shared directory
$ sudo mkdir /shared/project
$ sudo chgrp project-team /shared/project
$ sudo chmod 2775 /shared/project
# 2 = SGID bit - new files inherit group

# Now all team members can collaborate
```

---

## 8.4 Switching Users

### su - Switch User

**Syntax:**
```bash
su [options] [username]
```

**Examples:**
```bash
# Switch to root
$ su
Password: [root password]
# You're now root, but in same directory

# Switch to root with root's environment
$ su -
# or
$ su -l
Password: 
# Now in /root with root's environment

# Switch to another user
$ su john
Password: [john's password]

# Switch with that user's environment
$ su - john

# Run single command as another user
$ su -c "command" username
$ su -c "ls /root" root

# Switch to user without password (root only)
$ sudo su - john
```

**su vs su -:**
```bash
# su (without -)
$ su
# - Keeps current directory
# - Keeps current environment variables
# - Uses current PATH

# su - (with -)
$ su -
# - Changes to target user's home directory
# - Loads target user's environment
# - Uses target user's PATH and settings
```

**Checking who you are:**
```bash
# After su
$ whoami
root

$ who am i
john pts/0   2026-01-31 10:00
# Shows original login user
```

---

### sudo - Execute as Another User

**Syntax:**
```bash
sudo [options] command
```

**Basic Usage:**
```bash
# Run command as root
$ sudo apt update

# Run command as specific user
$ sudo -u alice cat /home/alice/file.txt

# Start shell as root
$ sudo -i
# or
$ sudo -s

# Run with preserved environment
$ sudo -E command

# List sudo privileges
$ sudo -l
User john may run the following commands on ubuntu:
    (ALL : ALL) ALL

# Run last command with sudo
$ apt update
E: Could not open lock file
$ sudo !!
# !! expands to previous command
```

**sudo vs su:**

| Feature | su | sudo |
|---------|-----|------|
| Password | Target user's password | Your own password |
| Logging | No detailed logs | Commands are logged |
| Granularity | All or nothing | Can limit specific commands |
| Timeout | None | Default 15 minutes |
| Best Practice | ✗ Outdated | ✓ Modern approach |

---

### Configuring sudo

**sudoers file:** `/etc/sudoers`

**⚠️ NEVER edit directly!** Always use:
```bash
$ sudo visudo
```

**Why visudo?**
- Checks syntax before saving
- Prevents broken configuration
- Locks file during editing

**sudoers Syntax:**
```
user    host=(run_as) commands
```

**Examples:**
```bash
# User john can run everything as root
john    ALL=(ALL:ALL) ALL

# Alice can run apt commands
alice   ALL=/usr/bin/apt, /usr/bin/apt-get

# Bob can run systemctl without password
bob     ALL=(ALL) NOPASSWD: /bin/systemctl

# Allow sudo group (Ubuntu default)
%sudo   ALL=(ALL:ALL) ALL

# Allow wheel group (RHEL/CentOS default)
%wheel  ALL=(ALL) ALL

# Allow developers to restart apache
%developers ALL=/usr/sbin/systemctl restart apache2

# Complex example
charlie ALL=(www-data) NOPASSWD: /var/www/scripts/*.sh
# Charlie can run scripts in /var/www/scripts/ as www-data without password
```

**Grant sudo Access:**
```bash
# Method 1: Add to sudo group (Debian/Ubuntu)
$ sudo usermod -aG sudo john

# Method 2: Add to wheel group (RHEL/CentOS)
$ sudo usermod -aG wheel john

# Method 3: Add to sudoers file
$ sudo visudo
# Add line:
john ALL=(ALL:ALL) ALL

# Method 4: Drop-in file (cleaner)
$ sudo visudo -f /etc/sudoers.d/john
# Add:
john ALL=(ALL:ALL) ALL
```

**Testing sudo Configuration:**
```bash
# Check what you can do
$ sudo -l

# Test specific command
$ sudo -l /usr/bin/apt

# Validate sudoers syntax
$ sudo visudo -c
/etc/sudoers: parsed OK
```

---

## 8.5 User Information and Login History

### id - Display User Identity

```bash
# Your identity
$ id
uid=1000(john) gid=1000(john) groups=1000(john),27(sudo),1001(developers)

# Specific user
$ id alice

# Just UID
$ id -u
1000

# Just GID
$ id -g
1000

# All groups (GIDs)
$ id -G
1000 27 1001

# All groups (names)
$ id -Gn
john sudo developers
```

---

### who - Who is Logged In

```bash
# Currently logged in users
$ who
john    pts/0   2026-01-31 10:00 (192.168.1.100)
alice   pts/1   2026-01-31 10:15 (192.168.1.101)

# With more details
$ who -a

# Just count
$ who | wc -l
2

# Who am I
$ who am i
john    pts/0   2026-01-31 10:00 (192.168.1.100)
```

---

### w - Who and What

```bash
# Extended who information
$ w
 10:30:15 up 5 days,  2:45,  2 users,  load average: 0.15, 0.20, 0.18
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
john     pts/0    192.168.1.100    10:00    1.00s  0.04s  0.00s w
alice    pts/1    192.168.1.101    10:15    5:00   0.12s  0.10s vim

# Without header
$ w -h

# Specific user
$ w john
```

---

### last - Login History

```bash
# Recent logins
$ last
john     pts/0    192.168.1.100   Fri Jan 31 10:00   still logged in
alice    pts/1    192.168.1.101   Fri Jan 31 10:15   still logged in
john     pts/0    192.168.1.100   Thu Jan 30 15:30 - 18:45  (03:15)

# Last 10 entries
$ last -n 10

# Specific user
$ last john

# Failed login attempts
$ sudo lastb
bob      ssh:notty    192.168.1.200   Fri Jan 31 09:30 - 09:30  (00:00)

# Last reboot
$ last reboot
reboot   system boot  5.4.0-42-generic Mon Jan 27 08:00   still running
```

---

### finger - User Information

```bash
# Install finger (if not present)
$ sudo apt install finger

# User info
$ finger john
Login: john                             Name: John Doe
Directory: /home/john                   Shell: /bin/bash
On since Fri Jan 31 10:00 (NPT) on pts/0 from 192.168.1.100
No mail.
No Plan.
```

---

## 💡 Key Concepts Summary

1. **User Types**: Root (0), System (1-999), Regular (1000+)
2. **Key Files**: /etc/passwd, /etc/shadow, /etc/group
3. **User Management**: useradd, usermod, userdel, passwd
4. **Group Management**: groupadd, groupmod, groupdel, gpasswd
5. **Switch Users**: su (use target password), sudo (use your password)
6. **sudo**: Modern, secure way to run privileged commands
7. **visudo**: Only way to edit /etc/sudoers safely

---

## 🔥 Real-World Use Cases

### Use Case 1: Setting Up New Employee

```bash
# Create user account
$ sudo useradd -m -s /bin/bash -c "Alice Johnson" -G developers,docker alice

# Set password
$ sudo passwd alice
New password: 
Retype new password: 

# Force password change on first login
$ sudo passwd -e alice

# Add to sudo group (if needed)
$ sudo usermod -aG sudo alice

# Set password aging
$ sudo chage -M 90 -m 7 -W 14 alice

# Verify
$ id alice
$ sudo chage -l alice
```

### Use Case 2: Creating Shared Team Directory

```bash
# Create group
$ sudo groupadd project-alpha

# Add team members
$ sudo usermod -aG project-alpha john
$ sudo usermod -aG project-alpha alice
$ sudo usermod -aG project-alpha bob

# Create shared directory
$ sudo mkdir -p /shared/project-alpha

# Set ownership and permissions
$ sudo chgrp project-alpha /shared/project-alpha
$ sudo chmod 2775 /shared/project-alpha
# 2 = SGID, 7 = rwx owner, 7 = rwx group, 5 = rx others

# Verify
$ ls -ld /shared/project-alpha
drwxrwsr-x 2 root project-alpha 4096 Jan 31 10:00 /shared/project-alpha

# Test: Create file as alice
$ sudo -u alice touch /shared/project-alpha/test.txt
$ ls -l /shared/project-alpha/test.txt
-rw-r--r-- 1 alice project-alpha 0 Jan 31 10:00 test.txt
#                   └─ Inherited group!
```

### Use Case 3: Service Account Setup

```bash
# Create system user for application
$ sudo useradd -r -s /usr/sbin/nologin -d /opt/myapp myapp

# Create application directory
$ sudo mkdir -p /opt/myapp

# Set ownership
$ sudo chown -R myapp:myapp /opt/myapp

# Verify
$ id myapp
uid=998(myapp) gid=998(myapp) groups=998(myapp)

$ ls -ld /opt/myapp
drwxr-xr-x 2 myapp myapp 4096 Jan 31 10:00 /opt/myapp
```

### Use Case 4: Temporary Contractor Access

```bash
# Create account with expiration
$ sudo useradd -m -s /bin/bash -e 2026-03-31 contractor

# Set password
$ sudo passwd contractor

# Add to limited group
$ sudo groupadd contractors
$ sudo usermod -aG contractors contractor

# Monitor login history
$ sudo last contractor

# When done, account auto-expires
# Or manually delete
$ sudo userdel -r contractor
```

### Use Case 5: Sudo Configuration for DevOps Team

```bash
# Create devops group
$ sudo groupadd devops

# Add team members
$ sudo usermod -aG devops john
$ sudo usermod -aG devops alice

# Configure sudo access
$ sudo visudo -f /etc/sudoers.d/devops

# Add:
%devops ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx
%devops ALL=(ALL) NOPASSWD: /bin/systemctl restart apache2
%devops ALL=(ALL) /usr/bin/tail -f /var/log/nginx/*
%devops ALL=(ALL) /usr/bin/tail -f /var/log/apache2/*

# Test
$ sudo -u john sudo -l
User john may run the following commands:
    (ALL) NOPASSWD: /bin/systemctl restart nginx
    (ALL) NOPASSWD: /bin/systemctl restart apache2
    (ALL) /usr/bin/tail -f /var/log/nginx/*
    (ALL) /usr/bin/tail -f /var/log/apache2/*
```

---

## 🧪 Practice Problems

### Beginner Level

**Problem 1**: Create a new user called "testuser" with a home directory.

**Problem 2**: Change your own password.

**Problem 3**: List all users on the system by extracting usernames from /etc/passwd.

**Problem 4**: Check which groups you belong to.

**Problem 5**: View the last 5 login attempts on the system.

### Intermediate Level

**Problem 6**: Create a user "developer" with a specific UID of 1500 and add them to the "sudo" group.

**Problem 7**: Create a group called "webteam" and add three users to it.

**Problem 8**: Lock a user account, verify it's locked, then unlock it.

**Problem 9**: Change a user's login shell from /bin/sh to /bin/bash.

**Problem 10**: Set password aging for a user: max 60 days, min 7 days, warning 14 days before expiration.

### Advanced Level

**Problem 11**: Create a complete user management script that creates a user, sets password aging, adds to groups, and creates a personal project directory.

**Problem 12**: Configure sudo so that a user can restart the nginx service without a password, but requires a password for all other sudo commands.

**Problem 13**: Find all files owned by a deleted user (UID 1005) and change ownership to another user.

**Problem 14**: Create a shared directory for the "marketing" team where all files created automatically belong to the marketing group.

**Problem 15**: Generate a report showing all users, their UIDs, home directories, and whether they have sudo access.

---

## 📝 Solutions

### Beginner Solutions

**Solution 1**:
```bash
$ sudo useradd -m testuser

# Verify
$ id testuser
$ ls -ld /home/testuser
drwxr-x--- 2 testuser testuser 4096 Jan 31 10:00 /home/testuser
```

**Solution 2**:
```bash
$ passwd
Changing password for john.
Current password: [enter old password]
New password: [enter new password]
Retype new password: [confirm]
passwd: password updated successfully
```

**Solution 3**:
```bash
$ cut -d: -f1 /etc/passwd
root
daemon
bin
sys
john
alice
testuser
...

# Or count them
$ cut -d: -f1 /etc/passwd | wc -l
42
```

**Solution 4**:
```bash
$ groups
john sudo developers

# Or more detailed
$ id
uid=1000(john) gid=1000(john) groups=1000(john),27(sudo),1001(developers)
```

**Solution 5**:
```bash
$ last -n 5
john     pts/0    192.168.1.100   Fri Jan 31 10:00   still logged in
alice    pts/1    192.168.1.101   Fri Jan 31 09:45   still logged in
john     pts/0    192.168.1.100   Thu Jan 30 15:30 - 18:45  (03:15)
bob      pts/2    192.168.1.102   Thu Jan 30 12:00 - 14:30  (02:30)
alice    pts/1    192.168.1.101   Thu Jan 30 09:00 - 17:00  (08:00)
```

### Intermediate Solutions

**Solution 6**:
```bash
$ sudo useradd -m -u 1500 -G sudo developer

# Verify UID
$ id developer
uid=1500(developer) gid=1500(developer) groups=1500(developer),27(sudo)

# Set password
$ sudo passwd developer
```

**Solution 7**:
```bash
# Create group
$ sudo groupadd webteam

# Create three users
$ sudo useradd -m -G webteam webuser1
$ sudo useradd -m -G webteam webuser2
$ sudo useradd -m -G webteam webuser3

# Verify
$ getent group webteam
webteam:x:1002:webuser1,webuser2,webuser3
```

**Solution 8**:
```bash
# Lock user
$ sudo passwd -l testuser
passwd: password expiry information changed.

# Verify locked
$ sudo passwd -S testuser
testuser L 01/31/2026 0 99999 7 -1
#        └─ L = Locked

# Try to login (will fail)
$ su testuser
Password: 
su: Authentication failure

# Unlock
$ sudo passwd -u testuser
passwd: password expiry information changed.

# Verify unlocked
$ sudo passwd -S testuser
testuser P 01/31/2026 0 99999 7 -1
#        └─ P = Password (usable)
```

**Solution 9**:
```bash
# Check current shell
$ grep testuser /etc/passwd
testuser:x:1001:1001::/home/testuser:/bin/sh

# Change to bash
$ sudo usermod -s /bin/bash testuser

# Verify
$ grep testuser /etc/passwd
testuser:x:1001:1001::/home/testuser:/bin/bash
```

**Solution 10**:
```bash
$ sudo chage -M 60 -m 7 -W 14 testuser

# Verify
$ sudo chage -l testuser
Last password change                                    : Jan 31, 2026
Password expires                                        : Apr 01, 2026
Password inactive                                       : never
Account expires                                         : never
Minimum number of days between password change          : 7
Maximum number of days between password change          : 60
Number of days of warning before password expires       : 14
```

### Advanced Solutions

**Solution 11**:
```bash
#!/bin/bash
# create_user.sh - Comprehensive user creation script

if [ $# -ne 3 ]; then
    echo "Usage: $0 username fullname groups"
    echo "Example: $0 jsmith 'John Smith' 'developers,sudo'"
    exit 1
fi

USERNAME=$1
FULLNAME=$2
GROUPS=$3

# Create user
sudo useradd -m -s /bin/bash -c "$FULLNAME" -G "$GROUPS" "$USERNAME"

# Set password (will prompt)
sudo passwd "$USERNAME"

# Set password aging
sudo chage -M 90 -m 7 -W 14 "$USERNAME"

# Create personal project directory
sudo mkdir -p "/projects/$USERNAME"
sudo chown "$USERNAME:$USERNAME" "/projects/$USERNAME"
sudo chmod 755 "/projects/$USERNAME"

# Display summary
echo ""
echo "User $USERNAME created successfully!"
echo "Home: /home/$USERNAME"
echo "Projects: /projects/$USERNAME"
echo "Groups: $GROUPS"
echo ""
echo "User details:"
id "$USERNAME"
sudo chage -l "$USERNAME"

# Usage:
# $ sudo ./create_user.sh jsmith "John Smith" "developers,sudo"
```

**Solution 12**:
```bash
$ sudo visudo -f /etc/sudoers.d/nginx-restart

# Add:
john ALL=(ALL) NOPASSWD: /bin/systemctl restart nginx
john ALL=(ALL) ALL

# Test
$ sudo -l
User john may run the following commands:
    (ALL) NOPASSWD: /bin/systemctl restart nginx
    (ALL) ALL

# Test without password
$ sudo systemctl restart nginx
# Works without password!

# Test other command
$ sudo apt update
[sudo] password for john: [requires password]
```

**Solution 13**:
```bash
# Find files owned by UID 1005
$ sudo find / -uid 1005 2>/dev/null

# Change ownership to user 'john' (assuming UID 1000)
$ sudo find / -uid 1005 -exec chown 1000:1000 {} \; 2>/dev/null

# Or to a specific user
$ sudo find / -uid 1005 -exec chown john:john {} \; 2>/dev/null

# Verify
$ sudo find / -uid 1005 2>/dev/null
# Should return nothing if all changed
```

**Solution 14**:
```bash
# Create group
$ sudo groupadd marketing

# Add users
$ sudo usermod -aG marketing alice
$ sudo usermod -aG marketing bob
$ sudo usermod -aG marketing charlie

# Create shared directory
$ sudo mkdir -p /shared/marketing

# Set group ownership
$ sudo chgrp marketing /shared/marketing

# Set permissions with SGID
$ sudo chmod 2775 /shared/marketing
# 2 = SGID bit
# 7 = rwx for owner
# 7 = rwx for group
# 5 = r-x for others

# Verify
$ ls -ld /shared/marketing
drwxrwsr-x 2 root marketing 4096 Jan 31 10:00 /shared/marketing
#       ↑ SGID bit (s instead of x)

# Test
$ sudo -u alice touch /shared/marketing/test.txt
$ ls -l /shared/marketing/test.txt
-rw-r--r-- 1 alice marketing 0 Jan 31 10:00 test.txt
#                   └─ Automatically assigned to marketing group!
```

**Solution 15**:
```bash
#!/bin/bash
# user_report.sh

echo "User Report - $(date)"
echo "================================================================"
printf "%-15s %-6s %-25s %-10s\n" "USERNAME" "UID" "HOME" "SUDO"
echo "----------------------------------------------------------------"

# Get all users with UID >= 1000 (regular users)
awk -F: '$3 >= 1000 {print $1, $3, $6}' /etc/passwd | while read user uid home; do
    # Check if user has sudo access
    if groups "$user" | grep -q sudo || groups "$user" | grep -q wheel; then
        sudo_access="YES"
    else
        sudo_access="NO"
    fi
    
    printf "%-15s %-6s %-25s %-10s\n" "$user" "$uid" "$home" "$sudo_access"
done

echo "================================================================"

# Run it:
# $ chmod +x user_report.sh
# $ ./user_report.sh
```

---

## 🎓 Learning Tips

### Username Rules
- Lowercase letters, numbers, dashes, underscores
- Must start with letter or underscore
- Max 32 characters (technically, but keep it short)
- Avoid special characters

### Remember UIDs
- **0** = Root (only one)
- **1-999** = System users (services)
- **1000+** = Regular users (humans)

### Group Management Mnemonic
- **groupadd** = Create group
- **groupmod** = Modify group
- **groupdel** = Delete group
- **gpasswd** = Group password (admin group)

### su vs sudo
```
su   = "Switch User" - become another user
sudo = "Superuser DO" - do something as root
```

**Remember**: sudo is preferred because:
- Uses your password (don't need to know root password)
- Logs what you do
- Temporary elevation
- Can limit which commands

### Common Mistakes to Avoid

1. **Forgetting -m with useradd**
   ```bash
   # WRONG - No home directory
   $ sudo useradd john
   
   # RIGHT
   $ sudo useradd -m john
   ```

2. **Using usermod -G without -a**
   ```bash
   # WRONG - Removes from other groups
   $ sudo usermod -G developers john
   
   # RIGHT - Appends to groups
   $ sudo usermod -aG developers john
   ```

3. **Editing /etc/sudoers directly**
   ```bash
   # WRONG - Can break sudo!
   $ sudo nano /etc/sudoers
   
   # RIGHT - Syntax checking
   $ sudo visudo
   ```

4. **Deleting user without checking**
   ```bash
   # Check first
   $ who | grep john
   $ ps -u john
   
   # Then delete
   $ sudo userdel -r john
   ```

5. **Not setting password after useradd**
   ```bash
   $ sudo useradd -m john
   # Account exists but can't login!
   
   $ sudo passwd john
   # Now can login
   ```

---

## 🔗 Connections to Future Chapters

- **Chapter 4**: File permissions depend on users and groups
- **Chapter 7**: Processes are owned by users
- **Chapter 9**: Package management often requires sudo
- **Chapter 14**: Services run as specific users
- **Chapter 19**: Security policies involve user management

---

## 📚 Additional Resources

### Man Pages
```bash
man useradd
man usermod
man userdel
man passwd
man groupadd
man sudo
man sudoers
man su
```

### Important Files
```bash
/etc/passwd          # User accounts
/etc/shadow          # Encrypted passwords
/etc/group           # Group information
/etc/gshadow         # Group passwords
/etc/sudoers         # Sudo configuration
/etc/sudoers.d/      # Sudo drop-in files
/etc/login.defs      # User/group defaults
/etc/skel/           # Template for home directories
```

### Further Reading
- [Linux Users and Groups](https://wiki.archlinux.org/title/Users_and_groups)
- [sudo Manual](https://www.sudo.ws/docs/man/sudoers.man/)
- [PAM Configuration](https://linux.die.net/man/8/pam)

---

## ✅ Self-Assessment Checklist

Before moving to Chapter 9, ensure you can:
- [ ] Understand user types and UIDs
- [ ] Read and interpret /etc/passwd and /etc/shadow
- [ ] Create users with useradd
- [ ] Modify users with usermod
- [ ] Delete users safely with userdel
- [ ] Manage passwords with passwd
- [ ] Create and manage groups
- [ ] Add users to groups properly
- [ ] Use su to switch users
- [ ] Use sudo for privileged commands
- [ ] Configure sudo access safely with visudo
- [ ] View login history with who, w, last
- [ ] Understand difference between su and sudo
- [ ] Set up shared group directories with SGID

---

## 🚀 Next Steps

Great work! You now understand user and group management - essential for system administration.

In **Chapter 9**, you'll learn:
- Package management systems (APT, YUM/DNF)
- Installing and removing software
- Managing repositories
- Updating the system
- Working with different package managers
- Dependency resolution

**Challenge**: Set up a complete team environment with three users, create a shared group directory with SGID, configure specific sudo permissions for each user, and document the entire process!

---

*"Users and groups are the foundation of Linux security. Master them, and you control access to your entire system."*