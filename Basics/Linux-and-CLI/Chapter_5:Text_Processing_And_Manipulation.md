# Chapter 5: Text Processing and Manipulation

## 🎯 Learning Objectives
By the end of this chapter, you will:
- Master basic text tools (echo, cat, sort, uniq)
- Search text effectively with grep and regular expressions
- Transform text with tr, cut, paste, and join
- Use sed for stream editing
- Perform pattern scanning with awk
- Edit text files with nano and vim
- Build powerful text-processing pipelines

---

## 5.1 Basic Text Tools

### echo - Display Text

**Basic Usage:**
```bash
# Simple output
$ echo "Hello, World!"
Hello, World!

# Without newline
$ echo -n "No newline"
No newline$

# Enable escape sequences
$ echo -e "Line 1\nLine 2\tTabbed"
Line 1
Line 2    Tabbed

# Print to file
$ echo "Some text" > file.txt

# Append to file
$ echo "More text" >> file.txt
```

**Escape Sequences (with -e):**
```bash
\n  # Newline
\t  # Tab
\\  # Backslash
\"  # Double quote
\a  # Alert (beep)

$ echo -e "Column1\tColumn2\nValue1\tValue2"
Column1    Column2
Value1     Value2
```

**Variable Expansion:**
```bash
$ name="John"
$ echo "Hello, $name"
Hello, John

$ echo 'Hello, $name'
Hello, $name    # Single quotes prevent expansion

$ echo "Today is $(date +%A)"
Today is Friday
```

---

### cat - Concatenate and Display

**Basic Usage:**
```bash
# Display file
$ cat file.txt

# Display multiple files
$ cat file1.txt file2.txt

# Number lines
$ cat -n file.txt
     1  First line
     2  Second line
     3  Third line

# Number non-empty lines only
$ cat -b file.txt

# Show ends of lines
$ cat -E file.txt
First line$
Second line$

# Show tabs
$ cat -T file.txt
First^Iline  # ^I represents tab

# Show all non-printing characters
$ cat -A file.txt
```

**Creating Files:**
```bash
# Create file from input
$ cat > newfile.txt
Type your content here
Press Ctrl+D to finish

# Concatenate files into new file
$ cat file1.txt file2.txt > combined.txt

# Append to file
$ cat file3.txt >> combined.txt
```

---

### tac - Reverse cat

```bash
# Display file in reverse (last line first)
$ cat file.txt
Line 1
Line 2
Line 3

$ tac file.txt
Line 3
Line 2
Line 1
```

---

### rev - Reverse Lines

```bash
# Reverse each line character by character
$ echo "Hello World" | rev
dlroW olleH

$ cat file.txt
Hello World
Linux is awesome

$ rev file.txt
dlroW olleH
emosewa si xuniL
```

---

### sort - Sort Lines

**Basic Sorting:**
```bash
# Create unsorted file
$ cat names.txt
Charlie
Alice
Bob

# Sort alphabetically
$ sort names.txt
Alice
Bob
Charlie

# Sort in reverse
$ sort -r names.txt
Charlie
Bob
Alice
```

**Numeric Sorting:**
```bash
$ cat numbers.txt
100
20
3

# Alphabetic sort (wrong for numbers)
$ sort numbers.txt
100
20
3

# Numeric sort
$ sort -n numbers.txt
3
20
100

# Reverse numeric
$ sort -rn numbers.txt
100
20
3
```

**Advanced Sorting:**
```bash
# Sort by column (field)
$ cat data.txt
John 25 Developer
Alice 30 Manager
Bob 25 Designer

# Sort by 2nd field (age)
$ sort -k2 -n data.txt
John 25 Developer
Bob 25 Designer
Alice 30 Manager

# Sort by 3rd field (job)
$ sort -k3 data.txt
Bob 25 Designer
John 25 Developer
Alice 30 Manager

# Case-insensitive sort
$ sort -f file.txt

# Remove duplicates while sorting
$ sort -u file.txt

# Check if file is sorted
$ sort -c file.txt
```

---

### uniq - Remove Duplicates

**Basic Usage:**
```bash
$ cat duplicates.txt
apple
banana
apple
cherry
banana

# Remove consecutive duplicates (must sort first!)
$ sort duplicates.txt | uniq
apple
banana
cherry

# Count occurrences
$ sort duplicates.txt | uniq -c
      2 apple
      2 banana
      1 cherry

# Show only duplicates
$ sort duplicates.txt | uniq -d
apple
banana

# Show only unique lines
$ sort duplicates.txt | uniq -u
cherry

# Ignore case
$ sort -f file.txt | uniq -i
```

**Common Pattern:**
```bash
# Count unique values
$ cut -d: -f7 /etc/passwd | sort | uniq -c | sort -rn
     25 /bin/bash
     18 /usr/sbin/nologin
      3 /bin/sh
      1 /bin/sync
```

---

## 5.2 Text Searching with grep

### grep - Global Regular Expression Print

**Basic Usage:**
```bash
# Search for pattern
$ grep "word" file.txt

# Case-insensitive search
$ grep -i "word" file.txt

# Show line numbers
$ grep -n "word" file.txt
15:This line contains word

# Invert match (show lines NOT containing pattern)
$ grep -v "word" file.txt

# Count matching lines
$ grep -c "word" file.txt
5

# Show only matching part
$ grep -o "word" file.txt
```

**Multiple Files:**
```bash
# Search in multiple files
$ grep "error" *.log

# Show filename with matches
$ grep -H "error" *.log
file1.log:Error occurred
file2.log:Error in module

# Suppress filename
$ grep -h "error" *.log

# List only filenames with matches
$ grep -l "error" *.log
file1.log
file2.log

# List filenames WITHOUT matches
$ grep -L "error" *.log
file3.log
```

**Recursive Search:**
```bash
# Search in current directory and subdirectories
$ grep -r "TODO" .

# Recursive with line numbers
$ grep -rn "TODO" .

# Exclude certain files/directories
$ grep -r "TODO" --exclude="*.log" .
$ grep -r "TODO" --exclude-dir="node_modules" .
```

**Context Options:**
```bash
# Show 3 lines before match
$ grep -B 3 "error" file.txt

# Show 3 lines after match
$ grep -A 3 "error" file.txt

# Show 3 lines before and after
$ grep -C 3 "error" file.txt
# or
$ grep -3 "error" file.txt
```

### Regular Expressions with grep

**Basic Patterns:**
```bash
# Match beginning of line
$ grep "^Start" file.txt

# Match end of line
$ grep "end$" file.txt

# Match any single character
$ grep "c.t" file.txt    # Matches: cat, cot, cut

# Match any character from set
$ grep "c[aou]t" file.txt    # Matches: cat, cot, cut

# Match range
$ grep "[0-9]" file.txt      # Any digit
$ grep "[a-z]" file.txt      # Any lowercase letter
$ grep "[A-Z]" file.txt      # Any uppercase letter

# Match zero or more
$ grep "ab*c" file.txt       # ac, abc, abbc, abbbc

# Match one or more (use -E for extended regex)
$ grep -E "ab+c" file.txt    # abc, abbc, abbbc (not ac)

# Match zero or one
$ grep -E "ab?c" file.txt    # ac, abc (not abbc)
```

**Extended Regular Expressions (-E or egrep):**
```bash
# Alternation (OR)
$ grep -E "cat|dog" file.txt
# or
$ egrep "cat|dog" file.txt

# Grouping
$ grep -E "(cat|dog)s" file.txt    # cats or dogs

# Repetition
$ grep -E "a{3}" file.txt          # aaa (exactly 3)
$ grep -E "a{2,4}" file.txt        # aa, aaa, aaaa (2 to 4)
$ grep -E "a{2,}" file.txt         # aa, aaa, aaaa... (2 or more)

# Word boundaries
$ grep -w "cat" file.txt           # Matches "cat" but not "catch"
$ grep -E "\bcat\b" file.txt       # Same as -w
```

**Practical Examples:**
```bash
# Find email addresses
$ grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt

# Find IP addresses
$ grep -E "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" file.txt

# Find phone numbers (XXX-XXX-XXXX)
$ grep -E "\b[0-9]{3}-[0-9]{3}-[0-9]{4}\b" file.txt

# Find lines with numbers
$ grep "[0-9]" file.txt

# Find empty lines
$ grep "^$" file.txt

# Find lines that start with #
$ grep "^#" file.txt

# Find TODO comments in code
$ grep -rn "TODO" --include="*.py" .
```

---

## 5.3 Text Transformation

### tr - Translate or Delete Characters

**Basic Usage:**
```bash
# Convert lowercase to uppercase
$ echo "hello" | tr 'a-z' 'A-Z'
HELLO

# Convert uppercase to lowercase
$ echo "HELLO" | tr 'A-Z' 'a-z'
hello

# Replace characters
$ echo "hello world" | tr 'hw' 'HW'
Hello World

# Delete characters
$ echo "hello123world" | tr -d '0-9'
helloworld

# Delete spaces
$ echo "hello world" | tr -d ' '
helloworld

# Squeeze repeated characters
$ echo "hellooo    world" | tr -s 'o '
helo world
```

**Practical Examples:**
```bash
# Convert Windows line endings to Unix
$ tr -d '\r' < windows.txt > unix.txt

# Remove all punctuation
$ echo "Hello, World!" | tr -d '[:punct:]'
Hello World

# Replace spaces with newlines
$ echo "one two three" | tr ' ' '\n'
one
two
three

# Create a character frequency count
$ cat file.txt | tr -cs 'A-Za-z' '\n' | sort | uniq -c | sort -rn
```

**Character Classes:**
```bash
[:alnum:]   # Alphanumeric
[:alpha:]   # Alphabetic
[:digit:]   # Digits
[:lower:]   # Lowercase
[:upper:]   # Uppercase
[:space:]   # Whitespace
[:punct:]   # Punctuation

# Example: Remove all non-alphanumeric
$ echo "abc-123_xyz!" | tr -cd '[:alnum:]'
abc123xyz
```

---

### cut - Extract Columns

**By Character Position:**
```bash
# Extract characters 1-5
$ echo "Hello World" | cut -c 1-5
Hello

# Extract characters 7 to end
$ echo "Hello World" | cut -c 7-
World

# Extract multiple ranges
$ echo "Hello World" | cut -c 1-5,7-11
HelloWorld
```

**By Field (Delimiter):**
```bash
# Create CSV file
$ cat data.csv
John,25,Developer
Alice,30,Manager
Bob,25,Designer

# Extract 1st field (name)
$ cut -d ',' -f 1 data.csv
John
Alice
Bob

# Extract 2nd field (age)
$ cut -d ',' -f 2 data.csv
25
30
25

# Extract multiple fields
$ cut -d ',' -f 1,3 data.csv
John,Developer
Alice,Manager
Bob,Designer

# Extract range of fields
$ cut -d ',' -f 1-2 data.csv
John,25
Alice,30
Bob,25
```

**Practical Examples:**
```bash
# Extract usernames from /etc/passwd
$ cut -d ':' -f 1 /etc/passwd
root
daemon
bin
...

# Extract home directories
$ cut -d ':' -f 6 /etc/passwd

# Extract first and last fields
$ cut -d ':' -f 1,7 /etc/passwd
root:/bin/bash
daemon:/usr/sbin/nologin
...

# Process command output
$ ls -l | cut -c 1-10
# Extracts permission string
```

---

### paste - Merge Lines

**Basic Usage:**
```bash
# Create two files
$ cat file1.txt
A
B
C

$ cat file2.txt
1
2
3

# Paste side by side
$ paste file1.txt file2.txt
A    1
B    2
C    3

# Custom delimiter
$ paste -d ',' file1.txt file2.txt
A,1
B,2
C,3

# Paste multiple files
$ paste file1.txt file2.txt file3.txt
A    1    X
B    2    Y
C    3    Z

# Serial paste (all from one file, then next)
$ paste -s file1.txt file2.txt
A    B    C
1    2    3
```

---

### join - Join Files on Common Field

**Basic Usage:**
```bash
# Create files with common field
$ cat employees.txt
1 John Developer
2 Alice Manager
3 Bob Designer

$ cat salaries.txt
1 75000
2 90000
3 65000

# Join on first field
$ join employees.txt salaries.txt
1 John Developer 75000
2 Alice Manager 90000
3 Bob Designer 65000

# Join on different fields
$ join -1 2 -2 1 file1.txt file2.txt

# Use different delimiter
$ join -t ',' file1.csv file2.csv
```

---

## 5.4 Stream Editing with sed

### sed - Stream Editor

**Basic Substitution:**
```bash
# Replace first occurrence on each line
$ echo "hello world hello" | sed 's/hello/hi/'
hi world hello

# Replace all occurrences (global)
$ echo "hello world hello" | sed 's/hello/hi/g'
hi world hi

# Replace on specific line
$ sed '3s/old/new/' file.txt    # Only line 3

# Replace in range of lines
$ sed '1,5s/old/new/' file.txt  # Lines 1-5

# Case-insensitive replace
$ sed 's/hello/hi/i' file.txt

# Replace and save (in-place editing)
$ sed -i 's/old/new/g' file.txt

# Create backup before in-place edit
$ sed -i.bak 's/old/new/g' file.txt
```

**Delete Lines:**
```bash
# Delete line 3
$ sed '3d' file.txt

# Delete lines 2-5
$ sed '2,5d' file.txt

# Delete last line
$ sed '$d' file.txt

# Delete empty lines
$ sed '/^$/d' file.txt

# Delete lines matching pattern
$ sed '/pattern/d' file.txt

# Delete lines NOT matching pattern
$ sed '/pattern/!d' file.txt
```

**Insert and Append:**
```bash
# Insert before line 3
$ sed '3i\New line' file.txt

# Append after line 3
$ sed '3a\New line' file.txt

# Insert before pattern
$ sed '/pattern/i\New line' file.txt

# Append after pattern
$ sed '/pattern/a\New line' file.txt
```

**Print Specific Lines:**
```bash
# Print line 5
$ sed -n '5p' file.txt

# Print lines 10-20
$ sed -n '10,20p' file.txt

# Print lines matching pattern
$ sed -n '/pattern/p' file.txt

# Print lines NOT matching pattern
$ sed -n '/pattern/!p' file.txt
```

**Advanced Examples:**
```bash
# Replace multiple patterns
$ sed -e 's/cat/dog/g' -e 's/mouse/rat/g' file.txt

# Add line numbers
$ sed = file.txt | sed 'N;s/\n/\t/'

# Remove HTML tags
$ sed 's/<[^>]*>//g' file.html

# Comment out lines
$ sed 's/^/#/' file.txt

# Uncomment lines
$ sed 's/^#//' file.txt

# Double-space file
$ sed G file.txt

# Remove trailing whitespace
$ sed 's/[ \t]*$//' file.txt

# Add text to end of each line
$ sed 's/$/ - added/' file.txt
```

---

## 5.5 Pattern Scanning with awk

### awk - Pattern Scanning and Processing

**Basic Syntax:**
```bash
awk 'pattern { action }' file
```

**Print Columns:**
```bash
# Create space-separated file
$ cat data.txt
John 25 Developer
Alice 30 Manager
Bob 25 Designer

# Print first column
$ awk '{print $1}' data.txt
John
Alice
Bob

# Print multiple columns
$ awk '{print $1, $3}' data.txt
John Developer
Alice Manager
Bob Designer

# Print with custom separator
$ awk '{print $1 "," $3}' data.txt
John,Developer
Alice,Manager
Bob,Designer

# Print all columns
$ awk '{print $0}' data.txt

# Print last column
$ awk '{print $NF}' data.txt
Developer
Manager
Designer
```

**Field Separator:**
```bash
# CSV file
$ cat data.csv
John,25,Developer
Alice,30,Manager
Bob,25,Designer

# Use comma as separator
$ awk -F ',' '{print $1, $3}' data.csv
John Developer
Alice Manager
Bob Designer

# /etc/passwd (colon-separated)
$ awk -F ':' '{print $1, $7}' /etc/passwd
root /bin/bash
daemon /usr/sbin/nologin
...
```

**Pattern Matching:**
```bash
# Print lines containing "Developer"
$ awk '/Developer/ {print}' data.txt
John 25 Developer

# Print specific column if pattern matches
$ awk '/Developer/ {print $1}' data.txt
John

# Multiple patterns
$ awk '/Developer|Manager/ {print $1}' data.txt
John
Alice

# Pattern with column condition
$ awk '$2 > 25 {print $1, $2}' data.txt
Alice 30
```

**Conditions:**
```bash
# Print if age > 25
$ awk '$2 > 25 {print $1, $2}' data.txt
Alice 30

# Print if age == 25
$ awk '$2 == 25 {print $1}' data.txt
John
Bob

# Print if column matches pattern
$ awk '$3 ~ /Dev/ {print $1}' data.txt
John

# Print if column doesn't match
$ awk '$3 !~ /Dev/ {print $1}' data.txt
Alice
Bob
```

**Built-in Variables:**
```bash
NR    # Current record (line) number
NF    # Number of fields in current record
$0    # Entire current record
$1, $2, $3...  # Field values
FS    # Field separator
OFS   # Output field separator
RS    # Record separator
ORS   # Output record separator

# Print with line numbers
$ awk '{print NR, $0}' file.txt

# Print number of fields in each line
$ awk '{print NF}' file.txt

# Print last field of each line
$ awk '{print $NF}' file.txt

# Print lines with more than 3 fields
$ awk 'NF > 3 {print}' file.txt
```

**Calculations:**
```bash
# Sum of second column
$ awk '{sum += $2} END {print sum}' data.txt
80

# Average
$ awk '{sum += $2; count++} END {print sum/count}' data.txt
26.6667

# Find maximum
$ awk 'max < $2 {max = $2} END {print max}' data.txt
30

# Count lines
$ awk 'END {print NR}' file.txt
```

**BEGIN and END Blocks:**
```bash
# BEGIN: Execute before processing
# END: Execute after processing

$ awk 'BEGIN {print "Name Age Job"} {print $1, $2, $3} END {print "Total:", NR}' data.txt
Name Age Job
John 25 Developer
Alice 30 Manager
Bob 25 Designer
Total: 3

# Format output
$ awk 'BEGIN {print "Report"; print "------"} {print} END {print "------"; print "Done"}' data.txt
```

**Practical Examples:**
```bash
# Extract specific users from /etc/passwd
$ awk -F ':' '$3 >= 1000 {print $1, $3}' /etc/passwd

# Print users with bash shell
$ awk -F ':' '$7 == "/bin/bash" {print $1}' /etc/passwd

# Calculate disk usage
$ df -h | awk 'NR > 1 {print $1, $5}'

# Process log files
$ awk '/ERROR/ {print $1, $2, $NF}' /var/log/syslog

# Format CSV to table
$ awk -F ',' '{printf "%-10s %-5s %-15s\n", $1, $2, $3}' data.csv

# Count occurrences
$ awk '{count[$1]++} END {for (word in count) print word, count[word]}' file.txt
```

---

## 5.6 Text Editors

### nano - Beginner-Friendly Editor

**Basic Usage:**
```bash
# Open file
$ nano filename.txt

# Create new file
$ nano newfile.txt
```

**Common Shortcuts:**
```
^G    # Help (^ means Ctrl)
^O    # Save (Write Out)
^X    # Exit
^K    # Cut line
^U    # Paste
^W    # Search
^\    # Search and replace
^C    # Show cursor position
^_    # Go to line number
```

**Usage Example:**
```bash
$ nano config.txt
# Edit the file
# Press Ctrl+O to save
# Press Enter to confirm filename
# Press Ctrl+X to exit
```

---

### vim - Powerful Text Editor

**Modes:**
1. **Normal Mode** - Navigate and execute commands (default)
2. **Insert Mode** - Type text (press `i`)
3. **Visual Mode** - Select text (press `v`)
4. **Command Mode** - Execute commands (press `:`)

**Opening Files:**
```bash
# Open file
$ vim filename.txt

# Open at specific line
$ vim +10 filename.txt

# Open multiple files
$ vim file1.txt file2.txt

# Open in read-only mode
$ vim -R filename.txt
```

**Basic Navigation (Normal Mode):**
```
h    # Left
j    # Down
k    # Up
l    # Right

w    # Next word
b    # Previous word
0    # Beginning of line
$    # End of line
gg   # First line of file
G    # Last line of file
:10  # Go to line 10
```

**Entering Insert Mode:**
```
i    # Insert before cursor
a    # Insert after cursor
I    # Insert at beginning of line
A    # Insert at end of line
o    # Open new line below
O    # Open new line above
```

**Editing (Normal Mode):**
```
x    # Delete character
dd   # Delete line
dw   # Delete word
D    # Delete to end of line
yy   # Copy (yank) line
yw   # Copy word
p    # Paste after cursor
P    # Paste before cursor
u    # Undo
Ctrl+r    # Redo
.    # Repeat last command
```

**Search and Replace:**
```
/pattern       # Search forward
?pattern       # Search backward
n             # Next match
N             # Previous match

:s/old/new/           # Replace first on line
:s/old/new/g          # Replace all on line
:%s/old/new/g         # Replace all in file
:%s/old/new/gc        # Replace with confirmation
```

**Save and Exit (Command Mode):**
```
:w         # Save
:w file    # Save as file
:q         # Quit
:q!        # Quit without saving
:wq        # Save and quit
:x         # Save and quit (only if changes)
ZZ         # Save and quit (Normal mode)
```

**Visual Mode:**
```
v          # Character-wise visual mode
V          # Line-wise visual mode
Ctrl+v     # Block-wise visual mode

# In visual mode:
d          # Delete selection
y          # Copy selection
>          # Indent
<          # Unindent
```

**Practical vim Usage:**
```bash
# Quick edit and save
$ vim file.txt
# Press i
# Type your text
# Press Esc
# Type :wq
# Press Enter

# Find and replace all
$ vim file.txt
# Type :%s/old/new/g
# Press Enter
# Type :wq

# Delete lines 10-20
$ vim file.txt
# Type :10,20d
# Type :wq
```

**vim Configuration (.vimrc):**
```bash
# Create ~/.vimrc
$ cat > ~/.vimrc << 'EOF'
set number          " Show line numbers
set tabstop=4       " Tab width
set expandtab       " Use spaces instead of tabs
set autoindent      " Auto indent
syntax on           " Syntax highlighting
set hlsearch        " Highlight search results
EOF
```

---

## 💡 Key Concepts Summary

1. **Basic tools**: echo, cat, sort, uniq for simple text operations
2. **grep**: Powerful text searching with regular expressions
3. **tr**: Character translation and deletion
4. **cut, paste, join**: Column extraction and merging
5. **sed**: Stream editing for find-and-replace
6. **awk**: Pattern scanning and text processing
7. **nano**: Beginner-friendly editor
8. **vim**: Powerful modal editor

---

## 🔥 Real-World Use Cases

### Use Case 1: Log File Analysis

```bash
# Find all errors in log
$ grep -i "error" /var/log/syslog

# Count errors by type
$ grep -i "error" /var/log/syslog | cut -d ':' -f 4 | sort | uniq -c

# Find top 10 most common errors
$ grep -i "error" /var/log/syslog | awk '{print $5}' | sort | uniq -c | sort -rn | head -10

# Extract IP addresses from access log
$ grep -E -o "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" access.log | sort | uniq -c | sort -rn
```

### Use Case 2: Data Processing

```bash
# CSV to formatted table
$ cat data.csv
Name,Age,Department
John,25,Engineering
Alice,30,Marketing

$ column -t -s ',' data.csv
Name   Age  Department
John   25   Engineering
Alice  30   Marketing

# Extract specific columns
$ cut -d ',' -f 1,3 data.csv | sed '1d'  # Skip header
John,Engineering
Alice,Marketing

# Calculate average
$ cut -d ',' -f 2 data.csv | sed '1d' | awk '{sum+=$1} END {print sum/NR}'
27.5
```

### Use Case 3: Configuration File Management

```bash
# Remove comments and empty lines
$ grep -v "^#" config.conf | grep -v "^$"

# Find all IP addresses in config
$ grep -E -o "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" config.conf

# Replace server address in all configs
$ sed -i 's/old-server.com/new-server.com/g' *.conf

# Add prefix to each line
$ sed 's/^/    /' file.txt  # Add 4 spaces
```

### Use Case 4: User Management

```bash
# List all human users (UID >= 1000)
$ awk -F ':' '$3 >= 1000 {print $1}' /etc/passwd

# Find users with bash shell
$ grep "/bin/bash$" /etc/passwd | cut -d ':' -f 1

# Count users per shell
$ cut -d ':' -f 7 /etc/passwd | sort | uniq -c
```

### Use Case 5: Text File Batch Processing

```bash
# Convert all .txt files to uppercase
$ for file in *.txt; do
    tr '[:lower:]' '[:upper:]' < "$file" > "${file%.txt}_upper.txt"
done

# Replace string in all files
$ find . -name "*.conf" -exec sed -i 's/old/new/g' {} \;

# Extract emails from all files
$ grep -rhE "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" . | sort -u
```

---

## 🧪 Practice Problems

### Beginner Level

**Problem 1**: Create a file with 5 lines of text, then use cat to display it with line numbers.

**Problem 2**: Use echo to create a file containing "Hello World", then append "Goodbye World" to it.

**Problem 3**: Create a file with unsorted names, then sort them alphabetically.

**Problem 4**: Search for the word "root" in /etc/passwd using grep.

**Problem 5**: Use tr to convert a sentence to all uppercase.

### Intermediate Level

**Problem 6**: Find all lines in /etc/passwd that don't contain "/nologin", and display just the usernames.

**Problem 7**: Create a CSV file with Name,Age,City. Extract only names and cities (columns 1 and 3).

**Problem 8**: Count how many times each word appears in a text file.

**Problem 9**: Replace all occurrences of "color" with "colour" in a file using sed.

**Problem 10**: Use awk to print lines from /etc/passwd where the UID (field 3) is greater than 1000.

### Advanced Level

**Problem 11**: Write a pipeline to find the top 10 most common words in a text file.

**Problem 12**: Extract all email addresses from a file and count unique domains.

**Problem 13**: Use sed to add line numbers to a file, but only for non-empty lines.

**Problem 14**: Create an awk script that processes a log file and prints: timestamp, log level, and message (assuming they're in fields 1, 2, and 4+).

**Problem 15**: Use vim to open a file, find all occurrences of "TODO", and replace them with "DONE", then save and quit. Write the command sequence.

---

## 📝 Solutions

### Beginner Solutions

**Solution 1**:
```bash
$ cat > myfile.txt
Line 1
Line 2
Line 3
Line 4
Line 5
^D

$ cat -n myfile.txt
     1  Line 1
     2  Line 2
     3  Line 3
     4  Line 4
     5  Line 5
```

**Solution 2**:
```bash
$ echo "Hello World" > greeting.txt
$ cat greeting.txt
Hello World

$ echo "Goodbye World" >> greeting.txt
$ cat greeting.txt
Hello World
Goodbye World
```

**Solution 3**:
```bash
$ cat > names.txt
Dave
Charlie
Alice
Bob
^D

$ sort names.txt
Alice
Bob
Charlie
Dave

# Or save to new file
$ sort names.txt > sorted_names.txt
```

**Solution 4**:
```bash
$ grep "root" /etc/passwd
root:x:0:0:root:/root:/bin/bash

# More specific - lines starting with root
$ grep "^root" /etc/passwd
root:x:0:0:root:/root:/bin/bash
```

**Solution 5**:
```bash
$ echo "hello world" | tr 'a-z' 'A-Z'
HELLO WORLD

# Or from file
$ cat file.txt | tr 'a-z' 'A-Z'
```

### Intermediate Solutions

**Solution 6**:
```bash
$ grep -v "/nologin" /etc/passwd | cut -d ':' -f 1
root
john
alice
bob
...

# Alternative with awk
$ awk -F ':' '$7 !~ /nologin/ {print $1}' /etc/passwd
```

**Solution 7**:
```bash
$ cat > data.csv
Name,Age,City
John,25,NYC
Alice,30,LA
Bob,28,Chicago
^D

$ cut -d ',' -f 1,3 data.csv
Name,City
John,NYC
Alice,LA
Bob,Chicago
```

**Solution 8**:
```bash
# Create test file
$ cat > words.txt
hello world
hello there
world of programming
^D

# Count word occurrences
$ cat words.txt | tr -s ' ' '\n' | sort | uniq -c | sort -rn
      2 hello
      2 world
      1 there
      1 programming
      1 of
```

**Solution 9**:
```bash
$ cat > british.txt
This color is my favorite color.
^D

$ sed 's/color/colour/g' british.txt
This colour is my favorite colour.

# In-place edit
$ sed -i 's/color/colour/g' british.txt
$ cat british.txt
This colour is my favourite colour.
```

**Solution 10**:
```bash
$ awk -F ':' '$3 > 1000 {print}' /etc/passwd
# or just print username and UID
$ awk -F ':' '$3 > 1000 {print $1, $3}' /etc/passwd
john 1000
alice 1001
bob 1002
```

### Advanced Solutions

**Solution 11**:
```bash
# Top 10 most common words
$ cat file.txt | \
  tr -cs 'A-Za-z' '\n' | \
  tr 'A-Z' 'a-z' | \
  sort | \
  uniq -c | \
  sort -rn | \
  head -10

# Explanation:
# tr -cs 'A-Za-z' '\n'  - Keep only letters, replace others with newline
# tr 'A-Z' 'a-z'        - Convert to lowercase
# sort                  - Sort words
# uniq -c               - Count unique occurrences
# sort -rn              - Sort by count (reverse numeric)
# head -10              - Top 10
```

**Solution 12**:
```bash
# Extract emails and count domains
$ grep -E -o "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt | \
  cut -d '@' -f 2 | \
  sort | \
  uniq -c | \
  sort -rn

# Example output:
#      5 gmail.com
#      3 company.com
#      2 outlook.com
```

**Solution 13**:
```bash
# Add line numbers to non-empty lines only
$ sed '/^$/!=' file.txt | sed 'N;s/\n/\t/'

# Alternative with awk
$ awk 'NF {printf "%d\t%s\n", ++count, $0} !NF {print}' file.txt

# Explanation:
# NF - number of fields (0 for empty lines)
# If line has fields, print line number and line
# If empty, just print empty line
```

**Solution 14**:
```bash
# Assuming log format: timestamp level user message
$ awk '{
    timestamp = $1
    level = $2
    message = ""
    for (i = 4; i <= NF; i++) {
        message = message $i " "
    }
    print timestamp, level, message
}' logfile.txt

# Or more concisely:
$ awk '{print $1, $2, substr($0, index($0,$4))}' logfile.txt
```

**Solution 15**:
```bash
# Vim command sequence:
vim file.txt
:%s/TODO/DONE/g
:wq

# Or as one-liner from command line:
$ vim -c '%s/TODO/DONE/g' -c 'wq' file.txt

# Or with sed (non-interactive):
$ sed -i 's/TODO/DONE/g' file.txt
```

---

## 🎓 Learning Tips

### grep Patterns Mnemonic

- **^** = "Caret points at start"
- **$** = "Dollar at end (payday at end of month)"
- **.** = "Dot matches any dot"
- **\*** = "Star means zero or more"
- **[]** = "Brackets hold choices"

### sed Quick Reference

```bash
s/old/new/        # Substitute
s/old/new/g       # Global substitute
s/old/new/i       # Case-insensitive
/pattern/d        # Delete matching lines
/pattern/p        # Print matching lines
5,10d             # Delete lines 5-10
```

### awk Field Reference

```bash
$0  = entire line
$1  = first field
$2  = second field
$NF = last field
NR  = line number
NF  = number of fields
```

### vim Survival Commands

```
i   - Enter insert mode
Esc - Return to normal mode
:wq - Save and quit
:q! - Quit without saving
dd  - Delete line
u   - Undo
```

### Best Practices

1. **Test on sample data first**
   ```bash
   # Test sed command before -i
   $ sed 's/old/new/g' file.txt | head
   # If good, then:
   $ sed -i 's/old/new/g' file.txt
   ```

2. **Use pipelines for complex tasks**
   ```bash
   # Break into steps
   $ cat file.txt | grep pattern | sort | uniq -c
   ```

3. **Quote patterns to avoid shell expansion**
   ```bash
   $ grep "pattern with spaces" file.txt
   $ sed 's/old/new/g' file.txt
   ```

4. **Backup before in-place editing**
   ```bash
   $ sed -i.bak 's/old/new/g' file.txt
   ```

5. **Learn one tool well before moving to next**
   - Master grep before sed
   - Master sed before awk
   - Master awk before perl/python

### Common Mistakes to Avoid

1. **Forgetting grep is case-sensitive**
   ```bash
   # WRONG - might miss "Error", "ERROR"
   $ grep "error" file.txt
   
   # RIGHT
   $ grep -i "error" file.txt
   ```

2. **Not sorting before uniq**
   ```bash
   # WRONG - uniq only removes consecutive duplicates
   $ uniq file.txt
   
   # RIGHT
   $ sort file.txt | uniq
   ```

3. **Using sed without test on important files**
   ```bash
   # DANGEROUS
   $ sed -i 's/old/new/g' important.txt
   
   # SAFER
   $ sed -i.bak 's/old/new/g' important.txt
   ```

4. **Not escaping special characters in regex**
   ```bash
   # WRONG - . matches any character
   $ grep "192.168.1.1" file.txt
   
   # RIGHT
   $ grep "192\.168\.1\.1" file.txt
   ```

---

## 🔗 Connections to Future Chapters

- **Chapter 6**: Text processing with pipes and redirection
- **Chapter 11**: Using these tools in shell scripts
- **Chapter 13**: Log file analysis for system monitoring
- **Chapter 17**: Advanced regular expressions
- **Chapter 21**: Troubleshooting with grep and log analysis

---

## 📚 Additional Resources

### Man Pages
```bash
man grep
man sed
man awk
man vim
man tr
man cut
```

### Online Resources
- Regex101.com - Test regular expressions
- Vim Adventures - Learn vim through a game
- Sed Tutorial: https://www.grymoire.com/Unix/Sed.html
- Awk Tutorial: https://www.grymoire.com/Unix/Awk.html

### Practice Sites
- RegexOne - Interactive regex tutorial
- Vim Tutor - Type `vimtutor` in terminal
- HackerRank - Shell challenges

---

## ✅ Self-Assessment Checklist

Before moving to Chapter 6, ensure you can:
- [ ] Use grep to search for patterns in files
- [ ] Understand and use basic regular expressions
- [ ] Sort and remove duplicates from text
- [ ] Extract columns with cut
- [ ] Use sed for find-and-replace operations
- [ ] Process text with awk (print fields, filter lines)
- [ ] Navigate and edit files in vim
- [ ] Build text-processing pipelines
- [ ] Transform text with tr
- [ ] Count words, lines, and occurrences
- [ ] Apply appropriate tool for each task

---

## 🚀 Next Steps

Congratulations! You've mastered text processing - one of Linux's most powerful capabilities.

In **Chapter 6**, you'll learn:
- Input/Output redirection (>, >>, <, 2>)
- Pipes and filters
- Building complex command pipelines
- Using tee and xargs
- Advanced data flow control

**Challenge**: Process a log file to find the top 10 most active IP addresses, showing their request count. Use a pipeline of grep, cut/awk, sort, and uniq.

---

*"Unix is user-friendly. It's just very selective about who its friends are. Text processing is the secret handshake."*