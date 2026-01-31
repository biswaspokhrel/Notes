# Chapter 7: Process Management

## 🎯 Learning Objectives
By the end of this chapter, you will:
- Understand what processes are and how they work
- View and monitor processes with ps, top, and htop
- Manage foreground and background jobs
- Send signals to processes with kill
- Control process priorities with nice and renice
- Use nohup for persistent processes
- Monitor system resources effectively
- Troubleshoot process-related issues

---

## 7.1 Understanding Processes

### What is a Process?

**A process is a running instance of a program.**

When you run a command or program, Linux creates a process for it. Each process:
- Has a unique Process ID (PID)
- Has an owner (user who started it)
- Uses system resources (CPU, memory)
- Can create other processes (child processes)
- Has a parent process (except PID 1)

**Example:**
```bash
# Start a program
$ firefox &
[1] 12345    # Job number [1], PID 12345

# This creates a process
# PID: 12345
# Owner: Current user
# Parent: Your shell (bash)
```

### Process Hierarchy

```
init/systemd (PID 1)
    │
    ├─ sshd (PID 1234)
    │   └─ sshd (PID 5678) - your session
    │       └─ bash (PID 5679)
    │           ├─ vim (PID 6789)
    │           └─ firefox (PID 7890)
    │
    └─ apache2 (PID 2000)
        ├─ apache2 (PID 2001)
        └─ apache2 (PID 2002)
```

**Key Concepts:**
- **PID (Process ID)**: Unique identifier for each process
- **PPID (Parent Process ID)**: PID of the parent process
- **UID (User ID)**: User who owns the process
- **GID (Group ID)**: Group of the process

**View Process Tree:**
```bash
$ pstree
systemd─┬─apache2───2*[apache2]
        ├─cron
        ├─sshd───sshd───bash───pstree
        ├─systemd-journal
        └─systemd-logind

# With PIDs
$ pstree -p
systemd(1)─┬─apache2(2000)─┬─apache2(2001)
           │               └─apache2(2002)
           ├─cron(1500)
           └─sshd(1234)───sshd(5678)───bash(5679)───pstree(8000)

# For specific user
$ pstree -u john
```

### Process States

Processes can be in different states:

| State | Symbol | Meaning |
|-------|--------|---------|
| Running | R | Currently executing or ready to run |
| Sleeping | S | Waiting for an event (interruptible) |
| Uninterruptible | D | Sleeping (cannot be interrupted) |
| Stopped | T | Stopped by job control signal |
| Zombie | Z | Terminated but not cleaned up by parent |

```bash
# View process states
$ ps aux | awk '{print $8}' | sort | uniq -c
     45 S      # 45 sleeping processes
      3 R      # 3 running
      1 Z      # 1 zombie
```

### Foreground vs Background Processes

**Foreground Process:**
- Takes control of terminal
- You can't run other commands until it finishes
- Can interact with it (Ctrl+C to stop)

**Background Process:**
- Runs without controlling terminal
- You can continue using the shell
- Started with `&` or sent there with Ctrl+Z and `bg`

```bash
# Foreground (blocks terminal)
$ sleep 100
# Terminal is blocked for 100 seconds

# Background (doesn't block)
$ sleep 100 &
[1] 12345
$ # You can continue working
```

---

## 7.2 Viewing Processes

### ps - Process Status

**Basic Usage:**
```bash
# Processes in current terminal
$ ps
  PID TTY          TIME CMD
 5679 pts/0    00:00:00 bash
 8001 pts/0    00:00:00 ps

# All processes for current user
$ ps x
  PID TTY      STAT   TIME COMMAND
 2341 ?        Ss     0:00 /lib/systemd/systemd --user
 2342 ?        S      0:00 (sd-pam)
 5679 pts/0    Ss     0:00 -bash
 8002 pts/0    R+     0:00 ps x

# All processes (all users)
$ ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  168976 11234 ?        Ss   Jan30   0:02 /sbin/init
root         2  0.0  0.0      0     0 ?        S    Jan30   0:00 [kthreadd]
john      5679  0.0  0.1  21432  5123 pts/0    Ss   10:00   0:00 -bash
```

**Understanding ps aux Output:**
```
USER    : Owner of process
PID     : Process ID
%CPU    : CPU usage percentage
%MEM    : Memory usage percentage
VSZ     : Virtual memory size (KB)
RSS     : Resident Set Size (physical memory, KB)
TTY     : Terminal associated with process (? = no terminal)
STAT    : Process state (R=running, S=sleeping, Z=zombie, etc.)
START   : Time process started
TIME    : CPU time consumed
COMMAND : Command that started the process
```

**Common ps Options:**
```bash
# BSD style (no dash)
ps aux          # All processes, all users
ps ax           # All processes
ps auxf         # Forest (tree) view
ps auxww        # Wide output (don't truncate)

# UNIX style (with dash)
ps -ef          # All processes, full format
ps -eF          # Extra full format
ps -ely         # Long format with priority
ps -u john      # Processes by user john
ps -C firefox   # Processes by command name

# Custom format
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%mem
  PID  PPID CMD                         %MEM %CPU
 7890  5679 /usr/bin/firefox             15.2  3.1
 6789  5679 vim document.txt              2.1  0.1
 5679  5678 -bash                         0.5  0.0
```

**Useful ps Commands:**
```bash
# Find specific process
$ ps aux | grep firefox
john  7890  3.1 15.2 2345678 987654 ?  Sl  10:00  2:15 /usr/bin/firefox

# Top 10 memory consumers
$ ps aux --sort=-%mem | head -11

# Top 10 CPU consumers  
$ ps aux --sort=-%cpu | head -11

# Process tree for user
$ ps f -u john
  PID TTY      STAT   TIME COMMAND
 5679 pts/0    Ss     0:00 -bash
 6789 pts/0    S+     0:00  \_ vim document.txt
 7890 ?        Sl     2:15 /usr/bin/firefox

# Count processes
$ ps aux | wc -l
156

# Processes by user
$ ps -u john --no-headers | wc -l
23
```

---

### top - Interactive Process Viewer

**Basic Usage:**
```bash
$ top

top - 10:30:15 up 5 days,  2:45,  2 users,  load average: 0.15, 0.20, 0.18
Tasks: 156 total,   1 running, 155 sleeping,   0 stopped,   0 zombie
%Cpu(s):  2.3 us,  0.7 sy,  0.0 ni, 96.8 id,  0.2 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :   7823.5 total,   1234.2 free,   3456.1 used,   3133.2 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   4123.4 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 7890 john      20   0 2345678 987654 123456 S   3.1  12.3   2:15.67 firefox
 2000 root      20   0  567890 234567  12345 S   1.0   2.9   5:30.12 apache2
 6789 john      20   0   21432   5123   3456 S   0.3   0.1   0:05.23 vim
    1 root      20   0  168976  11234   8765 S   0.0   0.1   0:02.45 systemd
```

**Understanding top Output:**

**Header Information:**
```
10:30:15                 : Current time
up 5 days, 2:45          : System uptime
2 users                  : Logged in users
load average: 0.15, 0.20, 0.18 : 1, 5, 15 minute load averages
```

**Tasks:**
```
156 total   : Total processes
1 running   : Currently running
155 sleeping: Waiting for resources
0 zombie    : Terminated but not cleaned up
```

**CPU Statistics:**
```
us (user)      : Time in user processes
sy (system)    : Time in kernel
ni (nice)      : Time in niced processes
id (idle)      : Idle time
wa (wait)      : Waiting for I/O
hi (hardware)  : Hardware interrupts
si (software)  : Software interrupts
st (steal)     : Stolen by hypervisor
```

**Memory:**
```
total : Total physical memory
free  : Unused memory
used  : Used memory
buff/cache : Buffers and cache
avail : Available for new processes
```

**Interactive Commands in top:**
```
Space    : Refresh immediately
h or ?   : Help
q        : Quit

k        : Kill process (asks for PID)
r        : Renice process (change priority)

M        : Sort by memory usage
P        : Sort by CPU usage
T        : Sort by running time
N        : Sort by PID

u        : Filter by user
c        : Show full command line
1        : Toggle individual CPU stats
t        : Toggle tasks/CPU summary
m        : Toggle memory summary

W        : Write configuration to file
```

**top with Options:**
```bash
# Batch mode (for scripts)
$ top -b -n 1
# -b : batch mode
# -n 1 : one iteration

# Monitor specific user
$ top -u john

# Update every 2 seconds
$ top -d 2

# Show specific process
$ top -p 7890

# Show multiple processes
$ top -p 7890,2000,6789

# Save output
$ top -b -n 1 > top_snapshot.txt
```

---

### htop - Enhanced Interactive Viewer

**htop** is a more user-friendly alternative to top (may need installation).

```bash
# Install htop
$ sudo apt install htop    # Debian/Ubuntu
$ sudo yum install htop    # RHEL/CentOS

# Run htop
$ htop
```

**htop Features:**
- Color-coded display
- Mouse support
- Vertical and horizontal scrolling
- Tree view
- Easy process killing
- Shows all CPUs graphically

**htop Navigation:**
```
F1 (h)   : Help
F2 (S)   : Setup (configuration)
F3 (/)   : Search processes
F4 (\)   : Filter processes
F5 (t)   : Tree view
F6 (<>)  : Sort by column
F7 (])   : Increase priority (nice +)
F8 ([)   : Decrease priority (nice -)
F9 (k)   : Kill process
F10 (q)  : Quit

Space    : Tag process
U        : Untag all
c        : Tag process and children
```

**htop Advantages:**
- Easier to read
- Better visualization
- Easier to kill processes
- Shows process tree easily
- Can scroll horizontally/vertically

---

### Other Process Viewing Tools

**pgrep - Find Processes by Name:**
```bash
# Find PID by name
$ pgrep firefox
7890

# With full command line
$ pgrep -a firefox
7890 /usr/bin/firefox

# For specific user
$ pgrep -u john firefox
7890

# Count processes
$ pgrep -c firefox
1

# Latest (most recently started)
$ pgrep -n firefox
7890

# Oldest
$ pgrep -o firefox
7890
```

**pidof - Find PID of Running Program:**
```bash
$ pidof firefox
7890

$ pidof apache2
2000 2001 2002
```

---

## 7.3 Managing Background Jobs

### Starting Background Jobs

**With & operator:**
```bash
# Start in background
$ sleep 100 &
[1] 12345    # [Job number] PID
$            # Prompt returns immediately

# Multiple background jobs
$ sleep 100 &
[1] 12345
$ sleep 200 &
[2] 12346
$ sleep 300 &
[3] 12347
```

### Viewing Jobs

**jobs command:**
```bash
$ jobs
[1]   Running                 sleep 100 &
[2]-  Running                 sleep 200 &
[3]+  Running                 sleep 300 &

# With PIDs
$ jobs -l
[1]  12345 Running                 sleep 100 &
[2]- 12346 Running                 sleep 200 &
[3]+ 12347 Running                 sleep 300 &

# Running jobs only
$ jobs -r

# Stopped jobs only
$ jobs -s
```

**Job Symbols:**
- `+` : Current job (most recent)
- `-` : Previous job
- ` ` : Older jobs

---

### Foreground and Background Control

**Ctrl+Z - Suspend Foreground Job:**
```bash
$ sleep 100
^Z
[1]+  Stopped                 sleep 100

# Process is stopped (not running)
```

**bg - Resume in Background:**
```bash
$ sleep 100
^Z
[1]+  Stopped                 sleep 100

$ bg
[1]+ sleep 100 &
# Job continues in background

# Resume specific job
$ bg %2    # Resume job 2
```

**fg - Bring to Foreground:**
```bash
# Current job
$ fg
sleep 100    # Now in foreground

# Specific job
$ fg %2      # Bring job 2 to foreground
$ fg %      # Same as fg %+ (current job)
$ fg %-      # Previous job
```

**Complete Workflow:**
```bash
# Start a job
$ vim document.txt

# Suspend it (Ctrl+Z)
^Z
[1]+  Stopped                 vim document.txt

# Do something else
$ grep "pattern" file.txt

# Return to vim
$ fg
vim document.txt

# Or keep vim in background and start another
$ vim other.txt

# List jobs
$ jobs
[1]-  Stopped                 vim document.txt
[2]+  Stopped                 vim other.txt

# Switch between them
$ fg %1    # Work on document.txt
# Ctrl+Z to suspend
$ fg %2    # Work on other.txt
```

---

### nohup - Persist After Logout

**nohup** runs command immune to hangup signal (survives logout).

```bash
# Basic usage
$ nohup command &
nohup: ignoring input and appending output to 'nohup.out'

# With custom output
$ nohup ./long_script.sh > script.log 2>&1 &

# Example: long download
$ nohup wget http://example.com/large_file.iso &

# Process continues even if you log out
```

**Checking nohup Jobs:**
```bash
# nohup output goes to nohup.out
$ cat nohup.out

# Or check custom log
$ tail -f script.log
```

**nohup vs Background:**
- `command &` : Background, but killed on logout
- `nohup command &` : Background, persists after logout

---

## 7.4 Killing Processes

### Understanding Signals

**Signals are notifications sent to processes.**

Common Signals:

| Signal | Number | Meaning | Effect |
|--------|--------|---------|--------|
| SIGHUP | 1 | Hangup | Terminate (or reload config) |
| SIGINT | 2 | Interrupt | Terminate (Ctrl+C) |
| SIGQUIT | 3 | Quit | Terminate with core dump |
| SIGKILL | 9 | Kill | Force terminate (cannot be caught) |
| SIGTERM | 15 | Terminate | Graceful termination (default) |
| SIGSTOP | 19 | Stop | Pause process (cannot be caught) |
| SIGCONT | 18 | Continue | Resume stopped process |
| SIGUSR1 | 10 | User-defined | Custom action |
| SIGUSR2 | 12 | User-defined | Custom action |

```bash
# List all signals
$ kill -l
 1) SIGHUP       2) SIGINT       3) SIGQUIT      4) SIGILL
 5) SIGTRAP      6) SIGABRT      7) SIGBUS       8) SIGFPE
 9) SIGKILL     10) SIGUSR1     11) SIGSEGV     12) SIGUSR2
13) SIGPIPE     14) SIGALRM     15) SIGTERM     16) SIGSTKFLT
...
```

---

### kill - Send Signal to Process

**Basic Usage:**
```bash
# Graceful termination (SIGTERM - default)
$ kill 12345

# Force kill (SIGKILL)
$ kill -9 12345
# or
$ kill -SIGKILL 12345
# or
$ kill -KILL 12345

# Send specific signal
$ kill -HUP 12345      # Reload configuration
$ kill -TERM 12345     # Graceful termination
$ kill -STOP 12345     # Pause process
$ kill -CONT 12345     # Resume process

# Kill by job number
$ kill %1
$ kill %2
```

**kill vs kill -9:**
```bash
# SIGTERM (15) - Polite request
$ kill 12345
# Process can:
# - Save data
# - Close files
# - Clean up
# - Ignore the signal

# SIGKILL (9) - Immediate termination
$ kill -9 12345
# Process:
# - Cannot catch or ignore
# - Terminated immediately
# - No cleanup
# - Use as last resort!
```

**Best Practice - Escalation:**
```bash
# Try graceful first
$ kill 12345
$ sleep 5

# Check if still running
$ ps -p 12345
# If still running:
$ kill -9 12345
```

---

### killall - Kill by Name

```bash
# Kill all processes with name
$ killall firefox

# Force kill
$ killall -9 firefox

# Interactive (ask for confirmation)
$ killall -i firefox
Kill firefox(7890) ? (y/N) y

# Kill processes by user
$ killall -u john firefox

# Dry run (show what would be killed)
$ killall -s firefox
```

---

### pkill - Kill by Pattern

```bash
# Kill by pattern
$ pkill firefox

# Kill by full command line
$ pkill -f "firefox.*private"

# Kill by user
$ pkill -u john

# Kill by terminal
$ pkill -t pts/0

# Signal option
$ pkill -9 firefox
$ pkill -HUP apache2    # Reload Apache

# Newest/oldest
$ pkill -n firefox    # Kill newest
$ pkill -o firefox    # Kill oldest
```

---

### timeout - Run with Time Limit

```bash
# Kill after 10 seconds
$ timeout 10 command

# Custom signal
$ timeout -s KILL 10 command

# Preserve exit status
$ timeout --preserve-status 10 command

# Example: limit backup time
$ timeout 1h tar -czf backup.tar.gz /data
# Kills if backup takes more than 1 hour
```

---

## 7.5 Process Priority

### Understanding Priority

**Linux uses priority to decide which processes get CPU time.**

- **Nice value**: -20 (highest priority) to 19 (lowest priority)
- Default: 0
- Lower nice = higher priority
- Only root can set negative nice values

**Priority Visualization:**
```
-20  -10    0    10    19
 ├────┼────┼────┼─────┤
 High      Normal     Low
Priority           Priority
```

---

### nice - Start with Priority

```bash
# Start with low priority (nice +10)
$ nice command

# Specific nice value
$ nice -n 10 command
$ nice -10 command      # Same as above

# High priority (requires root)
$ sudo nice -n -10 command

# Very low priority
$ nice -n 19 command

# Examples
$ nice -n 15 tar -czf backup.tar.gz /data
$ nice ffmpeg -i input.mp4 output.avi
$ sudo nice -n -5 important_process
```

---

### renice - Change Priority

```bash
# Change priority of running process
$ renice 10 -p 12345
# Sets PID 12345 to nice value 10

# By user
$ sudo renice 5 -u john
# All of john's processes to nice 5

# By group
$ sudo renice 10 -g developers

# Multiple processes
$ renice 15 -p 12345 -p 12346 -p 12347

# Increase priority (requires root)
$ sudo renice -10 -p 12345
```

**Checking Priority:**
```bash
# With ps
$ ps -eo pid,ni,cmd
  PID  NI CMD
 7890   0 /usr/bin/firefox
12345  10 tar -czf backup.tar.gz

# With top
$ top
# NI column shows nice value
```

---

### Practical Priority Use

```bash
# CPU-intensive backup (low priority)
$ nice -n 19 tar -czf backup.tar.gz /home &

# Important database (high priority)
$ sudo renice -5 -p $(pgrep postgres)

# Batch processing (low priority)
$ nice -n 15 ./process_videos.sh

# Real-time task (highest priority, root only)
$ sudo nice -n -20 ./critical_monitor

# Adjust running process
$ top
# Press 'r'
# Enter PID
# Enter new nice value
```

---

## 7.6 Monitoring System Resources

### System Load

**Load Average:**
Shows average number of processes waiting for CPU.

```bash
# View load average
$ uptime
10:30:15 up 5 days, 2:45, 2 users, load average: 0.15, 0.20, 0.18
                                                    1min 5min 15min

# Interpret load:
# For 4-core CPU:
# < 4.0  : Good
# 4.0    : Fully loaded
# > 4.0  : Overloaded (processes waiting)

# Check CPU count
$ nproc
4

# Or
$ grep -c processor /proc/cpuinfo
4
```

**Load Average Rules:**
- Load = Number of CPU cores: Perfect
- Load < Cores: Underutilized
- Load > Cores: Overloaded

---

### Process-Specific Monitoring

**strace - Trace System Calls:**
```bash
# Trace running process
$ sudo strace -p 12345

# Trace new command
$ strace ls -la

# Summary of calls
$ strace -c ls -la

# Save to file
$ strace -o trace.log command
```

**lsof - List Open Files:**
```bash
# Files opened by process
$ lsof -p 12345

# Processes using file
$ lsof /var/log/syslog

# Network connections
$ lsof -i
$ lsof -i :80        # Port 80
$ lsof -i tcp        # TCP connections

# By user
$ lsof -u john
```

---

## 💡 Key Concepts Summary

1. **Process**: Running instance of a program
2. **PID**: Unique process identifier
3. **ps**: View process snapshots
4. **top/htop**: Interactive process monitoring
5. **&**: Run in background
6. **Ctrl+Z**: Suspend; **bg**: resume in background; **fg**: bring to foreground
7. **kill**: Send signals (15=graceful, 9=force)
8. **nice**: Start with priority; **renice**: change priority
9. **nohup**: Persist after logout
10. **Load average**: System workload indicator

---

## 🔥 Real-World Use Cases

### Use Case 1: Finding Resource Hogs

```bash
# Find top CPU consumers
$ ps aux --sort=-%cpu | head -10

# Find memory hogs
$ ps aux --sort=-%mem | head -10

# Continuous monitoring
$ top -o %CPU    # Sort by CPU

# Kill the worst offender
$ kill $(ps aux --sort=-%cpu | awk 'NR==2 {print $2}')
```

### Use Case 2: Managing Long-Running Tasks

```bash
# Start long task that survives logout
$ nohup ./long_backup.sh > backup.log 2>&1 &
[1] 12345

# Check progress
$ tail -f backup.log

# Or with nice (low priority)
$ nohup nice -n 19 ./long_backup.sh > backup.log 2>&1 &

# Later, check if still running
$ ps -p 12345
$ pgrep -a long_backup
```

### Use Case 3: Gracefully Restarting Service

```bash
# Find Apache processes
$ ps aux | grep apache2
root  2000  ...  /usr/sbin/apache2
www-data 2001  ...  /usr/sbin/apache2

# Graceful reload (reread config)
$ sudo kill -HUP 2000

# Or with systemctl
$ sudo systemctl reload apache2

# Force restart if needed
$ sudo systemctl restart apache2
```

### Use Case 4: Debugging Hung Process

```bash
# Find the process
$ ps aux | grep stuck_program
john 12345  99.0  ...  stuck_program

# Try graceful termination
$ kill 12345
$ sleep 5

# Check if still running
$ ps -p 12345
# If still running, force kill
$ kill -9 12345

# Investigate with strace (before killing)
$ sudo strace -p 12345
# See what it's doing
```

### Use Case 5: Resource-Controlled Batch Processing

```bash
# Process videos with low priority
$ nice -n 19 ./batch_encode.sh &
[1] 12345

# Monitor progress
$ watch -n 5 'ps -p 12345 -o %cpu,%mem,etime,cmd'

# If system gets busy, lower priority more
$ renice 19 -p 12345

# Or pause temporarily
$ kill -STOP 12345
# Resume later
$ kill -CONT 12345
```

---

## 🧪 Practice Problems

### Beginner Level

**Problem 1**: List all processes currently running on your system.

**Problem 2**: Find the PID of your current bash shell.

**Problem 3**: Start a `sleep 100` command in the background and verify it's running.

**Problem 4**: Use `top` to find which process is using the most CPU.

**Problem 5**: Kill a background job you created.

### Intermediate Level

**Problem 6**: Find all processes owned by your user and count them.

**Problem 7**: Start a foreground process, suspend it with Ctrl+Z, then resume it in the background.

**Problem 8**: Use `ps` to show only the PID, user, and command for all Firefox processes.

**Problem 9**: Start a command with nice value of 10 and verify its priority.

**Problem 10**: Find the parent process ID (PPID) of your current shell.

### Advanced Level

**Problem 11**: Create a script that monitors a specific process and restarts it if it dies.

**Problem 12**: Find the top 5 memory-consuming processes and display their PID, %MEM, and command.

**Problem 13**: Use a pipeline to find all zombie processes on the system.

**Problem 14**: Start a long-running process with nohup, log out, log back in, and verify it's still running.

**Problem 15**: Write a one-liner that kills all processes using more than 50% CPU (be careful!).

---

## 📝 Solutions

### Beginner Solutions

**Solution 1**:
```bash
$ ps aux
# Shows all processes

# Or just count
$ ps aux | wc -l
156
```

**Solution 2**:
```bash
$ echo $$
5679

# Or
$ ps -p $$
  PID TTY          TIME CMD
 5679 pts/0    00:00:00 bash
```

**Solution 3**:
```bash
$ sleep 100 &
[1] 12345

# Verify
$ jobs
[1]+  Running                 sleep 100 &

# Or with ps
$ ps -p 12345
  PID TTY          TIME CMD
12345 pts/0    00:00:00 sleep
```

**Solution 4**:
```bash
$ top
# Press 'P' to sort by CPU
# Look at top of list
# Press 'q' to quit

# Or non-interactive
$ ps aux --sort=-%cpu | head -2
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
john      7890  3.1 15.2 2345678 987654 ?      Sl   10:00   2:15 /usr/bin/firefox
```

**Solution 5**:
```bash
$ jobs
[1]+  Running                 sleep 100 &

$ kill %1
[1]+  Terminated              sleep 100

# Verify
$ jobs
# No output - job is gone
```

### Intermediate Solutions

**Solution 6**:
```bash
$ ps -u $(whoami) | wc -l
24

# Or more accurately (without header)
$ ps -u $(whoami) --no-headers | wc -l
23

# With pgrep
$ pgrep -u $(whoami) | wc -l
23
```

**Solution 7**:
```bash
$ sleep 100
# Press Ctrl+Z
^Z
[1]+  Stopped                 sleep 100

$ jobs
[1]+  Stopped                 sleep 100

$ bg
[1]+ sleep 100 &

$ jobs
[1]+  Running                 sleep 100 &
```

**Solution 8**:
```bash
$ ps -C firefox -o pid,user,cmd
  PID USER     CMD
 7890 john     /usr/bin/firefox

# Or with grep
$ ps aux | grep firefox | grep -v grep | awk '{print $2, $1, $11}'
7890 john /usr/bin/firefox

# Or with pgrep
$ pgrep -a firefox
7890 /usr/bin/firefox
```

**Solution 9**:
```bash
$ nice -n 10 sleep 100 &
[1] 12345

# Verify priority
$ ps -p 12345 -o pid,ni,cmd
  PID  NI CMD
12345  10 sleep 100

# Or with top
$ top -p 12345
# Look at NI column
```

**Solution 10**:
```bash
$ ps -p $$ -o ppid=
5678

# Or with full info
$ ps -p $$ -o pid,ppid,cmd
  PID  PPID CMD
 5679  5678 -bash

# The PPID (5678) is the parent
```

### Advanced Solutions

**Solution 11**:
```bash
#!/bin/bash
# monitor_process.sh

PROCESS_NAME="myapp"
COMMAND="/usr/bin/myapp"

while true; do
    # Check if process is running
    if ! pgrep -x "$PROCESS_NAME" > /dev/null; then
        echo "$(date): $PROCESS_NAME not running, starting..."
        $COMMAND &
    fi
    
    # Check every 10 seconds
    sleep 10
done

# Run it
$ ./monitor_process.sh &
```

**Solution 12**:
```bash
$ ps aux --sort=-%mem | head -6 | awk '{print $2, $4, $11}'
PID %MEM COMMAND
7890 15.2 /usr/bin/firefox
6789 2.1 vim
2000 1.5 /usr/sbin/apache2
5679 0.5 -bash
1234 0.3 /usr/sbin/sshd

# Or with custom format
$ ps -eo pid,%mem,cmd --sort=-%mem | head -6
```

**Solution 13**:
```bash
# Find zombie processes
$ ps aux | awk '$8 == "Z" {print}'

# Or simpler
$ ps aux | grep 'Z'

# With just info we need
$ ps -eo pid,ppid,stat,cmd | grep 'Z'

# Count zombies
$ ps aux | awk '$8 == "Z"' | wc -l
```

**Solution 14**:
```bash
# Start process
$ nohup sleep 1000 > /dev/null 2>&1 &
[1] 12345
$ jobs
[1]+  Running                 nohup sleep 1000 > /dev/null 2>&1 &

# Note the PID
$ echo 12345 > /tmp/test_pid

# Log out
$ exit

# Log back in
$ cat /tmp/test_pid
12345

# Check if still running
$ ps -p 12345
  PID TTY          TIME CMD
12345 ?        00:00:00 sleep

# It's still running! (no TTY, detached from terminal)

# Clean up
$ kill 12345
```

**Solution 15**:
```bash
# DANGEROUS - Test carefully!
# Better version: with confirmation

$ ps aux --sort=-%cpu | \
  awk '$3 > 50 {print $2, $3, $11}' | \
  while read pid cpu cmd; do
    echo "Kill $cmd (PID $pid, CPU $cpu%)? (y/n)"
    read answer
    if [ "$answer" = "y" ]; then
        kill $pid
        echo "Killed $pid"
    fi
  done

# Automatic version (USE WITH EXTREME CAUTION!)
$ ps aux --sort=-%cpu | awk '$3 > 50 && NR > 1 {print $2}' | xargs kill

# Explanation:
# ps aux --sort=-%cpu : Sort by CPU descending
# awk '$3 > 50 && NR > 1' : CPU > 50%, skip header
# {print $2} : Print PID
# xargs kill : Kill those PIDs
```

---

## 🎓 Learning Tips

### Process State Mnemonics

- **R** = **R**unning (or ready to run)
- **S** = **S**leeping
- **D** = **D**isk wait (uninterruptible)
- **T** = s**T**opped
- **Z** = **Z**ombie

### Signal Number Memory

- **1** = HUP (first, hang up)
- **2** = INT (Ctrl+C sends this)
- **9** = KILL (forceful, "9mm bullet")
- **15** = TERM (default, polite)

### Job Control Quick Reference

```
Ctrl+Z   : Suspend (stop)
bg       : Background (continue in bg)
fg       : Foreground (bring to fg)
&        : Start in background
jobs     : List jobs
```

### Priority Scale

```
-20 = Highest priority (root only)
  0 = Default
 19 = Lowest priority

Remember: Lower nice value = Higher priority
(Be nice: give up priority = higher nice value)
```

### Best Practices

1. **Always try graceful kill first**
   ```bash
   $ kill PID           # Try this first
   $ sleep 5
   $ kill -9 PID        # Only if needed
   ```

2. **Use specific process selectors**
   ```bash
   # Bad (ambiguous)
   $ killall python
   
   # Good (specific)
   $ pkill -f "python.*myapp.py"
   ```

3. **Monitor before killing**
   ```bash
   $ ps aux | grep process_name
   # Verify it's the right process
   $ kill PID
   ```

4. **Background long tasks**
   ```bash
   # Bad (blocks terminal)
   $ ./long_task.sh
   
   # Good
   $ nohup ./long_task.sh > task.log 2>&1 &
   ```

5. **Use process groups for cleanup**
   ```bash
   # Start process group
   $ ( command1 & command2 & command3 & wait )
   
   # Kill entire group
   $ kill -TERM -$$
   ```

### Common Mistakes to Avoid

1. **kill -9 as first resort**
   ```bash
   # WRONG
   $ kill -9 12345
   
   # RIGHT
   $ kill 12345
   $ sleep 2
   $ ps -p 12345 || echo "Terminated"
   $ # If still running:
   $ kill -9 12345
   ```

2. **Not checking what you're killing**
   ```bash
   # DANGEROUS
   $ killall python
   
   # SAFER
   $ ps aux | grep python
   # Verify, then
   $ kill specific_PID
   ```

3. **Forgetting nohup for long tasks**
   ```bash
   # Will die on logout
   $ ./backup.sh &
   
   # Persists after logout
   $ nohup ./backup.sh &
   ```

4. **Not monitoring resource usage**
   ```bash
   # Don't just start without checking
   $ ./resource_intensive_task
   
   # Better
   $ nice -n 19 ./resource_intensive_task &
   $ top -p $!
   ```

---

## 🔗 Connections to Future Chapters

- **Chapter 8**: User management affects process ownership
- **Chapter 11**: Scripts create and manage processes
- **Chapter 13**: System monitoring uses process information
- **Chapter 14**: Services are persistent processes
- **Chapter 21**: Troubleshooting often involves process analysis

---

## 📚 Additional Resources

### Man Pages
```bash
man ps
man top
man kill
man nice
man renice
man nohup
man signal
```

### Proc Filesystem
```bash
# Process info in /proc
$ ls /proc/12345/
cmdline  cwd  environ  exe  fd/  maps  stat  status

# Command line
$ cat /proc/12345/cmdline

# Current directory
$ ls -l /proc/12345/cwd

# Environment
$ cat /proc/12345/environ | tr '\0' '\n'
```

### Further Reading
- Process states: `man ps` (see PROCESS STATE CODES)
- Signals: `man 7 signal`
- `/proc` filesystem: `man 5 proc`

---

## ✅ Self-Assessment Checklist

Before moving to Chapter 8, ensure you can:
- [ ] Understand what processes are and how they work
- [ ] View processes with ps in various formats
- [ ] Use top/htop to monitor system resources
- [ ] Start processes in background with &
- [ ] Manage jobs with jobs, fg, bg
- [ ] Send signals with kill (graceful and force)
- [ ] Kill processes by name with killall/pkill
- [ ] Set process priority with nice and renice
- [ ] Use nohup for persistent processes
- [ ] Interpret load averages
- [ ] Find resource-intensive processes
- [ ] Troubleshoot hung or zombie processes

---

## 🚀 Next Steps

Excellent progress! You now understand process management - essential for controlling your Linux system.

In **Chapter 8**, you'll learn:
- User and group management
- Creating and modifying user accounts
- Understanding /etc/passwd and /etc/shadow
- Group administration
- Switching users with su and sudo
- User permissions and access control

**Challenge**: Create a script that monitors system load and sends an alert (to a log file) if load average exceeds your CPU count. The script should run in the background with low priority and persist after logout!

---

*"Processes are the heartbeat of Linux. Master them, and you control the system's pulse."*