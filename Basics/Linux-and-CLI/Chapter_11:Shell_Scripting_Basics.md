# Chapter 11: Shell Scripting Basics

## 🎯 Learning Objectives
By the end of this chapter, you will:
- Create and run shell scripts
- Use variables and operators
- Build conditional logic with if/else and case
- Loop with for, while, and until
- Define and use functions
- Handle user input and script arguments
- Perform arithmetic operations
- Read and process files
- Build practical automation scripts

---

## 11.1 Introduction to Shell Scripts

### What is a Shell Script?

A shell script is a text file containing a sequence of commands that the shell executes. Scripts automate repetitive tasks, save time, and reduce errors.

**Why write scripts?**
- Automate repetitive tasks
- Ensure consistency
- Save complex workflows
- Share procedures with others
- Schedule with cron

---

### Creating Your First Script

```bash
# Create a script
$ cat > hello.sh << 'EOF'
#!/bin/bash
echo "Hello, World!"
echo "Today is $(date +%A)"
echo "Current user: $(whoami)"
EOF

# Make it executable
$ chmod +x hello.sh

# Run it
$ ./hello.sh
Hello, World!
Today is Friday
Current user: john

# Or run with bash explicitly
$ bash hello.sh
```

**The Shebang Line:**
```bash
#!/bin/bash    # Most common — use Bash shell
#!/bin/sh      # POSIX shell (most portable)
#!/bin/zsh     # Zsh shell
#!/usr/bin/env bash   # Portable Bash (recommended)
```

**Why the shebang?**
- Tells the OS which interpreter to use
- Without it, the system guesses (usually /bin/sh)
- Always include it!

---

### Script Structure

```bash
#!/bin/bash
# Script: backup.sh
# Author: John Doe
# Date: 2026-01-31
# Description: Backs up home directory

# === Variables ===
BACKUP_DIR="/tmp/backups"
SOURCE_DIR="$HOME"

# === Functions ===
create_backup() {
    echo "Creating backup..."
    tar -czf "$BACKUP_DIR/backup_$(date +%Y%m%d).tar.gz" "$SOURCE_DIR"
}

# === Main Logic ===
echo "Starting backup script"
create_backup
echo "Backup complete!"
```

---

## 11.2 Variables

### Declaring Variables

```bash
#!/bin/bash

# Simple assignment (no spaces around =!)
name="John"
age=30
city="New York"

# Print variables
echo "Name: $name"
echo "Age: $age"
echo "City: $city"

# WRONG (causes error)
# name = "John"     ← Spaces around = are INVALID!
```

**Variable Types:**
```bash
# String
greeting="Hello World"

# Number (stored as string, but can do math)
count=42
price=19.99

# Empty variable
empty=""

# Variable with spaces (must quote)
full_name="John Doe"
```

---

### Using Variables

```bash
#!/bin/bash

name="Linux"

# Dollar sign to use variable
echo "Hello, $name!"

# Curly braces for clarity
echo "Hello, ${name}!"

# Curly braces needed when adjacent to text
file="report"
echo "Opening ${file}_2026.txt"
# Without braces: $file_2026 would look for variable "file_2026"

# Variable in double quotes (expands)
echo "Name is $name"      # Output: Name is Linux

# Variable in single quotes (literal)
echo 'Name is $name'      # Output: Name is $name

# Backticks or $() for command substitution
today=$(date +%A)
echo "Today is $today"

# Nested substitution
echo "Uppercase: $(echo $name | tr 'a-z' 'A-Z')"
```

---

### Special Variables

```bash
$0      # Script name
$1      # First argument
$2      # Second argument
$@      # All arguments
$#      # Number of arguments
$$      # Current process ID
$?      # Exit status of last command
$!      # PID of last background process
$HOME   # Home directory
$USER   # Current username
$PATH   # Search path for commands
$PWD    # Current working directory
$SHELL  # Current shell
```

**Examples:**
```bash
#!/bin/bash
echo "Script name: $0"
echo "Arguments: $@"
echo "Number of arguments: $#"
echo "First argument: $1"
echo "Second argument: $2"
echo "My PID: $$"
echo "Home: $HOME"
echo "User: $USER"

# Usage:
# $ ./script.sh hello world
# Script name: ./script.sh
# Arguments: hello world
# Number of arguments: 2
# First argument: hello
# Second argument: world
```

---

### Variable Scope

```bash
#!/bin/bash

# Global variable
global_var="I am global"

my_function() {
    # Local variable (only inside function)
    local local_var="I am local"
    
    echo "Inside function:"
    echo "  global_var: $global_var"
    echo "  local_var: $local_var"
    
    # Modify global
    global_var="Modified global"
}

echo "Before function: $global_var"
my_function
echo "After function: $global_var"
echo "local_var outside: $local_var"   # Empty — not accessible

# Output:
# Before function: I am global
# Inside function:
#   global_var: I am global
#   local_var: I am local
# After function: Modified global
# local_var outside:
```

---

### Arrays

```bash
#!/bin/bash

# Declare array
fruits=("apple" "banana" "cherry" "date")

# Access elements
echo "First: ${fruits[0]}"
echo "Second: ${fruits[1]}"
echo "Last: ${fruits[-1]}"

# All elements
echo "All: ${fruits[@]}"

# Number of elements
echo "Count: ${#fruits[@]}"

# Add element
fruits+=("elderberry")

# Loop through array
for fruit in "${fruits[@]}"; do
    echo "- $fruit"
done

# Array from command output
files=($(ls *.txt))
echo "Text files: ${files[@]}"

# Associative array (key-value)
declare -A ages
ages["john"]=30
ages["alice"]=25
ages["bob"]=35

echo "John is ${ages[john]} years old"

# Loop through associative array
for name in "${!ages[@]}"; do
    echo "$name: ${ages[$name]}"
done
```

---

## 11.3 Operators and Expressions

### Arithmetic Operators

```bash
#!/bin/bash

a=10
b=3

# Using $(( )) for arithmetic
echo "Addition: $((a + b))"        # 13
echo "Subtraction: $((a - b))"     # 7
echo "Multiplication: $((a * b))"  # 30
echo "Division: $((a / b))"        # 3 (integer division!)
echo "Modulus: $((a % b))"         # 1
echo "Power: $((a ** b))"          # 1000

# Increment/Decrement
x=5
x=$((x + 1))
echo "Incremented: $x"             # 6

# Or use (( ))
((x++))
echo "After ++: $x"                # 7

((x--))
echo "After --: $x"                # 6

# Compound assignment
y=10
((y += 5))
echo "y += 5: $y"                  # 15
((y *= 2))
echo "y *= 2: $y"                  # 30
((y -= 10))
echo "y -= 10: $y"                 # 20
((y /= 4))
echo "y /= 4: $y"                  # 5
```

**Floating Point Math (use bc):**
```bash
# bc for decimal math
echo "scale=2; 10/3" | bc
3.33

echo "scale=4; 22/7" | bc
3.1428

# Store result
result=$(echo "scale=2; 19.99 * 1.08" | bc)
echo "Total with tax: $result"
21.59
```

---

### Comparison Operators

**For Integers (inside [ ] or [[ ]]):**
```bash
# -eq  Equal
# -ne  Not equal
# -gt  Greater than
# -lt  Less than
# -ge  Greater than or equal
# -le  Less than or equal

if [ $a -eq $b ]; then echo "equal"; fi
if [ $a -gt $b ]; then echo "a is greater"; fi
```

**For Strings (inside [[ ]]):**
```bash
# ==  Equal
# !=  Not equal
# <   Less than (alphabetical)
# >   Greater than (alphabetical)
# =~  Regex match

if [[ "$name" == "John" ]]; then echo "matched"; fi
if [[ "$email" =~ ^[a-zA-Z0-9]+@.*$ ]]; then echo "valid email"; fi
```

---

### Logical Operators

```bash
# && (AND) — both must be true
if [ $a -gt 0 ] && [ $b -gt 0 ]; then
    echo "Both positive"
fi

# || (OR) — either must be true
if [ $a -gt 0 ] || [ $b -gt 0 ]; then
    echo "At least one positive"
fi

# ! (NOT) — inverts result
if ! [ -f "file.txt" ]; then
    echo "File does not exist"
fi

# Inside [[ ]]
if [[ $a -gt 0 && $b -gt 0 ]]; then
    echo "Both positive"
fi
```

---

### String Operators

```bash
# String length
str="Hello World"
echo "Length: ${#str}"             # 11

# Substring extraction
echo "Substr: ${str:0:5}"         # Hello
echo "From 6: ${str:6}"           # World

# String replacement
name="Hello World"
echo "${name/World/Linux}"        # Hello Linux (first match)
echo "${name//o/0}"               # Hell0 W0rld (all matches)

# Default value
unset myvar
echo "${myvar:-default_value}"    # default_value (if unset or empty)
echo "${myvar:=default_value}"    # Sets and uses default
```

---

## 11.4 Conditional Statements

### if / elif / else

**Basic Syntax:**
```bash
if [ condition ]; then
    commands
elif [ another_condition ]; then
    commands
else
    commands
fi
```

**Examples:**
```bash
#!/bin/bash

age=25

if [ $age -lt 18 ]; then
    echo "Minor"
elif [ $age -lt 65 ]; then
    echo "Adult"
else
    echo "Senior"
fi
# Output: Adult

# File existence check
file="config.txt"
if [ -f "$file" ]; then
    echo "$file exists"
elif [ -d "$file" ]; then
    echo "$file is a directory"
else
    echo "$file does not exist"
fi

# String comparison
color="blue"
if [[ "$color" == "red" ]]; then
    echo "Stop!"
elif [[ "$color" == "yellow" ]]; then
    echo "Caution!"
elif [[ "$color" == "green" ]]; then
    echo "Go!"
else
    echo "Unknown color: $color"
fi
```

**One-liner if:**
```bash
# Short form
[ $age -ge 18 ] && echo "Can vote" || echo "Cannot vote"

# Or with if
if [ $age -ge 18 ]; then echo "Can vote"; fi
```

---

### File Test Operators

```bash
-e file    # File exists
-f file    # Is a regular file
-d file    # Is a directory
-r file    # Is readable
-w file    # Is writable
-x file    # Is executable
-s file    # File is not empty (has size > 0)
-z string  # String is empty
-n string  # String is not empty

# Examples
if [ -f "script.sh" ]; then
    echo "File exists"
fi

if [ -d "/var/log" ]; then
    echo "Log directory exists"
fi

if [ -r "/etc/passwd" ]; then
    echo "Can read passwd"
fi

# Combined checks
if [ -f "data.txt" ] && [ -s "data.txt" ]; then
    echo "File exists and is not empty"
fi
```

---

### case Statement

```bash
#!/bin/bash

day="Monday"

case $day in
    Monday|Tuesday|Wednesday|Thursday|Friday)
        echo "Weekday"
        ;;
    Saturday|Sunday)
        echo "Weekend"
        ;;
    *)
        echo "Invalid day"
        ;;
esac

# With variable input
echo "Enter a command (start/stop/restart):"
read command

case $command in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    restart)
        echo "Restarting service..."
        ;;
    *)
        echo "Unknown command: $command"
        echo "Usage: start | stop | restart"
        ;;
esac
```

---

## 11.5 Loops

### for Loop

**Iterate over a list:**
```bash
#!/bin/bash

# Simple list
for color in red green blue yellow; do
    echo "Color: $color"
done

# Number range
for i in $(seq 1 5); do
    echo "Count: $i"
done

# C-style for loop
for ((i=1; i<=5; i++)); do
    echo "Iteration: $i"
done

# Loop over files
for file in *.txt; do
    echo "Processing: $file"
    wc -l "$file"
done

# Loop over array
servers=("web1" "web2" "db1" "db2")
for server in "${servers[@]}"; do
    echo "Checking $server..."
    ping -c 1 "$server" > /dev/null 2>&1
    if [ $? -eq 0 ]; then
        echo "  $server is UP"
    else
        echo "  $server is DOWN"
    fi
done

# Loop with index
fruits=("apple" "banana" "cherry")
for i in "${!fruits[@]}"; do
    echo "$i: ${fruits[$i]}"
done
# Output:
# 0: apple
# 1: banana
# 2: cherry
```

---

### while Loop

```bash
#!/bin/bash

# Basic while loop
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < "input.txt"

# Countdown
n=10
while [ $n -gt 0 ]; do
    echo "$n..."
    ((n--))
    sleep 1
done
echo "Go!"

# Wait for condition
echo "Waiting for file..."
while [ ! -f "ready.txt" ]; do
    echo "Still waiting..."
    sleep 2
done
echo "File found!"

# Infinite loop with break
while true; do
    echo "Enter a number (0 to quit):"
    read num
    if [ "$num" -eq 0 ]; then
        break
    fi
    echo "You entered: $num"
done
echo "Goodbye!"
```

---

### until Loop

```bash
#!/bin/bash

# until = loop until condition is TRUE
# (opposite of while)

count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Wait for service to start
until curl -s http://localhost:8080 > /dev/null 2>&1; do
    echo "Waiting for server..."
    sleep 2
done
echo "Server is ready!"
```

---

### Loop Control: break and continue

```bash
#!/bin/bash

# break — exit the loop
echo "=== Break Example ==="
for i in $(seq 1 10); do
    if [ $i -eq 5 ]; then
        echo "Breaking at $i"
        break
    fi
    echo "i = $i"
done
# Output: 1, 2, 3, 4, Breaking at 5

# continue — skip current iteration
echo "=== Continue Example ==="
for i in $(seq 1 10); do
    if [ $((i % 2)) -eq 0 ]; then
        continue    # Skip even numbers
    fi
    echo "i = $i"
done
# Output: 1, 3, 5, 7, 9

# Nested loop with break
echo "=== Nested Break ==="
for i in 1 2 3; do
    for j in 1 2 3; do
        if [ $j -eq 2 ]; then
            break   # Only breaks inner loop
        fi
        echo "i=$i, j=$j"
    done
done
```

---

## 11.6 Functions

### Defining Functions

```bash
#!/bin/bash

# Define function
greet() {
    echo "Hello, $1!"
}

# Call function
greet "World"
greet "Linux"

# Function with return value
add() {
    local result=$(($1 + $2))
    echo $result       # "Return" by echoing
}

# Capture the return value
sum=$(add 10 20)
echo "Sum: $sum"       # Sum: 30

# Function with local variables
calculate_area() {
    local length=$1
    local width=$2
    local area=$((length * width))
    echo $area
}

result=$(calculate_area 5 10)
echo "Area: $result"   # Area: 50
```

---

### Functions with Exit Codes

```bash
#!/bin/bash

# Function returning success/failure
check_file() {
    local file=$1
    if [ -f "$file" ]; then
        return 0    # Success
    else
        return 1    # Failure
    fi
}

# Use the function
if check_file "config.txt"; then
    echo "Config file found"
else
    echo "Config file missing!"
fi

# Check return code explicitly
check_file "data.txt"
if [ $? -eq 0 ]; then
    echo "File exists"
else
    echo "File not found"
fi
```

---

### Practical Function Examples

```bash
#!/bin/bash

# Color output function
print_green() { echo -e "\033[0;32m$1\033[0m"; }
print_red()   { echo -e "\033[0;31m$1\033[0m"; }
print_yellow() { echo -e "\033[1;33m$1\033[0m"; }

# Logging function
LOG_FILE="/var/log/myapp.log"
log() {
    local level=$1
    local message=$2
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $message" >> $LOG_FILE
    echo "[$level] $message"
}

# Usage
log "INFO" "Script started"
log "ERROR" "Something went wrong"

# Reusable validation
validate_email() {
    local email=$1
    if [[ $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        return 0
    else
        return 1
    fi
}

# Usage
email="user@example.com"
if validate_email "$email"; then
    print_green "Valid email: $email"
else
    print_red "Invalid email: $email"
fi
```

---

## 11.7 Reading Input

### read Command

```bash
#!/bin/bash

# Basic read
echo "Enter your name:"
read name
echo "Hello, $name!"

# Read with prompt (no newline)
read -p "Enter your age: " age
echo "You are $age years old"

# Read with default value
read -p "Enter city [New York]: " city
city=${city:-"New York"}
echo "City: $city"

# Read with timeout
read -t 5 -p "Enter value (5 sec timeout): " value
if [ -z "$value" ]; then
    echo "Timeout! Using default."
    value="default"
fi

# Read password (hidden input)
read -sp "Enter password: " password
echo ""    # Add newline after hidden input
echo "Password length: ${#password}"

# Read multiple values
read -p "Enter first and last name: " first last
echo "First: $first, Last: $last"
```

---

### Script Arguments

```bash
#!/bin/bash
# Usage: ./script.sh <name> <age>

# Check argument count
if [ $# -lt 2 ]; then
    echo "Usage: $0 <name> <age>"
    exit 1
fi

name=$1
age=$2

echo "Name: $name"
echo "Age: $age"

# Process all arguments
echo "All arguments:"
for arg in "$@"; do
    echo "  - $arg"
done

# Shift arguments
while [ $# -gt 0 ]; do
    echo "Processing: $1"
    shift    # Move arguments left
done
```

**Named Arguments with getopts:**
```bash
#!/bin/bash
# Usage: ./script.sh -n name -a age -c city

while getopts "n:a:c:" opt; do
    case $opt in
        n) name="$OPTARG" ;;
        a) age="$OPTARG"  ;;
        c) city="$OPTARG" ;;
        *) echo "Usage: $0 -n name -a age -c city"; exit 1 ;;
    esac
done

echo "Name: $name"
echo "Age: $age"
echo "City: $city"

# Usage:
# $ ./script.sh -n "John" -a 30 -c "NYC"
```

---

## 11.8 Error Handling

### Exit Codes

```bash
#!/bin/bash

# exit 0 = success
# exit 1 = general error
# exit 2 = misuse of shell builtin

# Check last command's exit code
ls /nonexistent 2>/dev/null
echo "Exit code: $?"    # 2 (error)

ls /tmp 2>/dev/null
echo "Exit code: $?"    # 0 (success)

# Exit script with code
if [ ! -f "required.txt" ]; then
    echo "Error: required.txt not found"
    exit 1
fi
```

---

### set Options for Error Handling

```bash
#!/bin/bash

# Exit immediately on error
set -e

# Treat unset variables as error
set -u

# Pipe fails if any command fails
set -o pipefail

# Combine all (best practice!)
set -euo pipefail

# Show each command before executing (debug)
set -x

# Example with error handling
set -euo pipefail

echo "Starting script..."
cp source.txt dest.txt    # Script exits here if cp fails
echo "Copy successful"    # Only runs if cp succeeded
```

---

### Trap — Cleanup on Exit

```bash
#!/bin/bash

# Define cleanup function
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile.txt
    echo "Done"
}

# Run cleanup on exit (any exit)
trap cleanup EXIT

# Run cleanup on Ctrl+C
trap cleanup INT

# Combined
trap cleanup EXIT INT TERM

# Create temp file
echo "Working..." > /tmp/tempfile.txt
echo "Do work here..."
# cleanup runs automatically when script exits
```

---

## 💡 Key Concepts Summary

1. **Scripts** start with shebang (#!/bin/bash) and need +x permission
2. **Variables** use $ to reference, no spaces around =
3. **Arithmetic** uses $(( )) or (( )); use bc for decimals
4. **Conditionals**: if/elif/else/fi and case/esac
5. **Loops**: for, while, until with break and continue
6. **Functions**: Define with name() { }, call with name
7. **Input**: read for user input, $1 $2 for arguments
8. **Error handling**: set -euo pipefail, trap for cleanup
9. **Arrays** declared with () and accessed with ${array[index]}
10. **Special variables**: $0, $1, $@, $#, $?, $$

---

## 🔥 Real-World Use Cases

### Use Case 1: System Backup Script

```bash
#!/bin/bash
# backup.sh — Automated backup with logging

set -euo pipefail

# Configuration
BACKUP_DIR="/backup"
SOURCE_DIR="/home"
LOG_FILE="/var/log/backup.log"
MAX_BACKUPS=7    # Keep last 7 backups
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

# Cleanup function
cleanup() {
    if [ $? -ne 0 ]; then
        log "ERROR: Backup failed!"
    fi
}
trap cleanup EXIT

# Start
log "Starting backup..."

# Create backup directory
mkdir -p "$BACKUP_DIR"

# Create backup
BACKUP_FILE="${BACKUP_DIR}/backup_${TIMESTAMP}.tar.gz"
log "Creating backup: $BACKUP_FILE"
tar -czf "$BACKUP_FILE" "$SOURCE_DIR" 2>> "$LOG_FILE"

# Check size
SIZE=$(du -h "$BACKUP_FILE" | cut -f1)
log "Backup size: $SIZE"

# Remove old backups
log "Cleaning old backups..."
cd "$BACKUP_DIR"
ls -t backup_*.tar.gz | tail -n +$((MAX_BACKUPS + 1)) | while read old; do
    log "Removing old backup: $old"
    rm "$old"
done

log "Backup completed successfully!"
```

### Use Case 2: User Management Script

```bash
#!/bin/bash
# create_users.sh — Batch user creation

set -euo pipefail

# Check if running as root
if [ $EUID -ne 0 ]; then
    echo "Error: This script must be run as root"
    exit 1
fi

# Check if users file exists
USERS_FILE="${1:-users.txt}"
if [ ! -f "$USERS_FILE" ]; then
    echo "Error: Users file not found: $USERS_FILE"
    echo "Format: username:fullname:groups"
    exit 1
fi

# Process each user
while IFS=: read -r username fullname groups; do
    # Skip comments and empty lines
    [[ "$username" =~ ^#.*$ ]] && continue
    [[ -z "$username" ]] && continue
    
    echo "Processing user: $username"
    
    # Check if user exists
    if id "$username" &> /dev/null; then
        echo "  User $username already exists, skipping"
        continue
    fi
    
    # Create user
    useradd -m -s /bin/bash -c "$fullname" "$username"
    
    # Add to groups
    if [ -n "$groups" ]; then
        usermod -aG "$groups" "$username"
    fi
    
    # Set temporary password
    passwd -e "$username" <<< "${username}:${username}123"
    
    echo "  User $username created successfully"
done < "$USERS_FILE"

echo "All users processed!"

# users.txt format:
# # Comments start with #
# john:John Doe:sudo,developers
# alice:Alice Smith:developers
# bob:Bob Johnson:designers
```

### Use Case 3: Health Check Script

```bash
#!/bin/bash
# health_check.sh — Monitor multiple services

set -uo pipefail

# Configuration
SERVICES=("nginx" "postgresql" "redis" "ssh")
PORTS=(80 5432 6379 22)
LOG_FILE="/var/log/health_check.log"

# Colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

check_service() {
    local service=$1
    if systemctl is-active --quiet "$service" 2>/dev/null; then
        echo -e "${GREEN}✓ $service${NC}"
        return 0
    else
        echo -e "${RED}✗ $service${NC}"
        return 1
    fi
}

check_port() {
    local port=$1
    local service=$2
    if ss -tlnp | grep -q ":${port} "; then
        echo -e "  ${GREEN}Port $port open${NC}"
        return 0
    else
        echo -e "  ${RED}Port $port closed${NC}"
        return 1
    fi
}

# Main
echo "=== System Health Check — $(date) ==="
echo ""

failures=0

for i in "${!SERVICES[@]}"; do
    service="${SERVICES[$i]}"
    port="${PORTS[$i]}"
    
    check_service "$service"
    if [ $? -ne 0 ]; then
        ((failures++))
    fi
    
    check_port "$port" "$service"
    if [ $? -ne 0 ]; then
        ((failures++))
    fi
    echo ""
done

# Summary
echo "=========================="
if [ $failures -eq 0 ]; then
    echo -e "${GREEN}All checks passed!${NC}"
else
    echo -e "${RED}$failures check(s) failed${NC}"
fi

# Log results
echo "[$(date)] Failures: $failures" >> "$LOG_FILE"
exit $failures
```

---

## 🧪 Practice Problems

### Beginner Level

**Problem 1**: Write a script that prints your name, current date, and hostname.

**Problem 2**: Write a script that takes a number as argument and prints whether it's even or odd.

**Problem 3**: Write a script that loops from 1 to 10 and prints each number.

**Problem 4**: Write a script that asks for two numbers and prints their sum.

**Problem 5**: Write a script that checks if a given file exists.

### Intermediate Level

**Problem 6**: Write a script that takes a directory as argument, counts files, and displays total size.

**Problem 7**: Write a script that reads a file line by line and numbers each line.

**Problem 8**: Write a script with a function that calculates factorial of a number.

**Problem 9**: Write a script that uses a case statement to convert month numbers (1-12) to month names.

**Problem 10**: Write a script that backs up a specified file with a timestamp.

### Advanced Level

**Problem 11**: Write a script that monitors disk usage and alerts if any partition exceeds 80%.

**Problem 12**: Write a script that processes a CSV file, calculates averages of a numeric column, and generates a report.

**Problem 13**: Write a script that accepts named arguments (-i input -o output -v verbose) using getopts.

**Problem 14**: Write a script that implements a simple calculator supporting +, -, *, / operations.

**Problem 15**: Write a script that recursively finds duplicate files (same size and checksum) in a directory.

---

## 📝 Solutions

### Beginner Solutions

**Solution 1**:
```bash
#!/bin/bash
# info.sh — Display system information

echo "Name: $(whoami)"
echo "Date: $(date '+%Y-%m-%d %H:%M:%S')"
echo "Hostname: $(hostname)"
echo "OS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d'"' -f2)"

# Usage: chmod +x info.sh && ./info.sh
```

**Solution 2**:
```bash
#!/bin/bash
# even_odd.sh — Check if number is even or odd

if [ $# -ne 1 ]; then
    echo "Usage: $0 <number>"
    exit 1
fi

num=$1

# Validate input
if ! [[ "$num" =~ ^[0-9]+$ ]]; then
    echo "Error: Please enter a valid number"
    exit 1
fi

if [ $((num % 2)) -eq 0 ]; then
    echo "$num is EVEN"
else
    echo "$num is ODD"
fi

# Usage:
# $ ./even_odd.sh 42 → 42 is EVEN
# $ ./even_odd.sh 7  → 7 is ODD
```

**Solution 3**:
```bash
#!/bin/bash
# count.sh — Count from 1 to 10

echo "Counting from 1 to 10:"
for i in $(seq 1 10); do
    echo "  $i"
done
echo "Done!"

# Or with C-style loop
# for ((i=1; i<=10; i++)); do
#     echo "  $i"
# done
```

**Solution 4**:
```bash
#!/bin/bash
# add.sh — Add two numbers

read -p "Enter first number: " a
read -p "Enter second number: " b

# Validate
if ! [[ "$a" =~ ^[0-9]+$ ]] || ! [[ "$b" =~ ^[0-9]+$ ]]; then
    echo "Error: Both inputs must be numbers"
    exit 1
fi

sum=$((a + b))
echo "Result: $a + $b = $sum"
```

**Solution 5**:
```bash
#!/bin/bash
# check_file.sh — Check if file exists

if [ $# -ne 1 ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

file=$1

if [ -f "$file" ]; then
    echo "✓ '$file' exists"
    echo "  Size: $(du -h "$file" | cut -f1)"
    echo "  Modified: $(stat -c %y "$file" | cut -d. -f1)"
elif [ -d "$file" ]; then
    echo "✓ '$file' exists (directory)"
    echo "  Files inside: $(ls "$file" | wc -l)"
else
    echo "✗ '$file' does not exist"
    exit 1
fi

# Usage:
# $ ./check_file.sh /etc/passwd
# ✓ '/etc/passwd' exists
#   Size: 2.8K
#   Modified: 2026-01-25 15:30:00
```

### Intermediate Solutions

**Solution 6**:
```bash
#!/bin/bash
# dir_stats.sh — Directory statistics

if [ $# -ne 1 ]; then
    echo "Usage: $0 <directory>"
    exit 1
fi

dir=$1

if [ ! -d "$dir" ]; then
    echo "Error: '$dir' is not a directory"
    exit 1
fi

# Count files and directories
total_files=$(find "$dir" -type f | wc -l)
total_dirs=$(find "$dir" -type d | wc -l)
total_size=$(du -sh "$dir" | cut -f1)

# Largest file
largest=$(find "$dir" -type f -exec ls -lS {} \; 2>/dev/null | head -1 | awk '{print $5, $9}')

echo "Directory: $dir"
echo "========================"
echo "Files:      $total_files"
echo "Directories: $((total_dirs - 1))"
echo "Total Size: $total_size"
echo "Largest:    $largest"
```

**Solution 7**:
```bash
#!/bin/bash
# number_lines.sh — Number lines in a file

if [ $# -ne 1 ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

file=$1
if [ ! -f "$file" ]; then
    echo "Error: File '$file' not found"
    exit 1
fi

line_num=0
while IFS= read -r line; do
    ((line_num++))
    printf "%4d | %s\n" "$line_num" "$line"
done < "$file"

echo "========================"
echo "Total lines: $line_num"
```

**Solution 8**:
```bash
#!/bin/bash
# factorial.sh — Calculate factorial

factorial() {
    local n=$1
    
    if [ $n -lt 0 ]; then
        echo "Error: Negative number"
        return 1
    fi
    
    if [ $n -le 1 ]; then
        echo 1
        return 0
    fi
    
    local result=1
    for ((i=2; i<=n; i++)); do
        result=$((result * i))
    done
    echo $result
}

# Get input
read -p "Enter a number: " num

# Validate
if ! [[ "$num" =~ ^[0-9]+$ ]]; then
    echo "Error: Please enter a valid number"
    exit 1
fi

result=$(factorial $num)
echo "$num! = $result"

# Test:
# 0! = 1
# 5! = 120
# 10! = 3628800
```

**Solution 9**:
```bash
#!/bin/bash
# month.sh — Convert month number to name

if [ $# -ne 1 ]; then
    echo "Usage: $0 <month_number (1-12)>"
    exit 1
fi

month=$1

case $month in
    1)  echo "January"   ;;
    2)  echo "February"  ;;
    3)  echo "March"     ;;
    4)  echo "April"     ;;
    5)  echo "May"       ;;
    6)  echo "June"      ;;
    7)  echo "July"      ;;
    8)  echo "August"    ;;
    9)  echo "September" ;;
    10) echo "October"   ;;
    11) echo "November"  ;;
    12) echo "December"  ;;
    *)  echo "Error: Invalid month number. Use 1-12." ; exit 1 ;;
esac
```

**Solution 10**:
```bash
#!/bin/bash
# backup_file.sh — Backup a file with timestamp

if [ $# -ne 1 ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

source_file=$1
if [ ! -f "$source_file" ]; then
    echo "Error: File '$source_file' not found"
    exit 1
fi

# Create backup with timestamp
timestamp=$(date +%Y%m%d_%H%M%S)
backup_file="${source_file}.backup_${timestamp}"

cp "$source_file" "$backup_file"

if [ $? -eq 0 ]; then
    echo "Backup created: $backup_file"
    echo "Original size: $(du -h "$source_file" | cut -f1)"
    echo "Backup size:   $(du -h "$backup_file" | cut -f1)"
else
    echo "Error: Backup failed!"
    exit 1
fi
```

### Advanced Solutions

**Solution 11**:
```bash
#!/bin/bash
# disk_monitor.sh — Monitor disk usage and alert

set -euo pipefail

THRESHOLD=80
ALERT_FILE="/var/log/disk_alerts.log"

echo "=== Disk Usage Monitor ==="
echo "Threshold: ${THRESHOLD}%"
echo ""

alert_triggered=0

# Get disk usage (skip header)
df -h | tail -n +2 | while IFS= read -r line; do
    mount=$(echo "$line" | awk '{print $6}')
    usage=$(echo "$line" | awk '{print $5}' | tr -d '%')
    size=$(echo "$line" | awk '{print $2}')
    used=$(echo "$line" | awk '{print $3}')
    
    if [ "$usage" -gt "$THRESHOLD" ]; then
        echo -e "\033[0;31m⚠ $mount: ${usage}% used ($used / $size)\033[0m"
        echo "[$(date)] ALERT: $mount at ${usage}%" >> "$ALERT_FILE"
    else
        echo -e "\033[0;32m✓ $mount: ${usage}% used ($used / $size)\033[0m"
    fi
done

# Usage:
# chmod +x disk_monitor.sh && ./disk_monitor.sh
# crontab -e → 0 * * * * /path/to/disk_monitor.sh
```

**Solution 12**:
```bash
#!/bin/bash
# csv_report.sh — Process CSV and generate report
# Usage: ./csv_report.sh data.csv <column_number>

set -euo pipefail

if [ $# -ne 2 ]; then
    echo "Usage: $0 <csv_file> <numeric_column>"
    exit 1
fi

file=$1
col=$2

if [ ! -f "$file" ]; then
    echo "Error: File '$file' not found"
    exit 1
fi

# Get header
header=$(head -1 "$file" | cut -d',' -f"$col")
echo "=== Report for: $header ==="
echo ""

# Extract numeric column (skip header), filter valid numbers
values=$(tail -n +2 "$file" | cut -d',' -f"$col" | grep -E '^[0-9.]+$')

if [ -z "$values" ]; then
    echo "Error: No valid numeric data in column $col"
    exit 1
fi

# Calculate statistics
count=$(echo "$values" | wc -l)
sum=$(echo "$values" | awk '{sum+=$1} END {print sum}')
avg=$(echo "$values" | awk '{sum+=$1} END {printf "%.2f", sum/NR}')
min=$(echo "$values" | sort -n | head -1)
max=$(echo "$values" | sort -n | tail -1)

echo "Count:   $count"
echo "Sum:     $sum"
echo "Average: $avg"
echo "Min:     $min"
echo "Max:     $max"
echo ""
echo "Distribution:"
echo "$values" | sort -n | uniq -c | sort -rn | head -10
```

**Solution 13**:
```bash
#!/bin/bash
# process_file.sh — Process file with named arguments
# Usage: ./process_file.sh -i input.txt -o output.txt [-v]

set -euo pipefail

# Default values
INPUT=""
OUTPUT=""
VERBOSE=false

# Parse arguments
while getopts "i:o:v" opt; do
    case $opt in
        i) INPUT="$OPTARG"   ;;
        o) OUTPUT="$OPTARG"  ;;
        v) VERBOSE=true      ;;
        *)
            echo "Usage: $0 -i <input> -o <output> [-v]"
            exit 1
            ;;
    esac
done

# Validate required args
if [ -z "$INPUT" ] || [ -z "$OUTPUT" ]; then
    echo "Error: Both -i and -o are required"
    echo "Usage: $0 -i <input> -o <output> [-v]"
    exit 1
fi

# Validate input file
if [ ! -f "$INPUT" ]; then
    echo "Error: Input file '$INPUT' not found"
    exit 1
fi

# Process
$VERBOSE && echo "Input:  $INPUT"
$VERBOSE && echo "Output: $OUTPUT"
$VERBOSE && echo "Processing..."

# Example: uppercase transformation
tr 'a-z' 'A-Z' < "$INPUT" > "$OUTPUT"

$VERBOSE && echo "Done! Lines processed: $(wc -l < "$OUTPUT")"
echo "Output written to: $OUTPUT"

# Usage:
# $ ./process_file.sh -i input.txt -o output.txt -v
```

**Solution 14**:
```bash
#!/bin/bash
# calculator.sh — Simple calculator

set -euo pipefail

calculate() {
    local a=$1
    local op=$2
    local b=$3

    case $op in
        +) echo "scale=2; $a + $b" | bc ;;
        -) echo "scale=2; $a - $b" | bc ;;
        *) echo "scale=2; $a * $b" | bc ;;
        /) 
            if [ "$b" = "0" ]; then
                echo "Error: Division by zero"
                return 1
            fi
            echo "scale=2; $a / $b" | bc 
            ;;
        *)
            echo "Error: Unknown operator '$op'"
            return 1
            ;;
    esac
}

echo "=== Simple Calculator ==="
echo "Operators: + - * /"
echo "Type 'quit' to exit"
echo ""

while true; do
    read -p "Enter expression (e.g., 10 + 5): " input
    
    # Check for quit
    [[ "$input" == "quit" ]] && break
    
    # Parse input
    a=$(echo "$input" | awk '{print $1}')
    op=$(echo "$input" | awk '{print $2}')
    b=$(echo "$input" | awk '{print $3}')
    
    # Validate
    if [ -z "$a" ] || [ -z "$op" ] || [ -z "$b" ]; then
        echo "Invalid input. Format: number operator number"
        continue
    fi
    
    # Calculate
    result=$(calculate "$a" "$op" "$b" 2>&1)
    if [ $? -eq 0 ]; then
        echo "Result: $a $op $b = $result"
    else
        echo "$result"
    fi
    echo ""
done

echo "Goodbye!"
```

**Solution 15**:
```bash
#!/bin/bash
# find_duplicates.sh — Find duplicate files

set -euo pipefail

if [ $# -ne 1 ]; then
    echo "Usage: $0 <directory>"
    exit 1
fi

dir=$1
if [ ! -d "$dir" ]; then
    echo "Error: '$dir' is not a directory"
    exit 1
fi

echo "Scanning for duplicate files in: $dir"
echo "================================================"

# Step 1: Find files with same size
find "$dir" -type f -printf '%s %p\n' 2>/dev/null | \
    sort -n | \
    awk '{size[$1]=size[$1] ? size[$1]" "$2 : $2; count[$1]++} 
         END {for(s in count) if(count[s]>1) print size[s]}' | \
while IFS= read -r files; do
    # Step 2: For same-size files, compare checksums
    declare -A checksums
    for file in $files; do
        checksum=$(md5sum "$file" | cut -d' ' -f1)
        if [ -n "${checksums[$checksum]+x}" ]; then
            echo ""
            echo "DUPLICATE FOUND (MD5: $checksum):"
            echo "  Original: ${checksums[$checksum]}"
            echo "  Duplicate: $file"
            echo "  Size: $(du -h "$file" | cut -f1)"
        else
            checksums[$checksum]="$file"
        fi
    done
    unset checksums
done

echo ""
echo "Scan complete!"

# Usage:
# chmod +x find_duplicates.sh
# ./find_duplicates.sh /home/john/documents
```

---

## 🎓 Learning Tips

### Script Writing Checklist
```
1. Start with shebang: #!/bin/bash
2. Add comments explaining purpose
3. Use set -euo pipefail for safety
4. Validate inputs early
5. Use meaningful variable names
6. Add error messages
7. Test with edge cases
8. Make executable: chmod +x script.sh
```

### Variable Naming Conventions
```bash
CONSTANTS="ALL_CAPS"          # Constants
regular_var="snake_case"      # Variables  
MY_FUNCTION_var="mixed"       # Function-related
```

### Common Pitfalls
```bash
# WRONG — spaces around =
name = "John"

# RIGHT
name="John"

# WRONG — unquoted variable (breaks with spaces)
echo $name

# RIGHT
echo "$name"

# WRONG — word splitting in loop
for file in $(find . -name "*.txt"); do

# RIGHT — handles spaces
while IFS= read -r file; do
done < <(find . -name "*.txt")
```

### Debugging Scripts
```bash
# Show commands as they execute
bash -x script.sh

# Or add in script
set -x

# Check syntax without running
bash -n script.sh

# Verbose mode
bash -v script.sh
```

---

## 🔗 Connections to Future Chapters

- **Chapter 12**: Advanced scripting (arrays, regex, complex logic)
- **Chapter 13**: Monitoring scripts using shell automation
- **Chapter 14**: Service management scripts
- **Chapter 21**: Troubleshooting with diagnostic scripts

---

## 📚 Additional Resources

### Man Pages
```bash
man bash
man test      # [ ] conditions
man getopts
```

### Practice
```bash
# Built-in tutorial
$ vimtutor     # Learn vim
```

### Further Reading
- Bash Manual: https://www.gnu.org/software/bash/manual/
- Shell Scripting Guide: https://wiki.bash-hackers.org/
- Google Shell Style Guide: https://google.github.io/styleguide/shellguide.html

---

## ✅ Self-Assessment Checklist

Before moving to Chapter 12, ensure you can:
- [ ] Create and run shell scripts
- [ ] Use variables and special variables ($1, $@, $?)
- [ ] Perform arithmetic with $(( )) and bc
- [ ] Write if/elif/else conditionals
- [ ] Use case statements
- [ ] Write for, while, and until loops
- [ ] Use break and continue
- [ ] Define and call functions
- [ ] Read user input with read
- [ ] Handle script arguments with $1, $@ and getopts
- [ ] Use arrays (indexed and associative)
- [ ] Implement basic error handling
- [ ] Use trap for cleanup
- [ ] Debug scripts with set -x

---

## 🚀 Next Steps

Excellent work! You can now write powerful shell scripts to automate Linux tasks.

In **Chapter 12**, you'll learn:
- Advanced array manipulation
- Complex string processing in scripts
- Regular expressions in scripts
- Advanced error handling and logging
- Script optimization and best practices
- Building production-ready automation tools

**Challenge**: Write a complete backup manager script that supports multiple backup targets, rotation policies, compression, and generates a daily report!

---

*"Shell scripting turns you from a Linux user into a Linux power user. Automate everything!"*