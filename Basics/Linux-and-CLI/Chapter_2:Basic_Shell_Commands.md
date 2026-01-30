# Chapter 2: Basic Shell Commands

## Overview
This chapter covers the fundamental commands you need to navigate and interact with a Linux system. These are the building blocks that you'll use every day, and mastering them is essential for everything that follows.

---

## 2.1 Navigation Commands

### pwd - Print Working Directory

**Purpose:** Shows your current location in the file system

**Syntax:**
```bash
pwd [OPTIONS]
```

**How It Works:**
Every time you're in a terminal, you're "inside" a directory. The `pwd` command tells you exactly where you are in the directory tree.

**Example:**
```bash
$ pwd
/home/john/Documents
```

**Common Options:**
```bash
pwd -L    # Show logical path (follow symlinks)
pwd -P    # Show physical path (resolve symlinks)
```

**Real-World Use:**
- Confirm your location before running commands
- Use in scripts to determine the working directory
- Understand context when copying/moving files

**Analogy:** Like checking your location on a map before giving directions.

---

### ls - List Directory Contents

**Purpose:** Display files and directories in the current (or specified) location

**Basic Syntax:**
```bash
ls [OPTIONS] [DIRECTORY]
```

**Simple Examples:**
```bash
ls                  # List current directory
ls /home           # List /home directory
ls Documents       # List Documents subdirectory
ls file.txt        # Show info about specific file
```

**Essential Options:**

**1. Long Format (-l)**
```bash
ls -l
```
Output:
```
-rw-r--r-- 1 john users 1234 Jan 30 10:00 file.txt
│││││││││  │ │    │     │    │          └─ filename
│││││││││  │ │    │     │    └─ modification date/time
│││││││││  │ │    │     └─ file size (bytes)
│││││││││  │ │    └─ group owner
│││││││││  │ └─ user owner
│││││││││  └─ number of hard links
││││││││└─ others permissions (read, read)
│││││││└─ group permissions (read, read)
││││││└─ owner permissions (read, write)
│││││└─ special permissions
││││└─ execute permission
│││└─ write permission
││└─ read permission
│└─ file type (- = regular file, d = directory, l = link)
```

**2. Show Hidden Files (-a)**
```bash
ls -a              # Show all files (including hidden)
ls -A              # Show all except . and ..
```
Hidden files start with a dot (`.bashrc`, `.config`)

**3. Human-Readable Sizes (-h)**
```bash
ls -lh
# Shows: 1.2K, 3.4M, 2.1G instead of byte counts
```

**4. Sort by Time (-t)**
```bash
ls -lt             # Newest first
ls -ltr            # Oldest first (reverse)
```

**5. Recursive (-R)**
```bash
ls -R              # Show subdirectories and their contents
```

**6. Sort by Size (-S)**
```bash
ls -lS             # Largest first
ls -lSr            # Smallest first
```

**Combining Options:**
```bash
ls -lah            # Long format, all files, human-readable
ls -ltr            # Long format, sorted by time, reversed
ls -laRh           # All files, recursive, long format, human-readable
```

**Practical Examples:**
```bash
# Find largest files
ls -lhS

# See recently modified files
ls -lt | head

# Count files in directory
ls -1 | wc -l

# Show only directories
ls -d */
```

**Common Mistakes:**
```bash
ls -l file.txt     # ✓ Shows info about file.txt
ls -l file*        # ✓ Shows all files starting with "file"
ls -l *.txt        # ✓ Shows all .txt files
ls file.txt        # ✓ Shows just the name (not much info)
```

---

### cd - Change Directory

**Purpose:** Move to a different directory

**Syntax:**
```bash
cd [DIRECTORY]
```

**Basic Usage:**
```bash
cd /home/john      # Go to /home/john
cd Documents       # Go to Documents in current directory
cd ..              # Go up one level (parent directory)
cd ~               # Go to home directory
cd -               # Go to previous directory
cd                 # Go to home directory (same as cd ~)
```

**Detailed Examples:**

**1. Absolute Paths** (start with /)
```bash
cd /usr/local/bin
cd /home/john/Documents
cd /
```
Always works the same regardless of current location

**2. Relative Paths** (relative to current directory)
```bash
# If you're in /home/john:
cd Documents              # Goes to /home/john/Documents
cd Documents/Work         # Goes to /home/john/Documents/Work
cd ./Documents            # Same as above (./ is current dir)
```

**3. Parent Directory (..)**
```bash
cd ..                     # Go up one level
cd ../..                  # Go up two levels
cd ../../etc              # Go up two levels, then to /etc
```

**4. Home Directory Shortcuts**
```bash
cd ~                      # Your home directory
cd                        # Same as above
cd ~john                  # John's home directory
cd ~/Documents            # Documents in your home
```

**5. Previous Directory (-)**
```bash
cd /var/log
cd /etc
cd -                      # Back to /var/log
cd -                      # Back to /etc (toggles)
```

**Pro Tips:**
```bash
# Quick navigation
cd -                      # Toggle between two directories
pushd /etc                # Save current dir and go to /etc
popd                      # Return to saved directory

# Tab completion
cd Doc<TAB>               # Auto-completes to Documents
cd /usr/loc<TAB>          # Auto-completes to /usr/local
```

**Common Errors:**
```bash
cd file.txt               # ✗ Can't cd into a file
cd Documents/             # ✓ Works (trailing slash optional)
cd "My Documents"         # ✓ Use quotes for spaces
cd My\ Documents          # ✓ Or escape spaces
```

---

### Understanding Paths

**Absolute vs Relative Paths**

**Absolute Path:**
- Starts with `/` (root)
- Full path from root of filesystem
- Works from any location
- Example: `/home/john/Documents/file.txt`

**Relative Path:**
- Starts from current directory
- Depends on where you are
- Shorter and more convenient
- Example: `Documents/file.txt` (if in /home/john)

**Path Examples:**
```bash
# Absolute paths:
/home/john/file.txt
/usr/local/bin/script.sh
/etc/hosts

# Relative paths (from /home/john):
Documents/file.txt
../jane/file.txt
./script.sh
```

---

### Special Directories

**1. . (Current Directory)**
```bash
ls .                      # Same as: ls
cd .                      # Stay in current directory
./script.sh               # Run script in current directory
```

**2. .. (Parent Directory)**
```bash
cd ..                     # Go up one level
ls ..                     # List parent directory
cp file.txt ../           # Copy to parent directory
```

**3. ~ (Home Directory)**
```bash
cd ~                      # Go home
ls ~/Documents            # List Documents in home
cp file.txt ~             # Copy to home directory
```

**4. / (Root Directory)**
```bash
cd /                      # Go to root
ls /                      # List root directory
```

**5. - (Previous Directory)**
```bash
cd -                      # Toggle to previous directory
```

**Visual Example of Directory Tree:**
```
/                          (root)
├── home/
│   ├── john/              (~ when logged in as john)
│   │   ├── Documents/     (current directory in examples)
│   │   │   ├── file.txt
│   │   │   └── Work/
│   │   ├── Downloads/
│   │   └── .bashrc        (hidden file)
│   └── jane/
├── etc/
├── usr/
└── var/
```

---

## 2.2 File and Directory Operations

### mkdir - Make Directory

**Purpose:** Create new directories

**Basic Syntax:**
```bash
mkdir [OPTIONS] DIRECTORY_NAME
```

**Simple Examples:**
```bash
mkdir mydir                    # Create single directory
mkdir dir1 dir2 dir3          # Create multiple directories
```

**Important Options:**

**1. Create Parent Directories (-p)**
```bash
mkdir -p Documents/Work/Projects/2026
# Creates all directories in path if they don't exist
```

Without `-p`:
```bash
mkdir Documents/Work/New
# Error if Documents or Work don't exist
```

**2. Set Permissions on Creation (-m)**
```bash
mkdir -m 755 mydir            # Create with specific permissions
mkdir -m 700 private          # Create private directory (owner only)
```

**3. Verbose Output (-v)**
```bash
mkdir -v newdir
# Output: mkdir: created directory 'newdir'
```

**Practical Examples:**
```bash
# Project structure
mkdir -p project/{src,bin,docs,tests}

# Dated directories
mkdir -p backups/$(date +%Y/%m/%d)

# Multiple directories with common parent
mkdir -p ~/Documents/{Personal,Work,School}
```

---

### touch - Create Empty Files

**Purpose:** Create new empty files or update timestamps

**Basic Syntax:**
```bash
touch [OPTIONS] FILENAME
```

**Common Uses:**
```bash
touch file.txt                 # Create empty file
touch file1.txt file2.txt     # Create multiple files
touch .hidden                 # Create hidden file
```

**Advanced Options:**

**1. Set Specific Time (-t)**
```bash
touch -t 202601301200 file.txt    # YYYYMMDDhhmm format
```

**2. Use Another File's Time (-r)**
```bash
touch -r reference.txt newfile.txt
```

**3. Don't Create if Doesn't Exist (-c)**
```bash
touch -c file.txt              # Only update if exists
```

**Practical Uses:**
```bash
# Create test files
touch test{1..10}.txt          # Creates test1.txt to test10.txt

# Update modification time
touch existing_file.txt

# Create placeholder files
touch README.md LICENSE .gitignore
```

---

### cp - Copy Files and Directories

**Purpose:** Copy files or directories from one location to another

**Basic Syntax:**
```bash
cp [OPTIONS] SOURCE DESTINATION
```

**Simple Examples:**
```bash
cp file.txt copy.txt           # Copy file
cp file.txt ~/Documents/       # Copy to directory
cp file1.txt file2.txt dir/    # Copy multiple files to directory
```

**Essential Options:**

**1. Recursive (-r or -R)**
```bash
cp -r directory/ backup/       # Copy entire directory
```

**2. Preserve Attributes (-p)**
```bash
cp -p file.txt copy.txt        # Keep permissions, ownership, timestamps
```

**3. Interactive (-i)**
```bash
cp -i file.txt existing.txt    # Prompt before overwriting
```

**4. Verbose (-v)**
```bash
cp -v file.txt copy.txt
# Output: 'file.txt' -> 'copy.txt'
```

**5. Archive Mode (-a)**
```bash
cp -a directory/ backup/       # Preserve everything, recursive
# Equivalent to: cp -dpR
```

**6. Force (-f)**
```bash
cp -f file.txt protected.txt   # Force overwrite
```

**7. Update (-u)**
```bash
cp -u source.txt dest.txt      # Copy only if source is newer
```

**Practical Examples:**
```bash
# Backup with timestamp
cp file.txt file_$(date +%Y%m%d).txt

# Copy preserving structure
cp -a /source/* /destination/

# Safe copying
cp -i *.txt backup/

# Copy with progress (using rsync)
rsync -ah --progress source dest
```

**Common Patterns:**
```bash
cp *.txt backup/               # Copy all .txt files
cp -r dir1/* dir2/             # Copy contents (not dir itself)
cp -r dir1 dir2                # Copy directory into dir2
```

---

### mv - Move and Rename

**Purpose:** Move files/directories or rename them

**Basic Syntax:**
```bash
mv [OPTIONS] SOURCE DESTINATION
```

**Two Main Uses:**

**1. Renaming:**
```bash
mv oldname.txt newname.txt
mv old_dir new_dir
```

**2. Moving:**
```bash
mv file.txt /path/to/destination/
mv file.txt ../                # Move to parent directory
mv *.txt Documents/            # Move all .txt files
```

**Important Options:**

**1. Interactive (-i)**
```bash
mv -i file.txt existing.txt    # Prompt before overwriting
```

**2. No-Clobber (-n)**
```bash
mv -n file.txt existing.txt    # Never overwrite
```

**3. Force (-f)**
```bash
mv -f file.txt protected.txt   # Force overwrite without prompting
```

**4. Verbose (-v)**
```bash
mv -v old.txt new.txt
# Output: renamed 'old.txt' -> 'new.txt'
```

**5. Update (-u)**
```bash
mv -u source.txt dest.txt      # Move only if source is newer
```

**6. Backup (-b)**
```bash
mv -b file.txt existing.txt    # Create backup of destination
```

**Practical Examples:**
```bash
# Bulk rename with pattern
for file in *.txt; do
    mv "$file" "${file%.txt}.backup"
done

# Move files modified today
find . -name "*.log" -mtime -1 -exec mv {} archive/ \;

# Rename with date
mv report.pdf report_$(date +%Y%m%d).pdf

# Safe move
mv -i *.doc Documents/
```

**Common Patterns:**
```bash
mv file.txt dir/               # Move file into directory
mv dir1 dir2/                  # Move dir1 into dir2
mv dir1/* dir2/                # Move contents of dir1 to dir2
```

---

### rm - Remove Files and Directories

**Purpose:** Delete files and directories

**⚠️ WARNING: rm is dangerous! Deleted files cannot be easily recovered.**

**Basic Syntax:**
```bash
rm [OPTIONS] FILE...
```

**Simple Examples:**
```bash
rm file.txt                    # Delete single file
rm file1.txt file2.txt        # Delete multiple files
rm *.tmp                      # Delete all .tmp files
```

**Essential Options:**

**1. Interactive (-i)**
```bash
rm -i file.txt                 # Prompt before deletion
```

**2. Recursive (-r or -R)**
```bash
rm -r directory/               # Delete directory and contents
```

**3. Force (-f)**
```bash
rm -f file.txt                 # Force delete, no prompts
```

**4. Verbose (-v)**
```bash
rm -v file.txt
# Output: removed 'file.txt'
```

**Dangerous Combinations (BE CAREFUL!):**
```bash
rm -rf directory/              # Force delete directory recursively
```

**Safe Practices:**
```bash
# 1. Always use -i when unsure
rm -i *.txt

# 2. Check what you'll delete first
ls *.log                       # Verify
rm *.log                       # Then delete

# 3. Use trash instead
trash file.txt                 # Sends to trash (if installed)

# 4. Be specific with paths
rm ./file.txt                  # Not rm file.txt
```

**Common Mistakes to AVOID:**
```bash
# DANGEROUS - DO NOT RUN:
rm -rf /                       # Would try to delete everything (protected)
rm -rf /*                      # Same, extremely dangerous
rm -rf ./*                     # Deletes everything in current dir

# CAREFUL:
rm *.txt                       # Make sure you're in right directory!
rm -rf directory               # Double-check directory name
```

**Practical Examples:**
```bash
# Delete files older than 30 days
find . -name "*.log" -mtime +30 -delete

# Delete empty files
find . -type f -empty -delete

# Delete all but newest file
ls -t | tail -n +2 | xargs rm

# Safe bulk delete
find . -name "*.tmp" -exec rm -i {} \;
```

---

### rmdir - Remove Empty Directories

**Purpose:** Remove empty directories only

**Basic Syntax:**
```bash
rmdir [OPTIONS] DIRECTORY...
```

**Examples:**
```bash
rmdir empty_dir                # Remove single empty directory
rmdir dir1 dir2 dir3          # Remove multiple empty directories
```

**Important Options:**

**1. Parents (-p)**
```bash
rmdir -p dir1/dir2/dir3        # Remove dir3, then dir2, then dir1 if empty
```

**2. Verbose (-v)**
```bash
rmdir -v empty_dir
# Output: rmdir: removing directory, 'empty_dir'
```

**Key Difference from rm:**
```bash
rmdir directory/               # Fails if not empty (safe!)
rm -r directory/               # Deletes even if not empty (dangerous!)
```

**Why Use rmdir Instead of rm -r:**
- Safety: Won't accidentally delete non-empty directories
- Intentionality: Forces you to empty directories first
- Good practice for scripts

**Example Workflow:**
```bash
# Safe cleanup
rm directory/*                 # Remove files first
rmdir directory/               # Then remove directory
```

---

## 2.3 Viewing File Contents

### cat - Concatenate and Display

**Purpose:** Display file contents, concatenate files

**Basic Syntax:**
```bash
cat [OPTIONS] [FILE...]
```

**Simple Uses:**
```bash
cat file.txt                   # Display file contents
cat file1.txt file2.txt       # Display multiple files
cat file1.txt file2.txt > combined.txt  # Concatenate into new file
```

**Useful Options:**

**1. Number Lines (-n)**
```bash
cat -n file.txt
# 1  First line
# 2  Second line
```

**2. Number Non-Empty Lines (-b)**
```bash
cat -b file.txt
```

**3. Show Tabs as ^I (-T)**
```bash
cat -T file.txt
```

**4. Show End of Lines (-E)**
```bash
cat -E file.txt
# Each line ends with $
```

**5. All Special Characters (-A)**
```bash
cat -A file.txt
# Equivalent to -vET
```

**6. Squeeze Blank Lines (-s)**
```bash
cat -s file.txt                # Multiple blank lines become one
```

**Practical Examples:**
```bash
# Create quick file
cat > newfile.txt
Type content here
Press Ctrl+D when done

# Append to file
cat >> existing.txt
More content
Ctrl+D

# Display with line numbers
cat -n script.sh

# Combine files
cat part1.txt part2.txt part3.txt > complete.txt

# Display multiple files with filenames
cat file1.txt file2.txt | less
```

**When NOT to Use cat:**
```bash
cat large_file.txt             # ✗ Use less for large files
cat binary_file                # ✗ Will mess up terminal
```

---

### less and more - Page Through Files

**Purpose:** View files page by page

**less - Modern Pager (Recommended)**

**Basic Usage:**
```bash
less filename.txt
```

**Navigation in less:**
```
Space or f       # Forward one page
b                # Backward one page
Down/j           # Forward one line
Up/k             # Backward one line
g                # Go to beginning
G                # Go to end
/pattern         # Search forward
?pattern         # Search backward
n                # Next search result
N                # Previous search result
q                # Quit
h                # Help
```

**Useful less Options:**
```bash
less -N file.txt               # Show line numbers
less -S file.txt               # Don't wrap long lines
less -I file.txt               # Case-insensitive search
less +F file.txt               # Follow mode (like tail -f)
```

**more - Older Pager**
```bash
more filename.txt
```

**Navigation in more:**
```
Space            # Forward one page
Enter            # Forward one line
b                # Backward one page (may not work)
/pattern         # Search
q                # Quit
```

**Why less is Better:**
- Can scroll backward
- Faster for large files
- Better search capabilities
- Doesn't leave text in terminal after quitting

**Practical Examples:**
```bash
# View large log files
less /var/log/syslog

# Follow log file in real-time
less +F /var/log/syslog        # Press Ctrl+C to stop following

# View with line numbers
less -N script.py

# View command output
ls -la | less
ps aux | less
```

---

### head and tail - View Beginning or End

**head - View First Lines**

**Basic Syntax:**
```bash
head [OPTIONS] [FILE...]
```

**Default:** Shows first 10 lines
```bash
head file.txt                  # First 10 lines
```

**Specify Number of Lines (-n)**
```bash
head -n 20 file.txt            # First 20 lines
head -20 file.txt              # Same (shorthand)
head -n -5 file.txt            # All except last 5 lines
```

**Bytes Instead of Lines (-c)**
```bash
head -c 100 file.txt           # First 100 bytes
```

**Multiple Files:**
```bash
head -n 5 file1.txt file2.txt
==> file1.txt <==
...
==> file2.txt <==
...
```

**tail - View Last Lines**

**Basic Syntax:**
```bash
tail [OPTIONS] [FILE...]
```

**Default:** Shows last 10 lines
```bash
tail file.txt                  # Last 10 lines
```

**Specify Number of Lines (-n)**
```bash
tail -n 20 file.txt            # Last 20 lines
tail -20 file.txt              # Same (shorthand)
tail -n +5 file.txt            # From line 5 to end
```

**Follow Mode (-f) - VERY USEFUL**
```bash
tail -f /var/log/syslog        # Watch file in real-time
tail -f logfile.log            # Monitor log files
# Press Ctrl+C to stop
```

**Follow with Retry (-F)**
```bash
tail -F logfile.log            # Retry if file is rotated
```

**Practical Examples:**
```bash
# View last log entries
tail -n 50 /var/log/syslog

# Monitor multiple logs
tail -f /var/log/syslog /var/log/auth.log

# Monitor and highlight errors
tail -f app.log | grep --color error

# Show last 100 lines, then follow
tail -n 100 -f logfile.log

# View specific range of lines
head -n 50 file.txt | tail -n 10    # Lines 41-50
```

**Combine head and tail:**
```bash
# Get lines 20-30
head -n 30 file.txt | tail -n 10

# Sample file
head -n 1000 huge_file.txt > sample.txt
```

---

### wc - Word Count

**Purpose:** Count lines, words, and characters

**Basic Syntax:**
```bash
wc [OPTIONS] [FILE...]
```

**Default Output:**
```bash
wc file.txt
# Output: 10  50  300 file.txt
#         │   │   │   └─ filename
#         │   │   └─ bytes
#         │   └─ words
#         └─ lines
```

**Specific Counts:**
```bash
wc -l file.txt                 # Lines only
wc -w file.txt                 # Words only
wc -c file.txt                 # Bytes only
wc -m file.txt                 # Characters (UTF-8 aware)
wc -L file.txt                 # Longest line length
```

**Practical Examples:**
```bash
# Count files in directory
ls | wc -l

# Count lines of code
find . -name "*.py" | xargs wc -l

# Count unique words
cat file.txt | tr ' ' '\n' | sort | uniq | wc -l

# Count non-empty lines
grep -v '^$' file.txt | wc -l

# Total lines in all .txt files
wc -l *.txt

# Count users in system
wc -l /etc/passwd
```

**Combine with Other Commands:**
```bash
# Count running processes
ps aux | wc -l

# Count errors in log
grep ERROR logfile.log | wc -l

# Size of all files
ls -l | awk '{sum += $5} END {print sum}'
```

---

## 2.4 Getting Help

### man - Manual Pages

**Purpose:** Access comprehensive command documentation

**Basic Syntax:**
```bash
man COMMAND
```

**Examples:**
```bash
man ls                         # Manual for ls
man man                        # Manual for man itself
man 5 passwd                   # Section 5 of passwd manual
```

**Manual Sections:**
1. User commands
2. System calls
3. Library functions
4. Special files
5. File formats
6. Games
7. Miscellaneous
8. System administration commands
9. Kernel routines

**Navigation in man:**
```
Space/f          # Next page
b                # Previous page
/pattern         # Search forward
?pattern         # Search backward
n                # Next search result
q                # Quit
h                # Help
```

**Useful man Options:**
```bash
man -k keyword                 # Search all man pages
man -f command                 # Short description
man -a ls                      # Show all matching pages
```

**Man Page Structure:**
```
NAME           # Command name and brief description
SYNOPSIS       # How to use the command
DESCRIPTION    # Detailed explanation
OPTIONS        # Available flags and options
EXAMPLES       # Usage examples
SEE ALSO       # Related commands
BUGS           # Known issues
AUTHOR         # Who wrote it
```

---

### --help Flag

**Purpose:** Quick reference for command options

**Usage:**
```bash
command --help
command -h                     # Some commands use -h
```

**Examples:**
```bash
ls --help
mkdir --help
grep --help
```

**Advantages:**
- Faster than man pages
- Concise option list
- Immediately shows common usage
- Doesn't leave terminal

**When to Use:**
- Quick reminder of options
- Check if command accepts an option
- See basic syntax
- Verify command is installed

---

### info Command

**Purpose:** More detailed documentation than man

**Usage:**
```bash
info command
```

**Examples:**
```bash
info ls
info coreutils               # All core utilities
```

**Navigation in info:**
```
n                # Next node
p                # Previous node
u                # Up to parent
l                # Last visited
Space            # Next page
Backspace        # Previous page
q                # Quit
```

**Note:** Not all commands have info pages

---

### apropos - Search Manual Pages

**Purpose:** Find commands by keyword

**Syntax:**
```bash
apropos KEYWORD
```

**Examples:**
```bash
apropos copy                   # Find copy-related commands
apropos "copy files"           # Multi-word search
apropos -a user admin          # Multiple keywords (AND)
```

**Equivalent:**
```bash
man -k copy                    # Same as apropos
```

**Update apropos Database:**
```bash
sudo mandb                     # Rebuild man page database
```

---

### whatis - Brief Description

**Purpose:** One-line description of a command

**Syntax:**
```bash
whatis COMMAND
```

**Examples:**
```bash
whatis ls
# Output: ls (1) - list directory contents

whatis cp mv rm
# Shows descriptions for all three
```

**Equivalent:**
```bash
man -f ls                      # Same as whatis
```

---

## Practice Exercises

### Exercise Set 1: Navigation
1. Find your current directory
2. Go to your home directory
3. Create a directory called `practice`
4. Navigate into it
5. Go back to home
6. Navigate to `/etc` and list its contents
7. Return to your home directory using `cd -`

### Exercise Set 2: File Operations
1. Create a directory structure: `projects/web/css`
2. Create three empty files in the `css` directory
3. Copy one file to the parent directory
4. Move another file to the `web` directory
5. Rename the third file
6. Delete the `css` directory (including contents)

### Exercise Set 3: Viewing Files
1. View `/etc/passwd` using cat
2. View the same file using less
3. See the first 5 users in the system
4. See the last 3 users in the system
5. Count total number of users

### Exercise Set 4: Getting Help
1. Read the manual for the `ls` command
2. Find all commands related to "file"
3. Get a brief description of the `mkdir` command
4. Use `ls --help` to find the option for human-readable sizes

---

## Key Takeaways

1. **pwd** shows where you are
2. **ls** shows what's around you
3. **cd** moves you around
4. **mkdir/touch** creates directories/files
5. **cp** copies, **mv** moves/renames, **rm** deletes (carefully!)
6. **cat/less/head/tail** let you view files
7. **man/--help** are your best friends for learning

---

## Common Beginner Mistakes

1. **Not checking location before running commands**
   - Always run `pwd` if unsure
   
2. **Using rm -rf without thinking**
   - Double-check what you're deleting
   
3. **Forgetting to use tab completion**
   - Press TAB to autocomplete (saves time and errors)

4. **Not reading error messages**
   - They usually tell you exactly what's wrong

5. **Ignoring man pages**
   - `man command` will answer most questions

---

## Next Steps

Chapter 3 will dive deep into the Linux file system structure, teaching you where everything is located and why. You'll learn about special directories like `/etc`, `/var`, `/usr`, and understand file permissions in detail.