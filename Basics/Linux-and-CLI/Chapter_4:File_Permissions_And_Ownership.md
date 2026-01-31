# Chapter 4: File Permissions and Ownership

## 🎯 Learning Objectives
By the end of this chapter, you will:
- Understand Linux permission system in depth
- Use chmod to change file permissions (symbolic and numeric)
- Manage file ownership with chown and chgrp
- Work with special permissions (SUID, SGID, sticky bit)
- Use sudo safely and effectively
- Understand umask and default permissions
- Work with Access Control Lists (ACLs)

---

## 4.1 Understanding Permissions

### The Permission System

Linux uses a permission system to control who can read, write, or execute files. This is fundamental to Linux security.

**Three Permission Types:**
- **r (read)** - Permission to read file contents or list directory contents
- **w (write)** - Permission to modify file or create/delete files in directory
- **x (execute)** - Permission to run file as program or enter directory

**Three Permission Groups:**
- **User (u)** - File owner
- **Group (g)** - Users in the file's group
- **Others (o)** - Everyone else

### Reading Permission Strings

```bash
$ ls -l document.txt
-rw-r--r-- 1 john developers 1024 Jan 30 10:00 document.txt
```

**Breaking Down the Permission String:**
```
-rw-r--r--
│││││││││
│││││││└└─ others: r-- (read only)
│││└└└──── group: r-- (read only)
└└└─────── owner: rw- (read + write)
└────────── file type: - (regular file)
```

### Permissions for Files vs Directories

**For Files:**
- **r (read)**: View file contents (`cat`, `less`, `nano`)
- **w (write)**: Modify file contents, delete file
- **x (execute)**: Run file as program/script

**For Directories:**
- **r (read)**: List directory contents (`ls`)
- **w (write)**: Create/delete files in directory
- **x (execute)**: Enter directory (`cd`), access files inside

**Important:** Directory permissions work differently!
```bash
# Directory with r-- (read only)
$ ls directory/          # Works - can list
$ cd directory/          # Fails - no execute permission
$ cat directory/file     # Fails - can't access files without x

# Directory with --x (execute only)  
$ ls directory/          # Fails - can't list
$ cd directory/          # Works - can enter
$ cat directory/file     # Works if you know filename

# Directory with r-x (read + execute)
$ ls directory/          # Works
$ cd directory/          # Works
$ cat directory/file     # Works
```

### Permission Bits Explained

**Symbolic Notation:**
```
rwxr-xr-x
│││││││││
│││││││└└─ others (o): r-x = 5
│││└└└──── group (g): r-x = 5
└└└─────── user (u): rwx = 7
```

**Numeric Notation:**
```
r = 4 (read)
w = 2 (write)
x = 1 (execute)

Calculate each group:
rwx = 4+2+1 = 7
rw- = 4+2+0 = 6
r-x = 4+0+1 = 5
r-- = 4+0+0 = 4
-wx = 0+2+1 = 3
-w- = 0+2+0 = 2
--x = 0+0+1 = 1
--- = 0+0+0 = 0
```

**Common Permission Combinations:**
```bash
777  rwxrwxrwx  # Everyone can do everything (DANGEROUS!)
755  rwxr-xr-x  # Owner full, others read/execute (scripts, directories)
700  rwx------  # Owner only (private directories)
666  rw-rw-rw-  # Everyone can read/write (DANGEROUS!)
644  rw-r--r--  # Owner write, others read (documents)
600  rw-------  # Owner only (private files, SSH keys)
555  r-xr-xr-x  # Everyone read/execute, nobody write (locked)
444  r--r--r--  # Everyone read only
000  ---------  # Nobody can do anything
```

### Default Permissions and umask

**umask** determines default permissions for newly created files.

```bash
# View current umask
$ umask
0022

# umask is subtracted from base permissions:
# Files: 666 (rw-rw-rw-)
# Directories: 777 (rwxrwxrwx)

# With umask 0022:
# New files: 666 - 022 = 644 (rw-r--r--)
# New directories: 777 - 022 = 755 (rwxr-xr-x)
```

**Common umask Values:**
```bash
0022  # Default - files 644, dirs 755
0002  # Group writable - files 664, dirs 775
0077  # Private - files 600, dirs 700
0000  # No restrictions - files 666, dirs 777 (DANGEROUS!)
```

**Setting umask:**
```bash
# Set umask for current session
$ umask 0027

# Test it
$ touch testfile
$ ls -l testfile
-rw-r----- 1 john john 0 Jan 30 10:00 testfile
# 666 - 027 = 640 (rw-r-----)

# Set permanently in ~/.bashrc
echo "umask 0027" >> ~/.bashrc
```

---

## 4.2 Changing Permissions with chmod

### chmod - Change File Mode

**Basic Syntax:**
```bash
chmod [options] mode file
```

### Symbolic Mode

**Operators:**
- `+` Add permission
- `-` Remove permission
- `=` Set exact permission

**Who:**
- `u` User/owner
- `g` Group
- `o` Others
- `a` All (user + group + others)

**Examples:**
```bash
# Add execute for user
$ chmod u+x script.sh

# Remove write for group and others
$ chmod go-w file.txt

# Add read for all
$ chmod a+r document.txt

# Set exact permissions for group
$ chmod g=rx file.txt

# Multiple changes at once
$ chmod u+x,g+x,o-w script.sh

# Add read and execute for all
$ chmod a+rx directory/
```

**Practical Examples:**
```bash
# Make script executable
$ touch myscript.sh
$ ls -l myscript.sh
-rw-r--r-- 1 john john 0 Jan 30 10:00 myscript.sh
$ chmod u+x myscript.sh
$ ls -l myscript.sh
-rwxr--r-- 1 john john 0 Jan 30 10:00 myscript.sh

# Remove all permissions for others
$ chmod o-rwx private_file.txt
$ ls -l private_file.txt
-rw-r----- 1 john john 100 Jan 30 10:00 private_file.txt

# Set group to read and execute only
$ chmod g=rx program
$ ls -l program
-rwxr-x--- 1 john developers 2048 Jan 30 10:00 program
```

### Numeric Mode

**Syntax:**
```bash
chmod ### file
```

Each digit represents: user, group, others

**Examples:**
```bash
# 755 - rwxr-xr-x (standard for executables)
$ chmod 755 script.sh

# 644 - rw-r--r-- (standard for documents)
$ chmod 644 document.txt

# 600 - rw------- (private file)
$ chmod 600 ~/.ssh/id_rsa

# 700 - rwx------ (private directory)
$ chmod 700 ~/private/

# 777 - rwxrwxrwx (open to everyone - usually bad!)
$ chmod 777 public_share/
```

**Calculating Permissions:**
```bash
# Want: rwxr-x---
# User: rwx = 4+2+1 = 7
# Group: r-x = 4+0+1 = 5
# Others: --- = 0+0+0 = 0
$ chmod 750 file

# Want: rw-rw-r--
# User: rw- = 4+2+0 = 6
# Group: rw- = 4+2+0 = 6
# Others: r-- = 4+0+0 = 4
$ chmod 664 file
```

### Recursive Permission Changes

```bash
# Change all files in directory
$ chmod -R 755 project/

# Change only directories (using find)
$ find project/ -type d -exec chmod 755 {} \;

# Change only files
$ find project/ -type f -exec chmod 644 {} \;

# Safer: use -R with symbolic mode
$ chmod -R u+w,go-w project/
```

**Warning with Recursive:**
```bash
# DANGEROUS - makes everything executable
$ chmod -R 777 /

# SAFER - be specific
$ chmod -R u+rw,go+r,go-w documents/
```

---

## 4.3 Ownership Management

### Understanding Ownership

Every file has:
- **Owner (user)** - The user who owns the file
- **Group** - The group that owns the file

```bash
$ ls -l file.txt
-rw-r--r-- 1 john developers 1024 Jan 30 10:00 file.txt
               │    │
               │    └─ group owner
               └────── user owner
```

### chown - Change Owner

**Syntax:**
```bash
chown [options] user[:group] file
```

**Examples:**
```bash
# Change owner only
$ sudo chown alice file.txt

# Change owner and group
$ sudo chown alice:developers file.txt

# Change owner, group stays same
$ sudo chown alice: file.txt

# Recursive
$ sudo chown -R alice:developers project/

# Change using UID
$ sudo chown 1000:1000 file.txt
```

**Real-World Usage:**
```bash
# Fix web server permissions
$ sudo chown -R www-data:www-data /var/www/html/

# Give file to another user
$ sudo chown jane document.txt

# Transfer directory ownership
$ sudo chown -R bob:bob /home/bob/
```

### chgrp - Change Group

**Syntax:**
```bash
chgrp [options] group file
```

**Examples:**
```bash
# Change group
$ chgrp developers project.txt

# Recursive
$ chgrp -R staff shared_folder/

# Multiple files
$ chgrp developers file1.txt file2.txt file3.txt
```

**Use Cases:**
```bash
# Share files with team
$ chgrp developers shared_project/
$ chmod 770 shared_project/
# Now all developers group members can access

# Set up shared directory
$ sudo mkdir /shared
$ sudo chgrp team /shared
$ sudo chmod 2775 /shared  # SGID bit (explained later)
```

### User and Group IDs

**View IDs:**
```bash
# Your user ID
$ id
uid=1000(john) gid=1000(john) groups=1000(john),27(sudo),1001(developers)

# Another user's ID
$ id alice
uid=1001(alice) gid=1001(alice) groups=1001(alice),1001(developers)

# Just UID
$ id -u
1000

# Just GID
$ id -g
1000

# All groups
$ id -G
1000 27 1001
```

**System Users:**
```bash
# System users typically have UID < 1000
$ grep ^root /etc/passwd
root:x:0:0:root:/root:/bin/bash

$ grep ^www-data /etc/passwd
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

---

## 4.4 sudo and Root Privileges

### Understanding sudo

**sudo** = "Superuser Do" - Execute commands as another user (usually root).

**Why sudo instead of root login?**
- Accountability - Commands are logged with username
- Security - Requires password, limited time window
- Precision - Only specific commands elevated
- Safety - Reduces mistakes

**Basic Usage:**
```bash
# Run single command as root
$ sudo apt update

# Run command as specific user
$ sudo -u alice cat /home/alice/private.txt

# Start shell as root
$ sudo -i
# or
$ sudo -s

# Edit file safely
$ sudo nano /etc/hosts

# Run previous command with sudo
$ apt update
E: Could not open lock file - open (13: Permission denied)
$ sudo !!
# !! repeats last command
```

### Configuring sudo

**sudoers file:** `/etc/sudoers`

**⚠️ NEVER edit directly!** Always use `visudo`:
```bash
$ sudo visudo
```

**sudoers Syntax:**
```bash
# User privilege specification
root    ALL=(ALL:ALL) ALL
john    ALL=(ALL:ALL) ALL

# Allow user to run specific commands
alice   ALL=/usr/bin/apt, /usr/bin/systemctl

# No password required (DANGEROUS!)
bob     ALL=(ALL) NOPASSWD: ALL

# Allow group
%sudo   ALL=(ALL:ALL) ALL
%admin  ALL=(ALL) ALL
```

**Grant sudo Access:**
```bash
# Add user to sudo group (Debian/Ubuntu)
$ sudo usermod -aG sudo username

# Add user to wheel group (Red Hat/CentOS)
$ sudo usermod -aG wheel username

# Check sudo access
$ sudo -l
User john may run the following commands:
    (ALL : ALL) ALL
```

### sudo Best Practices

```bash
# Use sudo for specific commands, not shells
$ sudo apt update           # Good
$ sudo su                   # Avoid

# Check what sudo allows
$ sudo -l

# Use -i for login shell if needed
$ sudo -i

# Use -E to preserve environment
$ sudo -E command

# Specify user explicitly
$ sudo -u www-data command

# Run multiple commands
$ sudo sh -c 'command1; command2; command3'

# Timeout (default: 15 minutes)
# Reset timeout
$ sudo -k
```

---

## 4.5 Special Permissions

### SUID (Set User ID)

**Purpose:** Execute file with permissions of file owner (not runner).

**Numeric Value:** 4000

**Common Example:** `/usr/bin/passwd`
```bash
$ ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 68208 /usr/bin/passwd
    ↑
    s = SUID bit set
```

When you run `passwd`, it runs as root (to modify `/etc/shadow`), even though you're a regular user.

**Setting SUID:**
```bash
# Symbolic
$ chmod u+s program

# Numeric
$ chmod 4755 program

# Result
$ ls -l program
-rwsr-xr-x 1 john john 1024 program
```

**Security Warning:**
```bash
# SUID on shell is VERY DANGEROUS!
$ sudo chmod u+s /bin/bash  # DON'T DO THIS!
# Anyone could get root shell
```

### SGID (Set Group ID)

**On Files:** Execute with group's permissions
**On Directories:** New files inherit directory's group

**Numeric Value:** 2000

**Example with Directories:**
```bash
$ mkdir shared_project
$ sudo chgrp developers shared_project
$ sudo chmod 2775 shared_project
$ ls -ld shared_project
drwxrwsr-x 2 john developers 4096 shared_project
       ↑
       s = SGID bit set

# Now files created here inherit 'developers' group
$ cd shared_project
$ touch newfile.txt
$ ls -l newfile.txt
-rw-r--r-- 1 john developers 0 newfile.txt
                    ↑
                    Inherited group!
```

**Setting SGID:**
```bash
# Symbolic
$ chmod g+s directory/

# Numeric
$ chmod 2775 directory/

# Result
$ ls -ld directory/
drwxrwsr-x 2 john developers 4096 directory/
```

### Sticky Bit

**Purpose:** In a directory, only file owner can delete their files.

**Numeric Value:** 1000

**Common Example:** `/tmp`
```bash
$ ls -ld /tmp
drwxrwxrwt 20 root root 4096 Jan 30 10:00 /tmp
         ↑
         t = sticky bit set
```

Everyone can write to `/tmp`, but users can only delete their own files.

**Setting Sticky Bit:**
```bash
# Symbolic
$ chmod +t directory/

# Numeric
$ chmod 1777 directory/

# Result
$ ls -ld directory/
drwxrwxrwt 2 john john 4096 directory/
```

**Use Case:**
```bash
# Shared upload directory
$ mkdir /shared/uploads
$ chmod 1777 /shared/uploads

# Users can upload files but can't delete others' files
```

### Special Permissions Summary

| Permission | Numeric | Symbol | Effect on Files | Effect on Directories |
|------------|---------|--------|-----------------|----------------------|
| SUID | 4000 | s (user) | Execute as owner | (No effect) |
| SGID | 2000 | s (group) | Execute as group | Inherit group ownership |
| Sticky | 1000 | t (others) | (No effect) | Only owner can delete |

**Combined Example:**
```bash
# SUID + regular permissions
$ chmod 4755 file
$ ls -l file
-rwsr-xr-x

# SGID + regular permissions
$ chmod 2755 directory
$ ls -ld directory
drwxr-sr-x

# Sticky + regular permissions
$ chmod 1777 directory
$ ls -ld directory
drwxrwxrwt

# All special bits (rare)
$ chmod 7777 file  # Don't actually do this!
```

---

## 4.6 Access Control Lists (ACLs)

### What are ACLs?

ACLs provide more fine-grained permissions than standard Unix permissions.

**Standard Permissions:**
- Only 3 categories: user, group, others
- One user, one group

**ACLs:**
- Multiple users with different permissions
- Multiple groups with different permissions

### Viewing ACLs

```bash
# Check if file has ACL
$ ls -l file.txt
-rw-r--r--+ 1 john john 1024 file.txt
           ↑
           + indicates ACL is set

# View ACLs
$ getfacl file.txt
# file: file.txt
# owner: john
# group: john
user::rw-
user:alice:rw-
group::r--
mask::rw-
other::r--
```

### Setting ACLs

```bash
# Give alice read/write access
$ setfacl -m u:alice:rw file.txt

# Give developers group read/execute
$ setfacl -m g:developers:rx file.txt

# Remove ACL for alice
$ setfacl -x u:alice file.txt

# Remove all ACLs
$ setfacl -b file.txt

# Recursive
$ setfacl -R -m u:alice:rw directory/
```

**Default ACLs (for directories):**
```bash
# Set default ACL - applies to new files
$ setfacl -d -m u:alice:rw directory/

# New files in directory/ will give alice rw access
```

### ACL Examples

**Scenario: Shared Project Directory**
```bash
# Create directory
$ mkdir project
$ chmod 770 project

# Give alice full access
$ setfacl -m u:alice:rwx project/

# Give bob read-only
$ setfacl -m u:bob:r-x project/

# Set default for new files
$ setfacl -d -m u:alice:rwx project/
$ setfacl -d -m u:bob:r-x project/

# Verify
$ getfacl project/
# file: project/
# owner: john
# group: john
user::rwx
user:alice:rwx
user:bob:r-x
group::rwx
mask::rwx
other::---
default:user::rwx
default:user:alice:rwx
default:user:bob:r-x
default:group::rwx
default:mask::rwx
default:other::---
```

---

## 💡 Key Concepts Summary

1. **Permissions** control read, write, execute for user, group, others
2. **chmod** changes permissions (symbolic: u+x, numeric: 755)
3. **chown** changes owner, **chgrp** changes group
4. **sudo** allows running commands as root safely
5. **SUID/SGID/Sticky bit** provide special behaviors
6. **umask** sets default permissions for new files
7. **ACLs** provide fine-grained permissions beyond standard Unix model

---

## 🔥 Real-World Use Cases

### Use Case 1: Setting Up a Shared Development Directory

```bash
# Create directory
$ sudo mkdir /shared/dev-team

# Set group ownership
$ sudo chgrp developers /shared/dev-team

# Set SGID so new files inherit group
$ sudo chmod 2775 /shared/dev-team

# Verify
$ ls -ld /shared/dev-team
drwxrwsr-x 2 root developers 4096 /shared/dev-team

# Team members can now collaborate
$ cd /shared/dev-team
$ touch newfile.txt
$ ls -l newfile.txt
-rw-r--r-- 1 john developers 0 newfile.txt
                   ↑ Inherited!
```

### Use Case 2: Securing SSH Keys

```bash
# Private key must be read-only by owner
$ chmod 600 ~/.ssh/id_rsa
$ ls -l ~/.ssh/id_rsa
-rw------- 1 john john 1234 ~/.ssh/id_rsa

# Public key can be readable by all
$ chmod 644 ~/.ssh/id_rsa.pub
$ ls -l ~/.ssh/id_rsa.pub
-rw-r--r-- 1 john john 567 ~/.ssh/id_rsa.pub

# SSH directory
$ chmod 700 ~/.ssh
$ ls -ld ~/.ssh
drwx------ 2 john john 4096 ~/.ssh
```

### Use Case 3: Web Server Permissions

```bash
# Web files owned by web server user
$ sudo chown -R www-data:www-data /var/www/html/

# Directories: 755 (rwxr-xr-x)
$ sudo find /var/www/html/ -type d -exec chmod 755 {} \;

# Files: 644 (rw-r--r--)
$ sudo find /var/www/html/ -type f -exec chmod 644 {} \;

# Upload directory: writable by web server
$ sudo chmod 775 /var/www/html/uploads/
```

### Use Case 4: Script Deployment

```bash
# Make script executable
$ chmod +x deploy.sh

# Only owner can modify
$ chmod 744 deploy.sh
$ ls -l deploy.sh
-rwxr--r-- 1 john john 2048 deploy.sh

# Execute
$ ./deploy.sh
```

### Use Case 5: Temporary Shared Upload

```bash
# Create upload directory
$ sudo mkdir /uploads

# Everyone can write, but can only delete own files
$ sudo chmod 1777 /uploads
$ ls -ld /uploads
drwxrwxrwt 2 root root 4096 /uploads

# Users can upload but not delete others' files
```

---

## 🧪 Practice Problems

### Beginner Level

**Problem 1**: Create a file called `test.txt` with default permissions. What are they? Change them to 644.

**Problem 2**: Create a script file `hello.sh`, make it executable for the owner only.

**Problem 3**: Create a directory with permissions 755. Explain what each digit means.

**Problem 4**: View the permissions of `/etc/passwd`. Why do you think it has those permissions?

**Problem 5**: What umask value would give new files permissions of 640?

### Intermediate Level

**Problem 6**: Create a directory called `shared`. Set it up so:
- Owner has full permissions
- Group has read and execute
- Others have no permissions

**Problem 7**: Make a copy of `/bin/ls` in your home directory. Try to change its owner to root. What happens and why?

**Problem 8**: Find all files in your home directory that are executable by owner. (Hint: use `find` with `-perm`)

**Problem 9**: Create a directory with SGID bit set. Create a file inside it. What group does the file have?

**Problem 10**: Give user `alice` read and write access to a file using ACLs (you can practice the command syntax even if alice doesn't exist).

### Advanced Level

**Problem 11**: Create a collaborative directory where:
- All team members can read/write/execute
- New files automatically belong to the team group
- Only file owners can delete their own files
What commands would you use?

**Problem 12**: You have a script that needs to write to a log file owned by root. The script should run as a normal user but write to the log. How would you set this up? (Consider SUID, sudo, or group permissions)

**Problem 13**: Find all SUID files on the system. Are any unexpected?
```bash
find / -perm -4000 -type f 2>/dev/null
```

**Problem 14**: Calculate the numeric permission that represents:
- Owner: read, write, execute
- Group: read, execute
- Others: execute only

**Problem 15**: Explain the security implications of `chmod 777 -R /` and why this should never be done.

---

## 📝 Solutions

### Beginner Solutions

**Solution 1**:
```bash
$ touch test.txt
$ ls -l test.txt
-rw-r--r-- 1 john john 0 Jan 30 10:00 test.txt
# Default: 644 (rw-r--r--)

$ chmod 644 test.txt
# Already 644, but command works
```

**Solution 2**:
```bash
$ touch hello.sh
$ chmod u+x hello.sh
# or
$ chmod 744 hello.sh

$ ls -l hello.sh
-rwxr--r-- 1 john john 0 Jan 30 10:00 hello.sh
```

**Solution 3**:
```bash
$ mkdir mydir
$ chmod 755 mydir
$ ls -ld mydir
drwxr-xr-x 2 john john 4096 Jan 30 10:00 mydir

# 755 means:
# 7 (owner): rwx = read + write + execute
# 5 (group): r-x = read + execute
# 5 (others): r-x = read + execute
```

**Solution 4**:
```bash
$ ls -l /etc/passwd
-rw-r--r-- 1 root root 2847 Jan 25 15:30 /etc/passwd

# Permissions: 644 (rw-r--r--)
# Reason:
# - Owner (root) can read and write (to add/modify users)
# - Others can only read (to look up usernames, UIDs)
# - Nobody else should modify it (security)
# - Must be readable by all (many programs need user info)
```

**Solution 5**:
```bash
# New files default: 666 (rw-rw-rw-)
# Desired: 640 (rw-r-----)
# Difference: 026
# umask: 0026

$ umask 0026
$ touch testfile
$ ls -l testfile
-rw-r----- 1 john john 0 Jan 30 10:00 testfile
```

### Intermediate Solutions

**Solution 6**:
```bash
$ mkdir shared
$ chmod 750 shared
# or
$ chmod u=rwx,g=rx,o= shared

$ ls -ld shared
drwxr-x--- 2 john john 4096 Jan 30 10:00 shared

# Breakdown:
# 7 (owner): rwx
# 5 (group): r-x
# 0 (others): ---
```

**Solution 7**:
```bash
$ cp /bin/ls ~/myls
$ ls -l ~/myls
-rwxr-xr-x 1 john john 138856 Jan 30 10:00 /home/john/myls

$ chown root ~/myls
chown: changing ownership of '/home/john/myls': Operation not permitted

# Why: Only root can change file ownership to another user
# This prevents users from giving away files to bypass quotas
# or to make files look like they belong to root

$ sudo chown root ~/myls
# This would work
```

**Solution 8**:
```bash
# Find files with execute permission for user
$ find ~ -type f -perm -u+x

# or more specifically
$ find ~ -type f -perm -100

# Explanation:
# -perm -u+x: has user execute permission
# -perm -100: octal notation (1 in user execute position)
```

**Solution 9**:
```bash
$ mkdir sgid_test
$ chmod 2755 sgid_test
# or
$ chmod g+s sgid_test

$ ls -ld sgid_test
drwxr-sr-x 2 john john 4096 Jan 30 10:00 sgid_test
              ↑ SGID bit

$ cd sgid_test
$ touch newfile.txt
$ ls -l newfile.txt
-rw-r--r-- 1 john john 0 Jan 30 10:00 newfile.txt

# The file has the same group as the directory (john)
# because SGID causes new files to inherit directory's group
```

**Solution 10**:
```bash
# Give alice read/write access
$ setfacl -m u:alice:rw myfile.txt

# Verify
$ getfacl myfile.txt
# file: myfile.txt
# owner: john
# group: john
user::rw-
user:alice:rw-
group::r--
mask::rw-
other::r--

# Check with ls
$ ls -l myfile.txt
-rw-rw-r--+ 1 john john 0 Jan 30 10:00 myfile.txt
           ↑ indicates ACL is present
```

### Advanced Solutions

**Solution 11**:
```bash
# Create directory
$ mkdir collab_dir

# Set group ownership (assuming 'team' group exists)
$ sudo chgrp team collab_dir

# Set permissions with SGID and sticky bit
$ chmod 2775 collab_dir
# or
$ chmod g+s,o+t collab_dir
# and
$ chmod 775 collab_dir

# Full command
$ chmod 3775 collab_dir

$ ls -ld collab_dir
drwxrwsr-t 2 john team 4096 Jan 30 10:00 collab_dir

# Explanation:
# 3775 = 1000 (sticky) + 2000 (SGID) + 775
# rwx (owner), rws (group - SGID), r-t (others - sticky)
# - All team members can read/write/execute (7)
# - New files inherit team group (SGID)
# - Only file owner can delete own files (sticky bit)
```

**Solution 12**:
```bash
# Option 1: SUID approach (if script is compiled binary)
$ sudo chown root script
$ sudo chmod 4755 script
# Script runs as root, can write to log

# Option 2: Group permissions (better for scripts)
$ sudo touch /var/log/myapp.log
$ sudo chgrp logwriters /var/log/myapp.log
$ sudo chmod 664 /var/log/myapp.log
$ sudo usermod -aG logwriters john
# User john (in logwriters group) can write to log

# Option 3: Sudo with specific command
# In /etc/sudoers (via visudo):
# john ALL=(ALL) NOPASSWD: /usr/bin/tee -a /var/log/myapp.log
# In script:
# echo "log entry" | sudo tee -a /var/log/myapp.log

# Option 2 is generally best for log files
```

**Solution 13**:
```bash
$ sudo find / -perm -4000 -type f 2>/dev/null
/usr/bin/passwd
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
...

# Expected SUID files:
# - passwd (to modify /etc/shadow)
# - sudo (to run as root)
# - ping (to send ICMP packets)

# Unexpected files would be:
# - /bin/bash (should never have SUID)
# - Unknown binaries
# - Files in /tmp or user directories

# Security check: Research any unexpected SUID files!
```

**Solution 14**:
```bash
# Owner: rwx = 4+2+1 = 7
# Group: r-x = 4+0+1 = 5
# Others: --x = 0+0+1 = 1

# Answer: 751

$ chmod 751 file
$ ls -l file
-rwxr-x--x 1 john john 0 Jan 30 10:00 file
```

**Solution 15**:
```bash
# chmod 777 -R / would:

# 1. Security Nightmare:
#    - All files readable by everyone (passwords, keys, configs)
#    - All files writable by everyone (system corruption)
#    - All files executable (potential malware execution)

# 2. System Instability:
#    - SSH won't work (keys must be 600)
#    - Sudo won't work (sudoers must be 440)
#    - Many services will refuse to start

# 3. Data Loss Risk:
#    - Any user can delete any file
#    - No protection against accidents
#    - No file ownership respected

# 4. Unfixable Damage:
#    - Some programs check permissions on startup
#    - Can't easily restore correct permissions
#    - May require reinstallation

# Real-world impact:
# - SSH daemon will refuse to start
# - System basically becomes unusable
# - Massive security breach

# NEVER RUN THIS COMMAND!
```

---

## 🎓 Learning Tips

### Mnemonics for Permissions

**"Read Write eXecute"**
- **R**ead a book (4)
- **W**rite a letter (2)
- e**X**ecute a program (1)

**"User Group Others"**
- **U**ncle **G**ave **O**ranges

**Common Patterns:**
- **777** - "Triple 7" - everyone can do everything
- **755** - "Standard directory" - owner full, others read/execute
- **644** - "Standard file" - owner write, others read
- **600** - "Secret file" - owner only

### Permission Quick Reference

```
Remember the pattern:
rwx rwx rwx
421 421 421
```

**Calculate mentally:**
- Need r + x? That's 4 + 1 = 5
- Need r + w? That's 4 + 2 = 6
- Need all? That's 4 + 2 + 1 = 7

### Best Practices

1. **Principle of Least Privilege**
   - Give minimum permissions needed
   - Don't default to 777

2. **Secure SSH Keys**
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/id_rsa
   chmod 644 ~/.ssh/id_rsa.pub
   chmod 644 ~/.ssh/authorized_keys
   chmod 644 ~/.ssh/known_hosts
   ```

3. **Web Server Files**
   ```bash
   Directories: 755
   Files: 644
   Upload directories: 775 (with proper group)
   ```

4. **Scripts**
   ```bash
   Development: 744 (rwxr--r--)
   Production: 755 (rwxr-xr-x)
   Private: 700 (rwx------)
   ```

5. **Shared Directories**
   ```bash
   Use SGID: chmod 2775
   Consider sticky bit for uploads: chmod 1777
   ```

### Common Mistakes to Avoid

1. **Don't use 777**
   ```bash
   # WRONG
   chmod 777 file
   
   # RIGHT - be specific
   chmod 755 file
   ```

2. **Don't recursively change to 777**
   ```bash
   # DISASTER
   chmod -R 777 /
   
   # BETTER
   find /path -type d -exec chmod 755 {} \;
   find /path -type f -exec chmod 644 {} \;
   ```

3. **Don't forget sudo for system files**
   ```bash
   # FAILS
   chmod 644 /etc/hosts
   
   # WORKS
   sudo chmod 644 /etc/hosts
   ```

4. **Don't set SUID on scripts**
   ```bash
   # DANGEROUS - doesn't work as expected and is insecure
   chmod u+s script.sh
   ```

5. **Don't change permissions of files you don't own**
   - Always use sudo when necessary
   - Understand what you're changing

---

## 🔗 Connections to Future Chapters

- **Chapter 5**: Text processing requires proper read permissions
- **Chapter 7**: Process management - running programs requires execute permission
- **Chapter 8**: User management integrates with file ownership
- **Chapter 11**: Shell scripts need execute permissions
- **Chapter 13**: Log file analysis requires read permissions
- **Chapter 19**: Security depends heavily on proper permissions

---

## 📚 Additional Resources

### Man Pages to Read
```bash
man chmod
man chown
man chgrp
man umask
man sudo
man sudoers
man setfacl
man getfacl
```

### Important Files
```bash
/etc/sudoers        # Sudo configuration
/etc/group          # Group definitions
/etc/passwd         # User definitions
/etc/shadow         # Password hashes (600 permissions!)
```

### Further Reading
- Linux File Permissions Guide: https://wiki.archlinux.org/title/File_permissions_and_attributes
- Understanding umask: https://www.cyberciti.biz/tips/understanding-linux-unix-umask-value-usage.html
- sudo Best Practices: https://www.sudo.ws/docs/man/sudoers.man/

### Online Tools
- Permissions Calculator: https://chmod-calculator.com/
- Permission Visualizer: visualize rwx → 755

---

## ✅ Self-Assessment Checklist

Before moving to Chapter 5, ensure you can:
- [ ] Read and interpret permission strings (rwxr-xr-x)
- [ ] Convert between symbolic and numeric permissions
- [ ] Use chmod with both symbolic and numeric modes
- [ ] Change file ownership with chown and chgrp
- [ ] Use sudo safely and understand why it's better than root login
- [ ] Explain what SUID, SGID, and sticky bit do
- [ ] Calculate and set umask values
- [ ] Use ACLs for fine-grained permissions
- [ ] Secure SSH keys and important files properly
- [ ] Understand the principle of least privilege
- [ ] Know common permission patterns (755, 644, 600)
- [ ] Debug permission-related errors

---

## 🚀 Next Steps

Excellent work! You now understand Linux permissions and ownership - crucial for system security and proper file management.

In **Chapter 5**, you'll learn:
- Text processing with grep, sed, and awk
- Searching and filtering text
- Text transformation and manipulation
- Using powerful text editors (vim)
- Regular expressions basics

**Challenge**: Set up a shared project directory for a team with proper permissions, SGID bit, and appropriate access controls. Document your commands and explain why you chose each permission setting.

---

*"Good permissions are invisible when right, catastrophic when wrong. Master them, and you master Linux security."*