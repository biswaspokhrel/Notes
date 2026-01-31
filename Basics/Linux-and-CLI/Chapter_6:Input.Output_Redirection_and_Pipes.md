# Chapter 6: Input/Output Redirection and Pipes

## 🎯 Learning Objectives
By the end of this chapter, you will:
- Understand standard streams (stdin, stdout, stderr)
- Redirect output to files and other commands
- Redirect input from files
- Combine multiple commands with pipes
- Use advanced redirection techniques
- Master tee and xargs commands
- Build powerful command pipelines
- Handle errors and output separately

---

## 6.1 Understanding Standard Streams

### The Three Standard Streams

Every Linux program has three data streams automatically opened:

| Stream | Name | File Descriptor | Default | Purpose |
|--------|------|-----------------|---------|---------|
| stdin | Standard Input | 0 | Keyboard | Input data |
| stdout | Standard Output | 1 | Screen | Normal output |
| stderr | Standard Error | 2 | Screen | Error messages |

**Visualization:**
```
         ┌─────────┐
stdin  → │ Program │ → stdout
         │         │ → stderr
         └─────────┘
```

**Example:**
```bash
$ cat file.txt
# stdin: (none - reading from file)
# stdout: file contents displayed
# stderr: (none if successful)

$ cat nonexistent.txt
# stdin: (none)
# stdout: (none)
# stderr: cat: nonexistent.txt: No such file or directory
```

### File Descriptors

**File descriptors are integers that represent open files:**
- **0** = stdin (standard input)
- **1** = stdout (standard output)
- **2** = stderr (standard error)

```bash
# These are equivalent:
$ command > output.txt
$ command 1> output.txt    # Explicit file descriptor

$ command 2> errors.txt    # Redirect stderr
```

---

## 6.2 Output Redirection

### Redirect Standard Output (>)

**Overwrite file:**
```bash
# Redirect stdout to file (creates or overwrites)
$ echo "Hello World" > output.txt
$ cat output.txt
Hello World

# Command output to file
$ ls -l > filelist.txt
$ cat filelist.txt
total 8
-rw-r--r-- 1 john john 12 Jan 31 10:00 output.txt

# Save command results
$ date > timestamp.txt
$ cat timestamp.txt
Fri Jan 31 10:00:00 NPT 2026
```

**Important:** Using `>` **overwrites** the file!
```bash
$ echo "First line" > file.txt
$ cat file.txt
First line

$ echo "Second line" > file.txt    # OVERWRITES!
$ cat file.txt
Second line    # First line is gone!
```

---

### Append to File (>>)

**Append instead of overwrite:**
```bash
$ echo "First line" > file.txt
$ echo "Second line" >> file.txt    # APPENDS
$ echo "Third line" >> file.txt     # APPENDS
$ cat file.txt
First line
Second line
Third line

# Append command output
$ date >> logfile.txt
$ uptime >> logfile.txt
$ cat logfile.txt
Fri Jan 31 10:00:00 NPT 2026
10:00:00 up 5 days, 2:30, 1 user, load average: 0.15, 0.20, 0.18
```

**Use Cases:**
```bash
# Build a log file
$ echo "Starting backup..." >> backup.log
$ tar -czf backup.tar.gz /data >> backup.log 2>&1
$ echo "Backup completed" >> backup.log

# Collect system info
$ echo "=== System Info ===" > sysinfo.txt
$ hostname >> sysinfo.txt
$ uname -a >> sysinfo.txt
$ df -h >> sysinfo.txt
```

---

### Redirect Standard Error (2>)

**Redirect error messages:**
```bash
# Redirect errors to file
$ ls /nonexistent 2> errors.txt
$ cat errors.txt
ls: cannot access '/nonexistent': No such file or directory

# Suppress errors (send to /dev/null)
$ ls /nonexistent 2> /dev/null
# No output - errors discarded

# Save errors for debugging
$ find / -name "*.conf" 2> find_errors.txt
# Normal output to screen, errors to file
```

---

### Redirect Both stdout and stderr

**Method 1: Redirect separately**
```bash
$ command > output.txt 2> errors.txt
# stdout → output.txt
# stderr → errors.txt
```

**Method 2: Redirect both to same file**
```bash
# Old syntax (still works)
$ command > output.txt 2>&1
# stdout → output.txt
# stderr → same as stdout (output.txt)

# New syntax (bash 4+)
$ command &> output.txt
# Both stdout and stderr → output.txt

# Append both
$ command >> output.txt 2>&1
# or
$ command &>> output.txt
```

**Understanding 2>&1:**
```
2>&1 means: "redirect file descriptor 2 (stderr) to wherever 
            file descriptor 1 (stdout) is going"

Order matters!
$ command > file.txt 2>&1     # CORRECT
  1. stdout goes to file.txt
  2. stderr goes to where stdout is (file.txt)

$ command 2>&1 > file.txt     # WRONG!
  1. stderr goes to where stdout is (screen)
  2. stdout goes to file.txt
  3. stderr still goes to screen!
```

**Examples:**
```bash
# Capture everything
$ make &> build.log
$ cat build.log
# Contains both normal output and errors

# Silent execution
$ command &> /dev/null
# Completely silent - all output discarded

# Separate streams
$ gcc program.c > compile_output.txt 2> compile_errors.txt
# Normal output: compile_output.txt
# Errors: compile_errors.txt

# Log everything
$ ./script.sh >> script.log 2>&1
# Appends both stdout and stderr to log
```

---

### The Black Hole: /dev/null

**/dev/null** is a special file that discards everything written to it.

```bash
# Discard all output
$ command > /dev/null

# Discard errors
$ command 2> /dev/null

# Discard everything
$ command &> /dev/null
# or
$ command > /dev/null 2>&1

# Use in scripts
$ if grep -q "pattern" file.txt &> /dev/null; then
    echo "Pattern found"
fi
```

**Use Cases:**
```bash
# Check if command succeeds (ignore output)
$ ping -c 1 google.com &> /dev/null && echo "Internet works"

# Suppress warning messages
$ apt-get update 2> /dev/null

# Silent cron jobs
$ script.sh > /dev/null 2>&1

# Test file existence without output
$ ls file.txt > /dev/null 2>&1
```

---

## 6.3 Input Redirection

### Redirect Input (<)

**Read from file instead of keyboard:**
```bash
# Standard input from keyboard
$ cat
Type something
^D

# Input from file
$ cat < file.txt
# Displays file.txt contents

# Count lines in file
$ wc -l < file.txt
5

# Sort file contents
$ sort < unsorted.txt
apple
banana
cherry
```

**Difference: < vs filename as argument**
```bash
# These are usually equivalent:
$ cat file.txt      # File as argument
$ cat < file.txt    # File as stdin

# But sometimes behave differently:
$ wc -l file.txt
5 file.txt          # Shows filename

$ wc -l < file.txt
5                   # No filename (reading from stdin)
```

---

### Here Documents (<<)

**Provide multi-line input:**
```bash
# Basic syntax
$ command << DELIMITER
line 1
line 2
line 3
DELIMITER

# Example with cat
$ cat << EOF
This is line 1
This is line 2
This is line 3
EOF
```

**Common Uses:**
```bash
# Create file with here document
$ cat > config.txt << EOF
server=192.168.1.1
port=8080
timeout=30
EOF

$ cat config.txt
server=192.168.1.1
port=8080
timeout=30

# Send email (with mail command)
$ mail user@example.com << EOF
Subject: System Alert

The backup completed successfully.
EOF

# Generate SQL script
$ mysql -u root -p database << EOF
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);
INSERT INTO users VALUES (1, 'John');
EOF

# Create script
$ cat > script.sh << 'EOF'
#!/bin/bash
echo "Hello, World!"
date
EOF
$ chmod +x script.sh
```

**Variables in Here Documents:**
```bash
# Variables are expanded
$ name="John"
$ cat << EOF
Hello, $name
Today is $(date +%A)
EOF
Hello, John
Today is Friday

# Prevent expansion with quoted delimiter
$ cat << 'EOF'
Hello, $name
Today is $(date +%A)
EOF
Hello, $name
Today is $(date +%A)
```

---

### Here Strings (<<<)

**Provide single-line input:**
```bash
# Basic syntax
$ command <<< "string"

# Count words in string
$ wc -w <<< "Hello World from Linux"
4

# Convert to uppercase
$ tr 'a-z' 'A-Z' <<< "hello world"
HELLO WORLD

# Read variable
$ text="Multiple words here"
$ wc -w <<< "$text"
3

# With grep
$ grep "pattern" <<< "This is a pattern test"
This is a pattern test
```

**Use Cases:**
```bash
# Quick testing
$ bc <<< "2+2"
4

$ bc <<< "scale=2; 10/3"
3.33

# Process variables
$ url="https://example.com/path"
$ cut -d'/' -f3 <<< "$url"
example.com

# One-liner input to command
$ md5sum <<< "text to hash"
```

---

## 6.4 Pipes and Filters

### The Pipe Operator (|)

**Pipe connects stdout of one command to stdin of another:**

```
command1 | command2 | command3
   │          │          │
   └──stdout→stdin──stdout→stdin
```

**Basic Examples:**
```bash
# List files and count them
$ ls | wc -l
25

# Search in command output
$ ps aux | grep firefox

# Sort and unique
$ cat names.txt | sort | uniq

# Page through long output
$ ls -la /usr/bin | less

# Find and count
$ find . -name "*.txt" | wc -l
```

---

### Building Command Pipelines

**Simple Pipelines:**
```bash
# Process text
$ cat file.txt | tr 'a-z' 'A-Z' | sort
# Read → Uppercase → Sort

# Log analysis
$ cat access.log | grep "404" | wc -l
# Read → Filter 404s → Count

# System info
$ ps aux | grep apache | awk '{print $2}'
# List processes → Filter → Extract PID
```

**Complex Pipelines:**
```bash
# Top 10 most common words in file
$ cat file.txt | \
  tr -cs 'A-Za-z' '\n' | \
  tr 'A-Z' 'a-z' | \
  sort | \
  uniq -c | \
  sort -rn | \
  head -10

# Disk usage by directory (top 10)
$ du -h /var | \
  sort -rh | \
  head -10

# Find large files
$ find / -type f -size +100M 2>/dev/null | \
  xargs ls -lh | \
  awk '{print $5, $9}' | \
  sort -rh

# Network analysis
$ netstat -an | \
  grep ESTABLISHED | \
  awk '{print $5}' | \
  cut -d: -f1 | \
  sort | \
  uniq -c | \
  sort -rn
```

---

### Common Pipeline Patterns

**1. Filter and Count**
```bash
# Count errors in log
$ grep "ERROR" logfile.txt | wc -l

# Count unique IP addresses
$ cat access.log | cut -d' ' -f1 | sort | uniq | wc -l
```

**2. Sort and Extract Top/Bottom**
```bash
# Top 5 largest files in current directory
$ ls -lh | sort -k5 -h | tail -5

# Bottom 10 by number
$ cat numbers.txt | sort -n | head -10
```

**3. Search, Extract, Process**
```bash
# Find all TODO comments in Python files
$ grep -r "TODO" --include="*.py" . | \
  cut -d: -f2 | \
  sort | \
  uniq

# Extract and count error types
$ grep "ERROR" system.log | \
  awk '{print $5}' | \
  sort | \
  uniq -c | \
  sort -rn
```

**4. Transform and Save**
```bash
# Convert all to uppercase and save
$ cat file.txt | tr 'a-z' 'A-Z' > upper.txt

# Remove duplicates and save
$ cat list.txt | sort | uniq > unique_list.txt
```

---

### Combining Redirection and Pipes

```bash
# Pipe and redirect output
$ ls -la | grep "^d" > directories.txt

# Pipe and redirect errors
$ find / -name "*.conf" 2> /dev/null | grep nginx > nginx_configs.txt

# Complex combination
$ cat file.txt | \
  grep "ERROR" | \
  sort | \
  uniq -c > error_summary.txt 2> errors.log

# Pipeline with input from file
$ sort < unsorted.txt | uniq | tee sorted.txt | wc -l
```

---

## 6.5 Advanced Tools

### tee - Read and Write

**tee reads from stdin and writes to both stdout and files:**

```
stdin → [tee] → stdout
          │
          └→ file(s)
```

**Basic Usage:**
```bash
# Display and save
$ ls -la | tee filelist.txt
# Shows output on screen AND saves to file

# Append instead of overwrite
$ date | tee -a logfile.txt

# Write to multiple files
$ echo "Important data" | tee file1.txt file2.txt file3.txt

# In pipeline
$ cat file.txt | tee original.txt | tr 'a-z' 'A-Z' | tee uppercase.txt
```

**Practical Examples:**
```bash
# Watch and log simultaneously
$ ping google.com | tee ping.log

# Save intermediate pipeline results
$ cat data.csv | \
  tee raw.csv | \
  cut -d',' -f1,3 | \
  tee extracted.csv | \
  sort | \
  tee sorted.csv | \
  uniq

# Sudo with tee (write to protected file)
$ echo "new content" | sudo tee /etc/protected.conf
# Can't redirect with sudo, but can pipe to sudo tee

# Monitor and save build output
$ make 2>&1 | tee build.log
```

---

### xargs - Build and Execute Commands

**xargs builds command lines from standard input:**

```bash
# Basic syntax
$ command1 | xargs command2

# Example: Delete files from list
$ cat files_to_delete.txt
old1.log
old2.log
old3.log

$ cat files_to_delete.txt | xargs rm
# Executes: rm old1.log old2.log old3.log
```

**Common Options:**
```bash
# -n: Execute with N arguments at a time
$ echo "1 2 3 4 5" | xargs -n 2 echo
1 2
3 4
5

# -I: Replace string
$ cat files.txt | xargs -I {} cp {} /backup/{}
# For each line, replace {} with that line

# -P: Parallel execution
$ cat urls.txt | xargs -P 4 -I {} wget {}
# Download 4 files at a time

# -t: Print command before executing
$ echo "a b c" | xargs -t echo
echo a b c
a b c

# -p: Prompt before executing
$ cat files.txt | xargs -p rm
rm file1.txt file2.txt? ...y
```

**Practical Examples:**
```bash
# Find and delete old files
$ find . -name "*.tmp" | xargs rm

# Find and move files
$ find . -name "*.log" | xargs -I {} mv {} /logs/

# Download multiple URLs
$ cat urls.txt | xargs -n 1 -P 5 wget

# Search in multiple files
$ find . -name "*.txt" | xargs grep "pattern"

# Change permissions on found files
$ find . -type f -name "*.sh" | xargs chmod +x

# Count lines in multiple files
$ find . -name "*.py" | xargs wc -l

# Copy multiple files
$ ls *.txt | xargs -I {} cp {} /backup/

# Parallel processing
$ cat task_list.txt | xargs -P 10 -I {} process_task.sh {}
```

**xargs vs Loop:**
```bash
# Without xargs (slower - one process per file)
$ for file in $(cat files.txt); do
    rm "$file"
done

# With xargs (faster - fewer processes)
$ cat files.txt | xargs rm

# But be careful with spaces in filenames!
$ find . -name "*.txt" -print0 | xargs -0 rm
# -print0 and -0 handle spaces/special chars
```

---

## 6.6 Advanced Redirection Techniques

### Swapping stdout and stderr

```bash
# Swap stdout and stderr
$ command 3>&1 1>&2 2>&3
# 3>&1 : Create fd 3 as copy of stdout
# 1>&2 : Redirect stdout to stderr
# 2>&3 : Redirect stderr to fd 3 (original stdout)

# Practical use: Get errors on stdout
$ ls /nonexistent /etc 3>&1 1>&2 2>&3 | grep "cannot"
```

### Appending to Protected Files

```bash
# Can't use >> with sudo
$ sudo echo "text" >> /etc/file    # FAILS!
# The redirection happens as regular user

# Solution: Use tee
$ echo "text" | sudo tee -a /etc/file

# Or use sudo with bash -c
$ sudo bash -c 'echo "text" >> /etc/file'
```

### Multiple Redirections

```bash
# Save stdout and stderr separately
$ command 1> output.txt 2> errors.txt

# Save to file and display on screen (using tee)
$ command 2>&1 | tee output.txt

# Save stdout, display stderr
$ command > output.txt
# (stderr still goes to screen)

# Display stdout, save stderr
$ command 2> errors.txt
# (stdout still goes to screen)
```

### Process Substitution

**Use command output as a file:**
```bash
# <(command) creates temporary file with command output

# Compare outputs of two commands
$ diff <(ls /dir1) <(ls /dir2)

# Use with commands expecting file arguments
$ paste <(cut -d',' -f1 file.csv) <(cut -d',' -f3 file.csv)

# Complex example
$ join <(sort file1.txt) <(sort file2.txt)
```

---

## 💡 Key Concepts Summary

1. **Three streams**: stdin (0), stdout (1), stderr (2)
2. **>** overwrites, **>>** appends
3. **2>** redirects errors, **&>** redirects both
4. **/dev/null** discards everything
5. **|** pipes stdout to next command's stdin
6. **tee** displays and saves simultaneously
7. **xargs** builds commands from input
8. **<<** here document, **<<<** here string
9. **Order matters** in redirections

---

## 🔥 Real-World Use Cases

### Use Case 1: Log File Management

```bash
# Separate normal and error logs
$ ./app.sh > app.log 2> app.errors

# Combined log with timestamp
$ ./app.sh 2>&1 | while read line; do
    echo "$(date '+%Y-%m-%d %H:%M:%S') $line"
done | tee -a app_combined.log

# Rotate logs
$ cat current.log >> archive.log
$ > current.log    # Truncate current log
```

### Use Case 2: System Monitoring

```bash
# Collect system stats
$ {
    echo "=== System Report $(date) ==="
    echo "=== Disk Usage ==="
    df -h
    echo "=== Memory Usage ==="
    free -h
    echo "=== Top Processes ==="
    ps aux --sort=-%mem | head -10
} > system_report_$(date +%Y%m%d).txt

# Monitor and alert
$ disk_usage=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
$ if [ $disk_usage -gt 80 ]; then
    echo "Disk usage critical: ${disk_usage}%" | \
    mail -s "Alert: Disk Space" admin@example.com
fi
```

### Use Case 3: Data Processing Pipeline

```bash
# Process CSV: extract, filter, sort, save
$ cat sales_data.csv | \
  grep -v "^#" | \               # Remove comments
  cut -d',' -f1,3,4 | \           # Extract columns
  grep "2026" | \                 # Filter by year
  sort -t',' -k3 -n | \           # Sort by amount
  tee processed.csv | \           # Save intermediate
  awk -F',' '{sum+=$3} END {print "Total:", sum}' > summary.txt
```

### Use Case 4: Backup Script

```bash
#!/bin/bash
# Backup script with logging

LOG_FILE="/var/log/backup.log"
ERROR_LOG="/var/log/backup_errors.log"

{
    echo "Starting backup at $(date)"
    
    tar -czf /backup/data_$(date +%Y%m%d).tar.gz /data 2>&1 | \
    tee -a $LOG_FILE
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        echo "Backup successful"
    else
        echo "Backup failed" | tee -a $ERROR_LOG
    fi
    
    echo "Finished at $(date)"
} >> $LOG_FILE 2>&1
```

### Use Case 5: Parallel Processing

```bash
# Download multiple files in parallel
$ cat urls.txt | \
  xargs -P 10 -I {} wget -q {} -P downloads/

# Process multiple files
$ find /data -name "*.dat" -print0 | \
  xargs -0 -P 4 -I {} process_file.sh {}

# Parallel compression
$ find . -name "*.txt" -print0 | \
  xargs -0 -P $(nproc) -I {} gzip {}
```

---

## 🧪 Practice Problems

### Beginner Level

**Problem 1**: Redirect the output of `ls -la` to a file called `filelist.txt`.

**Problem 2**: Append the current date to a file called `dates.log` without overwriting it.

**Problem 3**: Run a command that produces an error, and redirect the error message to `errors.txt`.

**Problem 4**: Use a pipe to count how many files are in your home directory.

**Problem 5**: Create a file using a here document that contains 3 lines of text.

### Intermediate Level

**Problem 6**: Find all `.txt` files in your home directory and save the list to `textfiles.txt`, but send any error messages to `/dev/null`.

**Problem 7**: Display the output of `ps aux` on the screen AND save it to `processes.txt`.

**Problem 8**: Use a pipeline to find the 10 largest files in `/var/log` (you may need sudo).

**Problem 9**: Create a command that redirects both stdout and stderr to the same file `combined.log`.

**Problem 10**: Use xargs to delete all `.tmp` files found by the find command.

### Advanced Level

**Problem 11**: Create a pipeline that processes `/etc/passwd` to show only usernames of users with UID >= 1000, sorted alphabetically.

**Problem 12**: Write a command that monitors a log file in real-time, filters for ERROR lines, and saves them to `errors_only.log` while still displaying them on screen.

**Problem 13**: Use process substitution to compare the list of installed packages on two systems (simulate with two different package listings).

**Problem 14**: Create a command pipeline that finds the top 5 IP addresses accessing your web server from the access log.

**Problem 15**: Build a pipeline that counts unique words in a file, sorts by frequency, and saves top 20 to `word_freq.txt`.

---

## 📝 Solutions

### Beginner Solutions

**Solution 1**:
```bash
$ ls -la > filelist.txt
$ cat filelist.txt
# Contains directory listing
```

**Solution 2**:
```bash
$ date >> dates.log
$ cat dates.log
Fri Jan 31 10:00:00 NPT 2026

# Run again
$ date >> dates.log
$ cat dates.log
Fri Jan 31 10:00:00 NPT 2026
Fri Jan 31 10:05:00 NPT 2026
```

**Solution 3**:
```bash
$ ls /nonexistent 2> errors.txt
$ cat errors.txt
ls: cannot access '/nonexistent': No such file or directory

# Or with cat
$ cat nonexistent.txt 2> errors.txt
```

**Solution 4**:
```bash
$ ls ~ | wc -l
25

# Or more specifically
$ ls -1 ~ | wc -l
```

**Solution 5**:
```bash
$ cat > myfile.txt << EOF
First line
Second line
Third line
EOF

$ cat myfile.txt
First line
Second line
Third line
```

### Intermediate Solutions

**Solution 6**:
```bash
$ find ~ -name "*.txt" > textfiles.txt 2> /dev/null

# Or with both in one command
$ find ~ -name "*.txt" 2> /dev/null > textfiles.txt
```

**Solution 7**:
```bash
$ ps aux | tee processes.txt

# Verify both
$ # Output was displayed on screen
$ cat processes.txt    # And it's in the file
```

**Solution 8**:
```bash
$ sudo du -ah /var/log | sort -rh | head -10

# Or finding actual files
$ sudo find /var/log -type f -exec ls -lh {} \; 2>/dev/null | \
  awk '{print $5, $9}' | \
  sort -rh | \
  head -10
```

**Solution 9**:
```bash
# Method 1 (modern)
$ command &> combined.log

# Method 2 (traditional)
$ command > combined.log 2>&1

# Test it
$ ls /etc /nonexistent &> combined.log
$ cat combined.log
ls: cannot access '/nonexistent': No such file or directory
/etc:
...
```

**Solution 10**:
```bash
$ find . -name "*.tmp" | xargs rm

# Safer with -print0 and -0 (handles spaces)
$ find . -name "*.tmp" -print0 | xargs -0 rm

# Even safer - prompt before delete
$ find . -name "*.tmp" -print0 | xargs -0 -p rm
```

### Advanced Solutions

**Solution 11**:
```bash
$ awk -F':' '$3 >= 1000 {print $1}' /etc/passwd | sort

# Or with cut and grep
$ grep -E ":[0-9]{4,}:" /etc/passwd | cut -d':' -f1 | sort

# Explanation:
# awk -F':' : Use colon as field separator
# $3 >= 1000 : Filter where 3rd field (UID) >= 1000
# {print $1} : Print 1st field (username)
# sort : Sort alphabetically
```

**Solution 12**:
```bash
# Method 1: Using tee
$ tail -f /var/log/application.log | \
  grep --line-buffered "ERROR" | \
  tee -a errors_only.log

# Method 2: Using awk
$ tail -f /var/log/application.log | \
  awk '/ERROR/ {print | "tee -a errors_only.log"}'

# --line-buffered ensures immediate output
# Ctrl+C to stop monitoring
```

**Solution 13**:
```bash
# Assuming you have two package lists
# system1_packages.txt and system2_packages.txt

# Show packages only on system1
$ comm -23 <(sort system1_packages.txt) <(sort system2_packages.txt)

# Show packages only on system2
$ comm -13 <(sort system1_packages.txt) <(sort system2_packages.txt)

# Show common packages
$ comm -12 <(sort system1_packages.txt) <(sort system2_packages.txt)

# With diff
$ diff <(sort system1_packages.txt) <(sort system2_packages.txt)
```

**Solution 14**:
```bash
# Assuming Apache/Nginx access log format
$ cat access.log | \
  awk '{print $1}' | \
  sort | \
  uniq -c | \
  sort -rn | \
  head -5

# Or more detailed
$ awk '{print $1}' access.log | \
  sort | \
  uniq -c | \
  sort -rn | \
  head -5 | \
  awk '{print $2, "(" $1, "requests)"}'

# Example output:
# 192.168.1.100 (1543 requests)
# 10.0.0.50 (892 requests)
# 172.16.0.10 (654 requests)
```

**Solution 15**:
```bash
$ cat file.txt | \
  tr -cs 'A-Za-z' '\n' | \
  tr 'A-Z' 'a-z' | \
  sort | \
  uniq -c | \
  sort -rn | \
  head -20 > word_freq.txt

$ cat word_freq.txt
    145 the
     98 and
     87 to
     76 of
     65 a
     ...

# Explanation:
# tr -cs 'A-Za-z' '\n' : Extract words (one per line)
# tr 'A-Z' 'a-z' : Lowercase everything
# sort : Sort words
# uniq -c : Count occurrences
# sort -rn : Sort by count (descending)
# head -20 : Top 20
# > word_freq.txt : Save to file
```

---

## 🎓 Learning Tips

### Redirection Memory Aids

**Think of > as an arrow pointing to a file:**
- `>` = Create/overwrite
- `>>` = Add more (double arrow = append)
- `<` = Come from file

**File Descriptors:**
- **0** = **Z**ero input (stdin)
- **1** = **One** output (stdout)
- **2** = **Two** errors (stderr)

### Common Patterns

```bash
# Save output
command > file.txt

# Save and view
command | tee file.txt

# Silent execution
command &> /dev/null

# Errors only
command 2> errors.txt 1> /dev/null

# Everything to log
command >> log.txt 2>&1
```

### Pipeline Building Strategy

1. **Start simple** - Test each command separately
2. **Add one pipe at a time** - Verify each stage
3. **Use tee for debugging** - See intermediate results
4. **Save complex pipelines** - Put in scripts

```bash
# Build incrementally
$ cat file.txt                    # Step 1: view data
$ cat file.txt | grep pattern     # Step 2: filter
$ cat file.txt | grep pattern | sort    # Step 3: sort
$ cat file.txt | grep pattern | sort | uniq    # Step 4: unique
```

### Best Practices

1. **Quote variables in redirections**
   ```bash
   $ echo "text" > "$filename"    # Good
   $ echo "text" > $filename      # Risky with spaces
   ```

2. **Check PIPESTATUS for pipeline errors**
   ```bash
   $ command1 | command2 | command3
   $ echo ${PIPESTATUS[@]}    # Shows exit codes
   ```

3. **Use tee for important pipelines**
   ```bash
   # Save intermediate results
   $ command1 | tee step1.txt | command2 | tee step2.txt | command3
   ```

4. **Handle spaces in filenames with xargs**
   ```bash
   $ find . -name "*.txt" -print0 | xargs -0 command
   ```

5. **Buffer output for performance**
   ```bash
   $ unbuffer command | tee log.txt    # For line-buffered output
   ```

### Common Mistakes to Avoid

1. **Wrong redirection order**
   ```bash
   # WRONG - stderr still goes to screen
   $ command 2>&1 > file.txt
   
   # RIGHT
   $ command > file.txt 2>&1
   ```

2. **Using > when you mean >>**
   ```bash
   # DESTROYS existing content
   $ important_command > log.txt
   
   # APPENDS (usually what you want)
   $ important_command >> log.txt
   ```

3. **Forgetting quotes with special characters**
   ```bash
   # WRONG
   $ echo $PATH > file.txt
   
   # RIGHT
   $ echo "$PATH" > file.txt
   ```

4. **Not handling errors**
   ```bash
   # Errors go to screen, might miss them
   $ command > output.txt
   
   # Better - save errors too
   $ command > output.txt 2>&1
   ```

5. **Inefficient xargs usage**
   ```bash
   # Creates one process per line (slow)
   $ cat files.txt | while read f; do rm "$f"; done
   
   # Creates fewer processes (fast)
   $ cat files.txt | xargs rm
   ```

---

## 🔗 Connections to Future Chapters

- **Chapter 7**: Process management uses redirection for process control
- **Chapter 11**: Shell scripts heavily use redirection and pipes
- **Chapter 13**: System monitoring requires log file redirection
- **Chapter 14**: Cron jobs need proper output redirection
- **Chapter 21**: Troubleshooting uses pipelines to analyze logs

---

## 📚 Additional Resources

### Man Pages
```bash
man bash    # See "REDIRECTION" section
man xargs
man tee
```

### Advanced Topics
- Named Pipes (FIFOs): `man mkfifo`
- Process Substitution: `man bash` (search for "Process Substitution")
- Coproc: Bidirectional pipes in bash

### Practice
```bash
# Create test files
$ for i in {1..10}; do echo "Line $i" > file$i.txt; done

# Practice pipelines
$ cat file*.txt | grep "Line" | sort | uniq
```

---

## ✅ Self-Assessment Checklist

Before moving to Chapter 7, ensure you can:
- [ ] Understand stdin, stdout, and stderr
- [ ] Redirect output with > and >>
- [ ] Redirect errors with 2>
- [ ] Redirect both stdout and stderr
- [ ] Use /dev/null to discard output
- [ ] Read input from files with <
- [ ] Create here documents with <<
- [ ] Use pipes to connect commands
- [ ] Build multi-stage pipelines
- [ ] Use tee to split output
- [ ] Use xargs to build commands
- [ ] Combine redirection and pipes effectively
- [ ] Debug pipelines with intermediate output
- [ ] Handle errors in pipelines

---

## 🚀 Next Steps

Excellent work! You now understand I/O redirection and pipes - the plumbing of Linux that makes it so powerful.

In **Chapter 7**, you'll learn:
- Process management and control
- Viewing processes with ps, top, htop
- Managing foreground and background jobs
- Killing and controlling processes
- Process priorities with nice
- Monitoring system resources

**Challenge**: Create a pipeline that monitors your system's CPU usage, filters for processes using >50% CPU, and logs them to a file with timestamps. Use what you learned about pipes, redirection, and tee!

---

*"The pipe is the most powerful concept in Linux. Master it, and you master the system."*