# Chapter 2: Basic Shell Commands

## 🎯 Learning Objectives
By the end of this chapter, you will:
- Navigate the Linux filesystem confidently
- Create, copy, move, and delete files and directories
- View and examine file contents
- Find help and documentation for commands

---

## 2.1 Navigation Commands

### The pwd Command - Print Working Directory

**Purpose**: Shows your current location in the filesystem.

```bash
$ pwd
/home/john/Documents
```

**Explanation**: 
- `pwd` = Print Working Directory
- Always shows **absolute path** from root (`/`)
- Essential for knowing where you are

**Use Cases**:
```bash
# Before running destructive commands
$ pwd
/home/john/important_docs
# Good, safe location

# Verify location in scripts
if [ "$(pwd)" = "/home/john/projects" ]; then
    echo "In projects directory"
fi
```

---

### The ls Command - List Directory Contents

**Basic Usage**:
```bash
$ ls
Desktop  Documents  Downloads  Music  Pictures  Videos
```

**Common Options**:

```bash
# Long format (detailed)
$ ls -l
total 24
drwxr-xr-x 2 john john 4096 Jan 30 10:00 Desktop
drwxr-xr-x 5 john john 4096 Jan 30 09:30 Documents
drwxr-xr-x 2 john john 4096 Jan 30 08:00 Downloads

# Show hidden files (starting with .)
$ ls -a
.  ..  .bashrc  .profile  Desktop  Documents

# Human-readable sizes
$ ls -lh
total 24K
drwxr-xr-x 2 john john 4.0K Jan 30 10:00 Desktop
drwxr-xr-x 5 john john 4.0K Jan 30 09:30 Documents

# Sort by modification time (newest first)
$ ls -lt

# Reverse order
$ ls -lr

# Recursive (show subdirectories)
$ ls -R

# Combine options
$ ls -lah
```

**Understanding ls -l Output**:
```
-rw-r--r-- 1 john john 4096 Jan 30 10:00 file.txt
│││││││││  │ │    │    │    │            └─ filename
│││││││││  │ │    │    │    └── modification time
│││││││││  │ │    │    └──────── file size (bytes)
│││││││││  │ │    └───────────── group owner
│││││││││  │ └────────────────── user owner
│││││││││  └─────────────────── number of links
││││││││└── execute (others)
│││││││└─── write (others)
││││││└──── read (others)
│││││└───── execute (group)
││││└────── write (group)
│││└─────── read (group)
││└──────── execute (owner)
│└───────── write (owner)
└────────── read (owner) / file type
```

**File Type Indicators**:
- `-` : Regular file
- `d` : Directory
- `l` : Symbolic link
- `c` : Character device
- `b` : Block device

---

### The cd Command - Change Directory

**Basic Usage**:
```bash
# Go to Documents
$ cd Documents

# Go to home directory
$ cd
# or
$ cd ~

# Go to previous directory
$ cd -

# Go up one level
$ cd ..

# Go up two levels
$ cd ../..

# Absolute path
$ cd /var/log

# Relative path
$ cd ./subfolder/data
```

**Special Directory Symbols**:
```bash
.   # Current directory
..  # Parent directory
~   # Home directory
-   # Previous directory
/   # Root directory
```

**Examples**:
```bash
$ pwd
/home/john

$ cd Documents
$ pwd
/home/john/Documents

$ cd ..
$ pwd
/home/john

$ cd /etc
$ pwd
/etc

$ cd -
/home/john
```

---

### Understanding Paths

**Absolute Path**:
- Starts from root (`/`)
- Complete path from the beginning
- Same meaning regardless of current location

```bash
/home/john/Documents/report.txt
/etc/nginx/nginx.conf
/var/log/syslog
```

**Relative Path**:
- Relative to current directory
- Does not start with `/`
- Shorter but depends on where you are

```bash
# If you're in /home/john
Documents/report.txt        # same as /home/john/Documents/report.txt
./Documents/report.txt      # explicitly current directory
../jane/Pictures            # same as /home/jane/Pictures
```

**Comparison**:
```bash
$ pwd
/home/john/Documents

# Absolute - always works
$ ls /home/john/Documents/projects

# Relative - depends on current location
$ ls ./projects
$ ls projects              # ./ is optional
```

---

## 2.2 File and Directory Operations

### mkdir - Make Directory

**Basic Usage**:
```bash
# Create single directory
$ mkdir new_folder

# Create multiple directories
$ mkdir folder1 folder2 folder3

# Create parent directories as needed
$ mkdir -p projects/web/frontend/src
# Creates entire path even if intermediate dirs don't exist

# Create with specific permissions
$ mkdir -m 755 public_folder
```

**Real-World Example**:
```bash
# Set up a project structure
$ mkdir -p myproject/{src,tests,docs,data}
$ ls myproject/
src  tests  docs  data
```

---

### touch - Create Empty Files

**Basic Usage**:
```bash
# Create new file
$ touch newfile.txt

# Create multiple files
$ touch file1.txt file2.txt file3.txt

# Update timestamp of existing file
$ touch existing_file.txt
```

**Use Cases**:
```bash
# Create placeholder files
$ touch README.md LICENSE .gitignore

# Create test files
$ touch test_{1..5}.txt
# Creates: test_1.txt test_2.txt test_3.txt test_4.txt test_5.txt
```

---

### cp - Copy Files and Directories

**Basic Syntax**: `cp [options] source destination`

```bash
# Copy file
$ cp file.txt backup.txt

# Copy file to directory
$ cp file.txt Documents/

# Copy multiple files to directory
$ cp file1.txt file2.txt file3.txt Documents/

# Copy directory (recursive)
$ cp -r folder1 folder2

# Copy with preservation of attributes
$ cp -p file.txt backup.txt
# Preserves: mode, ownership, timestamps

# Interactive (ask before overwrite)
$ cp -i file.txt existing.txt

# Verbose (show what's being done)
$ cp -v file.txt backup.txt
'file.txt' -> 'backup.txt'

# Combine options
$ cp -riv source_dir/ backup_dir/
```

**Practical Examples**:
```bash
# Backup configuration file
$ cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.backup

# Copy entire project
$ cp -r ~/projects/webapp ~/backups/webapp_$(date +%Y%m%d)

# Copy only newer files
$ cp -u source/* destination/
```

---

### mv - Move and Rename Files

**Basic Syntax**: `mv [options] source destination`

```bash
# Rename file
$ mv oldname.txt newname.txt

# Move file to directory
$ mv file.txt Documents/

# Move multiple files
$ mv file1.txt file2.txt file3.txt Documents/

# Move and rename
$ mv ~/Downloads/document.pdf ~/Documents/important_doc.pdf

# Move directory
$ mv old_folder new_folder

# Interactive mode
$ mv -i file.txt existing.txt

# No clobber (don't overwrite)
$ mv -n file.txt existing.txt

# Verbose
$ mv -v old.txt new.txt
renamed 'old.txt' -> 'new.txt'
```

**Real-World Examples**:
```bash
# Organize downloads
$ mv ~/Downloads/*.pdf ~/Documents/PDFs/
$ mv ~/Downloads/*.jpg ~/Pictures/

# Rename with timestamp
$ mv logfile.txt logfile_$(date +%Y%m%d).txt
```

---

### rm - Remove Files and Directories

**⚠️ WARNING**: `rm` is permanent! No recycle bin!

```bash
# Remove file
$ rm file.txt

# Remove multiple files
$ rm file1.txt file2.txt

# Interactive (ask for each file)
$ rm -i file.txt
rm: remove regular file 'file.txt'? y

# Force (no prompts)
$ rm -f file.txt

# Remove directory and contents (recursive)
$ rm -r folder/

# Combine for safe removal of directory
$ rm -ri folder/

# Remove empty directory
$ rmdir empty_folder/

# Verbose
$ rm -v file.txt
removed 'file.txt'
```

**Safety Tips**:
```bash
# Always verify what you're deleting
$ ls file*
file1.txt  file2.txt  file_important.txt

# Use -i for interactive confirmation
$ rm -i file*

# For directories, list first
$ ls -la folder/
# Then remove
$ rm -r folder/

# Double-check path
$ pwd
/home/john/trash
$ rm -rf *  # Safe in trash folder
```

**Common Mistakes to Avoid**:
```bash
# DANGEROUS - deletes everything in current directory
$ rm -rf *

# VERY DANGEROUS - never do this!
$ rm -rf /

# Accidental patterns
$ rm * .txt  # Deletes all files, then tries to delete ".txt"
$ rm *.txt   # Correct - deletes only .txt files
```

---

## 2.3 Viewing File Contents

### cat - Concatenate and Display Files

**Basic Usage**:
```bash
# Display file contents
$ cat file.txt
Hello World
This is a test file.

# Display multiple files
$ cat file1.txt file2.txt

# Number lines
$ cat -n file.txt
     1  Hello World
     2  This is a test file.
     3  

# Show non-printing characters
$ cat -A file.txt
Hello World$
This is a test file.$
```

**Practical Uses**:
```bash
# Create small file quickly
$ cat > newfile.txt
Type content here
Press Ctrl+D when done

# Append to file
$ cat >> existing.txt
Additional content
Ctrl+D

# Combine files
$ cat file1.txt file2.txt > combined.txt

# Display configuration file
$ cat /etc/hostname
```

---

### less and more - Page Through Files

**less (recommended)**:
```bash
$ less large_file.txt
```

**Navigation in less**:
- `Space` - Next page
- `b` - Previous page
- `g` - Go to beginning
- `G` - Go to end
- `/pattern` - Search forward
- `?pattern` - Search backward
- `n` - Next search result
- `N` - Previous search result
- `q` - Quit

**more (older, simpler)**:
```bash
$ more file.txt
```
- `Space` - Next page
- `Enter` - Next line
- `q` - Quit
- Limited backward navigation

**When to use what**:
- **less**: Large files, need search, backward navigation
- **more**: Quick view, forward-only, older systems
- **cat**: Small files, piping output, combining files

---

### head and tail - View Beginning or End

**head - First Lines**:
```bash
# Default: first 10 lines
$ head file.txt

# First 20 lines
$ head -n 20 file.txt
# Or
$ head -20 file.txt

# First 100 bytes
$ head -c 100 file.txt

# Multiple files
$ head file1.txt file2.txt
==> file1.txt <==
[first 10 lines]

==> file2.txt <==
[first 10 lines]
```

**tail - Last Lines**:
```bash
# Default: last 10 lines
$ tail file.txt

# Last 20 lines
$ tail -n 20 file.txt

# Follow file (live updates)
$ tail -f /var/log/syslog
# Press Ctrl+C to stop

# Start from line 5
$ tail -n +5 file.txt
```

**Real-World Examples**:
```bash
# Check start of log file
$ head -20 /var/log/syslog

# Monitor live logs
$ tail -f /var/log/apache2/access.log

# See recent errors
$ tail -50 /var/log/errors.log

# Check first few lines of CSV
$ head -5 data.csv
```

---

### wc - Word, Line, and Character Count

**Basic Usage**:
```bash
$ wc file.txt
 100  500 3000 file.txt
  │    │    │   └── filename
  │    │    └────── bytes
  │    └─────────── words
  └──────────────── lines

# Count only lines
$ wc -l file.txt
100 file.txt

# Count only words
$ wc -w file.txt
500 file.txt

# Count only characters
$ wc -c file.txt
3000 file.txt

# Count multiple files
$ wc *.txt
 100  500 3000 file1.txt
 200 1000 6000 file2.txt
 300 1500 9000 total
```

**Practical Uses**:
```bash
# Count lines of code
$ wc -l *.py
  150 main.py
  200 utils.py
  350 total

# Count files in directory
$ ls | wc -l

# Check log file size
$ wc -l /var/log/syslog
```

---

## 2.4 Getting Help

### man - Manual Pages

**Usage**:
```bash
$ man command_name
```

**Navigation**:
- Arrow keys: Move up/down
- `Space`: Next page
- `/pattern`: Search
- `n`: Next search result
- `q`: Quit

**Man Page Sections**:
```bash
1. User commands
2. System calls
3. Library functions
4. Special files
5. File formats
6. Games
7. Miscellaneous
8. System administration commands

# View specific section
$ man 5 passwd     # File format
$ man 1 passwd     # Command
```

**Example**:
```bash
$ man ls
LS(1)                    User Commands                    LS(1)

NAME
       ls - list directory contents

SYNOPSIS
       ls [OPTION]... [FILE]...

DESCRIPTION
       List information about the FILEs...
```

---

### --help Flag

**Usage**:
```bash
$ command --help
# or
$ command -h
```

**Example**:
```bash
$ ls --help
Usage: ls [OPTION]... [FILE]...
List information about the FILEs (the current directory by default).

Mandatory arguments to long options are mandatory for short options too.
  -a, --all                  do not ignore entries starting with .
  -A, --almost-all           do not list implied . and ..
  -l                         use a long listing format
...
```

**Advantages**:
- Quick reference
- No need to open separate viewer
- Shows version info

---

### Other Help Commands

**info - Info Pages**:
```bash
$ info command_name
# More detailed than man pages
# Hyperlinked documentation
```

**apropos - Search Man Pages**:
```bash
$ apropos "list directory"
ls (1)               - list directory contents

$ apropos copy
cp (1)               - copy files and directories
...
```

**whatis - Brief Description**:
```bash
$ whatis ls
ls (1)               - list directory contents

$ whatis cp mv rm
cp (1)               - copy files and directories
mv (1)               - move (rename) files
rm (1)               - remove files or directories
```

**type - Command Information**:
```bash
$ type ls
ls is aliased to `ls --color=auto'

$ type cd
cd is a shell builtin

$ type python
python is /usr/bin/python
```

---

## 💡 Key Concepts Summary

1. **pwd** shows where you are
2. **ls** shows what's in a directory
3. **cd** moves you around
4. **mkdir** creates directories
5. **touch** creates files
6. **cp** copies, **mv** moves/renames, **rm** deletes
7. **cat** displays, **less** pages, **head/tail** show portions
8. **wc** counts lines/words/characters
9. **man** provides detailed help

---

## 🔥 Real-World Use Cases

### Use Case 1: Organizing Download Folder

```bash
# Navigate to Downloads
$ cd ~/Downloads

# See what's there
$ ls -lh

# Create organization folders
$ mkdir -p {Documents,Images,Videos,Archives}

# Move files
$ mv *.pdf Documents/
$ mv *.jpg *.png Images/
$ mv *.mp4 *.avi Videos/
$ mv *.zip *.tar.gz Archives/

# Verify
$ ls -R
```

### Use Case 2: Project Setup

```bash
# Create project structure
$ mkdir -p myapp/{src,tests,docs,config}
$ cd myapp

# Create initial files
$ touch README.md LICENSE
$ touch src/{main.py,utils.py}
$ touch tests/test_main.py
$ touch config/{dev.conf,prod.conf}

# Verify structure
$ ls -R
```

### Use Case 3: Log Analysis

```bash
# Check log file size
$ wc -l /var/log/syslog

# See recent entries
$ tail -20 /var/log/syslog

# Follow live log
$ tail -f /var/log/syslog

# Search for errors
$ less /var/log/syslog
# Press / and type "error"
```

### Use Case 4: Backup Workflow

```bash
# Create backup directory with date
$ mkdir ~/backups/backup_$(date +%Y%m%d)

# Copy important files
$ cp -r ~/Documents ~/backups/backup_$(date +%Y%m%d)/

# Verify backup
$ ls -lh ~/backups/backup_$(date +%Y%m%d)/

# Count files backed up
$ find ~/backups/backup_$(date +%Y%m%d)/ -type f | wc -l
```

---

## 🧪 Practice Problems

### Beginner Level

**Problem 1**: Navigate to your home directory, create a folder called "practice", and verify it was created.

**Problem 2**: Inside "practice", create three subdirectories: "folder1", "folder2", "folder3" in one command.

**Problem 3**: Create 5 empty files named file1.txt through file5.txt using a single command.

**Problem 4**: List all files in long format showing human-readable sizes.

**Problem 5**: Copy file1.txt to file1_backup.txt and verify both files exist.

### Intermediate Level

**Problem 6**: Create a directory structure: `projects/web/frontend/src` in one command.

**Problem 7**: Move all .txt files from current directory to folder1, then verify they're there.

**Problem 8**: Display the first 5 lines of /etc/passwd and explain what you see.

**Problem 9**: Count how many directories are in /etc using ls and wc.

**Problem 10**: Create a file called notes.txt, add some text to it using cat, then display it.

### Advanced Level

**Problem 11**: Create a script-like sequence to organize files by extension into separate folders.

**Problem 12**: Compare the output of `ls -lt` and `ls -lS`. What's the difference?

**Problem 13**: Use a combination of commands to find and display the 5 largest files in your home directory.

**Problem 14**: Create a "project template" with one command that creates this structure:
```
project/
├── src/
├── tests/
├── docs/
├── config/
└── README.md
```

**Problem 15**: Without using cd, list contents of a directory three levels up from your current location.

---

## 📝 Solutions

### Beginner Solutions

**Solution 1**:
```bash
$ cd ~
$ mkdir practice
$ ls -d practice
practice
```

**Solution 2**:
```bash
$ cd practice
$ mkdir folder1 folder2 folder3
# Or: mkdir folder{1..3}
$ ls
folder1  folder2  folder3
```

**Solution 3**:
```bash
$ touch file{1..5}.txt
$ ls
file1.txt  file2.txt  file3.txt  file4.txt  file5.txt
```

**Solution 4**:
```bash
$ ls -lh
```

**Solution 5**:
```bash
$ cp file1.txt file1_backup.txt
$ ls -l file1*
-rw-r--r-- 1 john john 0 Jan 30 10:00 file1.txt
-rw-r--r-- 1 john john 0 Jan 30 10:01 file1_backup.txt
```

### Intermediate Solutions

**Solution 6**:
```bash
$ mkdir -p projects/web/frontend/src
$ ls -R projects
```

**Solution 7**:
```bash
$ mv *.txt folder1/
$ ls folder1/
file1.txt  file2.txt  file3.txt  file4.txt  file5.txt
```

**Solution 8**:
```bash
$ head -5 /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync

# Explanation:
# Format: username:password:UID:GID:comment:home:shell
# x means password is in /etc/shadow
# Shows system users and their configurations
```

**Solution 9**:
```bash
$ ls -l /etc | grep ^d | wc -l
```

**Solution 10**:
```bash
$ cat > notes.txt
This is my first note.
Learning Linux is fun!
^D (Press Ctrl+D)

$ cat notes.txt
This is my first note.
Learning Linux is fun!
```

### Advanced Solutions

**Solution 11**:
```bash
$ mkdir Documents Images Archives
$ mv *.doc *.txt *.pdf Documents/ 2>/dev/null
$ mv *.jpg *.png *.gif Images/ 2>/dev/null
$ mv *.zip *.tar.gz Archives/ 2>/dev/null
```

**Solution 12**:
```bash
$ ls -lt  # Sorted by modification time (newest first)
$ ls -lS  # Sorted by size (largest first)

# Difference:
# -t: Time-based sorting (when files were modified)
# -S: Size-based sorting (file sizes)
```

**Solution 13**:
```bash
$ ls -lhS ~ | head -6
# or better:
$ du -ah ~ 2>/dev/null | sort -rh | head -5
```

**Solution 14**:
```bash
$ mkdir -p project/{src,tests,docs,config} && touch project/README.md
$ tree project  # or ls -R project
```

**Solution 15**:
```bash
$ ls -l ../../../
# or
$ ls -l $(pwd | cut -d/ -f1-3)
```

---

## 🎓 Learning Tips

### Mnemonics
- **pwd** = "Present Working Directory" or "Print Where (I) Definitely (am)"
- **ls** = "LiSt"
- **cd** = "Change Directory"
- **mkdir** = "MaKe DIRectory"
- **cp** = "CoPy"
- **mv** = "MoVe"
- **rm** = "ReMove"

### Common Mistakes to Avoid

1. **Forgetting the path type**:
   ```bash
   $ cd Documents      # Relative - works if Documents is in current dir
   $ cd /Documents     # Absolute - looks for Documents in root!
   ```

2. **Not verifying before rm**:
   ```bash
   $ rm -rf *          # DANGER! Always check pwd first
   $ pwd; ls           # Safe: check before removing
   $ rm -rf unwanted_folder/
   ```

3. **Overwriting files with cp/mv**:
   ```bash
   $ cp -i file.txt existing.txt  # -i asks before overwriting
   ```

4. **Confusing cp and mv**:
   - **cp**: Creates copy, source remains
   - **mv**: Moves file, source is gone

### Keyboard Shortcuts
- `Ctrl + A`: Move to beginning of line
- `Ctrl + E`: Move to end of line
- `Ctrl + U`: Clear line before cursor
- `Ctrl + K`: Clear line after cursor
- `Ctrl + L`: Clear screen (same as `clear`)
- `Tab`: Auto-complete file/directory names
- `Tab Tab`: Show all possibilities

---

## 🔗 Connections to Future Chapters

- **Chapter 3**: Understanding the directory structure in detail
- **Chapter 4**: File permissions affect what you can do with files
- **Chapter 5**: Text processing builds on cat, less, head, tail
- **Chapter 6**: Redirection uses output from these commands
- **Chapter 11**: Shell scripts automate these operations

---

## 📚 Additional Resources

### Commands to Explore
```bash
man hier    # Filesystem hierarchy description
man bash    # Bash reference manual
info coreutils  # GNU core utilities info
```

### Practice More
- Create a fake project structure
- Organize your Downloads folder
- Practice with temporary directory:
  ```bash
  $ mkdir ~/temp_practice
  $ cd ~/temp_practice
  # Practice freely here
  $ rm -rf ~/temp_practice  # Clean up when done
  ```

### Useful Websites
- [explainshell.com](https://explainshell.com/) - Explains shell commands
- [The Linux Command Line](http://linuxcommand.org/)
- [Learn Linux TV](https://www.learnlinux.tv/) - Video tutorials

---

## ✅ Self-Assessment Checklist

Before moving to Chapter 3, ensure you can:
- [ ] Use pwd to find your current location
- [ ] Navigate using cd with both absolute and relative paths
- [ ] List files with various ls options
- [ ] Understand the difference between . and .. and ~
- [ ] Create directories with mkdir and mkdir -p
- [ ] Create files with touch
- [ ] Copy files and directories correctly
- [ ] Move and rename files safely
- [ ] Remove files and directories (carefully!)
- [ ] View file contents with cat, less, head, tail
- [ ] Count lines, words, characters with wc
- [ ] Find help using man, --help, and apropos

---

## 🚀 Next Steps

Excellent progress! You now have the fundamental skills to navigate and manipulate the Linux filesystem.

In **Chapter 3**, you'll learn:
- The Linux directory structure (/home, /etc, /var, etc.)
- File types and how Linux organizes files
- File permissions and what they mean
- The concept of inodes and file metadata

**Challenge**: Before moving on, spend a day organizing your actual home directory using only the command line. Create a sensible folder structure for Documents, Projects, Downloads, etc.

---

*"The command line is not about typing more—it's about typing smarter."*