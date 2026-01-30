# Chapter 5: Viewing History

## Introduction

One of Git's most powerful features is its ability to maintain a complete history of your project. This chapter teaches you how to explore that history, understand how your project evolved, find specific changes, and navigate through past commits.

## Why View History?

- **Understand evolution**: See how features were built over time
- **Debug issues**: Find when bugs were introduced
- **Learn from others**: Understand team members' approaches
- **Find changes**: Locate specific modifications
- **Code archaeology**: Discover why decisions were made
- **Generate reports**: Track progress and contributions

## Basic Log Commands

### Simple Log

```bash
# View commit history
git log

# Each commit shows:
# - Commit hash (SHA-1)
# - Author
# - Date
# - Commit message
```

### Compact Log

```bash
# One line per commit
git log --oneline

# Shows abbreviated hash and message
# Example output:
# a1b2c3d Add login feature
# e4f5g6h Fix navigation bug
# i7j8k9l Update README
```

### Limit Number of Commits

```bash
# Show last 5 commits
git log -5

# Show last 10 commits
git log -n 10

# One-line format, last 3 commits
git log --oneline -3
```

## Formatting Log Output

### Custom Formats

```bash
# Show with stats
git log --stat

# Show with patch (full diff)
git log -p
# or
git log --patch

# Show specific number with patch
git log -p -2

# Abbreviated stats
git log --shortstat

# Show commit graph
git log --graph

# Show all branches
git log --all

# Combine options
git log --oneline --graph --all
```

### Pretty Formats

```bash
# Predefined formats
git log --pretty=oneline
git log --pretty=short
git log --pretty=full
git log --pretty=fuller

# Custom format
git log --pretty=format:"%h - %an, %ar : %s"
# %h = abbreviated hash
# %an = author name
# %ar = author date, relative
# %s = subject/message

# Useful custom format
git log --pretty=format:"%C(yellow)%h%C(reset) - %C(green)%an%C(reset), %ar : %s"

# With graph
git log --graph --pretty=format:"%C(yellow)%h%C(reset) %C(blue)%an%C(reset) - %s %C(green)(%cr)%C(reset)"
```

### Format Placeholders

Common placeholders for `--pretty=format`:

- `%H` - Commit hash (full)
- `%h` - Commit hash (abbreviated)
- `%T` - Tree hash
- `%t` - Tree hash (abbreviated)
- `%P` - Parent hashes
- `%p` - Parent hashes (abbreviated)
- `%an` - Author name
- `%ae` - Author email
- `%ad` - Author date
- `%ar` - Author date, relative (e.g., "2 weeks ago")
- `%cn` - Committer name
- `%ce` - Committer email
- `%cd` - Committer date
- `%cr` - Committer date, relative
- `%s` - Subject (commit message)
- `%b` - Body (commit message)

### Colors

```bash
# Color codes in format string
# %C(color) = start color
# %C(reset) = reset to default

# Available colors:
# red, green, blue, yellow, magenta, cyan, white
# black, normal

# Styles:
# bold, dim, ul (underline), blink, reverse
```

## Filtering Commits

### By Author

```bash
# Commits by specific author
git log --author="John Doe"

# Multiple authors (regex)
git log --author="John\|Jane"

# Case insensitive
git log --author="john" --regexp-ignore-case
```

### By Date

```bash
# Since specific date
git log --since="2024-01-01"
git log --after="2024-01-01"

# Before specific date
git log --until="2024-12-31"
git log --before="2024-12-31"

# Date range
git log --since="2 weeks ago" --until="yesterday"

# Relative dates
git log --since="1 month ago"
git log --since="2 weeks ago"
git log --since="yesterday"

# Specific time
git log --since="2024-01-15 14:30:00"
```

### By Commit Message

```bash
# Search commit messages
git log --grep="bug fix"

# Case insensitive
git log --grep="feature" -i

# Multiple patterns (OR)
git log --grep="bug" --grep="fix"

# Multiple patterns (AND)
git log --grep="bug" --grep="fix" --all-match

# Invert match
git log --grep="WIP" --invert-grep
```

### By File or Directory

```bash
# Commits affecting specific file
git log -- filename.txt

# Commits affecting files in directory
git log -- src/components/

# Multiple files
git log -- file1.txt file2.txt

# Follow file through renames
git log --follow -- filename.txt

# Show patches for specific file
git log -p -- filename.txt
```

### By Content Changes

```bash
# Find commits that added or removed a string
git log -S "function login()"

# Same, but with regex
git log -G "function.*login"

# Show the actual changes
git log -S "function login()" -p
```

### Combining Filters

```bash
# Commits by author in date range
git log --author="John" --since="1 month ago"

# Commits with message containing "fix" affecting specific file
git log --grep="fix" -- src/app.js

# Complex query
git log --author="John" --since="2024-01-01" --until="2024-01-31" --grep="feature" -- src/
```

## Viewing Specific Commits

### Show Commit Details

```bash
# Show specific commit
git show abc123

# Show commit with stats
git show --stat abc123

# Show only message
git show --no-patch abc123
# or
git show -s abc123

# Show specific file from commit
git show abc123:path/to/file.txt

# Show commit's parent
git show abc123^
git show abc123~1
```

### Commit Ranges

```bash
# Commits between two points
git log abc123..def456

# Commits on branch-A not on branch-B
git log branch-B..branch-A

# Commits on either branch but not both
git log branch-A...branch-B

# Visualize the difference
git log --left-right --oneline branch-A...branch-B
```

## Viewing File History

### Log for Specific File

```bash
# History of a file
git log -- filename.txt

# With changes
git log -p -- filename.txt

# Following renames
git log --follow -- filename.txt

# Compact view
git log --oneline --follow -- filename.txt

# Who changed what
git log --pretty=format:"%h %an %s" -- filename.txt
```

### File at Specific Point

```bash
# View file as it was in commit
git show abc123:path/to/file.txt

# View file as it was in branch
git show feature-branch:src/app.js

# View file from 2 commits ago
git show HEAD~2:README.md
```

### Blame - Line by Line History

```bash
# See who last modified each line
git blame filename.txt

# With line numbers
git blame -L 10,20 filename.txt

# Show author email
git blame -e filename.txt

# Show commit date
git blame -t filename.txt

# Ignore whitespace changes
git blame -w filename.txt

# More compact output
git blame -s filename.txt
```

## Comparing Commits

### Diff Between Commits

```bash
# Compare two commits
git diff abc123 def456

# Compare with HEAD
git diff abc123

# Compare specific file
git diff abc123 def456 -- file.txt

# Summary only
git diff --stat abc123 def456

# Names only
git diff --name-only abc123 def456

# Names and status (Modified, Added, Deleted)
git diff --name-status abc123 def456
```

### Diff Between Branches

```bash
# Changes in branch-B not in branch-A
git diff branch-A..branch-B

# Same as above
git diff branch-A branch-B

# Three-dot diff (since divergence point)
git diff branch-A...branch-B

# Summary
git diff --stat main..feature-branch
```

## Graphical History

### Built-in Graphical View

```bash
# Text-based graph
git log --graph --oneline --all

# More detailed graph
git log --graph --all --decorate

# Beautiful colored graph
git log --graph --all --pretty=format:'%C(yellow)%h%Creset -%C(red)%d%Creset %s %C(green)(%cr) %C(blue)<%an>%Creset' --abbrev-commit --date=relative
```

### Gitk - GUI Tool

```bash
# Open gitk (comes with Git)
gitk

# View all branches
gitk --all

# View specific file history
gitk filename.txt

# View date range
gitk --since="1 month ago"
```

### Other GUI Tools

- **GitKraken**: Cross-platform, beautiful interface
- **SourceTree**: Free, by Atlassian
- **GitHub Desktop**: Simple, GitHub integration
- **Git GUI**: Comes with Git
- **Sublime Merge**: By Sublime Text creators

## Finding Commits

### Find When Bug Was Introduced

Git bisect helps find the commit that introduced a bug.

```bash
# Start bisect
git bisect start

# Mark current commit as bad
git bisect bad

# Mark a known good commit
git bisect good abc123

# Git checks out middle commit
# Test it and mark:
git bisect good  # if working
# or
git bisect bad   # if broken

# Repeat until Git finds the first bad commit

# End bisect
git bisect reset
```

**Automated bisect:**
```bash
# With a test script
git bisect start HEAD abc123
git bisect run ./test-script.sh
# Script should exit 0 for good, 1 for bad
```

### Find Commit by Message

```bash
# Search messages
git log --grep="search term"

# Show matches with context
git log --grep="bug" --oneline

# Case insensitive
git log --grep="feature" -i
```

### Find Commit that Changed Content

```bash
# Find when "old_function" was removed
git log -S "old_function" --oneline

# Find with regex
git log -G "function.*login" --oneline

# Show the changes
git log -S "old_function" -p
```

## Reference Log (reflog)

The reflog records when the tips of branches are updated.

### View Reflog

```bash
# Show reflog
git reflog

# Show reflog for specific branch
git reflog show branch-name

# Show with dates
git reflog --date=relative

# Last 10 entries
git reflog -10
```

### Using Reflog

```bash
# Reference specific entry
git show HEAD@{2}

# See where HEAD was 1 hour ago
git show HEAD@{1.hour.ago}

# See where main was yesterday
git show main@{yesterday}

# Recover lost commit
git reflog
# Find the commit hash
git checkout -b recovery abc123
```

### Reflog Expiration

```bash
# Reflog entries expire after 90 days by default

# View unreachable commits before they expire
git fsck --lost-found

# Expire reflog now
git reflog expire --expire=now --all

# Keep reflog forever for specific branch
git config gc.main.reflogExpire never
```

## Advanced History Queries

### Merge Commits

```bash
# Show only merge commits
git log --merges

# Exclude merge commits
git log --no-merges

# Show first-parent only (main line)
git log --first-parent
```

### Authors and Contributors

```bash
# List all contributors
git shortlog -sn

# With email
git shortlog -sne

# All commits by contributor
git shortlog

# Since specific date
git shortlog --since="6 months ago" -sn
```

### Statistics

```bash
# Total commits
git rev-list --count HEAD

# Commits per author
git shortlog -sn --all

# Files changed in commit
git show --stat abc123

# Insertion/deletion stats
git log --stat

# Number of commits per day
git log --format="%ad" --date=short | sort | uniq -c
```

## Practical Exercises

### Exercise 1: Exploring History

```bash
# Initialize a repository with history
git init history-practice
cd history-practice

# Create some commits
echo "Version 1" > file.txt
git add file.txt
git commit -m "Initial commit"

echo "Version 2" >> file.txt
git commit -am "Add version 2"

echo "Version 3" >> file.txt
git commit -am "Add version 3"

# View different log formats
git log
git log --oneline
git log --graph --oneline
git log --stat
git log -p

# View specific commit
git show HEAD~1

# Compare commits
git diff HEAD~2 HEAD
```

### Exercise 2: Finding Changes

```bash
# Search commit messages
git log --grep="version"

# Find when a line was added
git log -S "Version 2" --oneline

# See who modified lines
git blame file.txt

# View file history
git log --follow -- file.txt
```

### Exercise 3: Using Bisect

```bash
# Create scenario with a bug
git checkout -b bisect-demo
for i in {1..10}; do
  echo "Commit $i" >> test.txt
  git add test.txt
  git commit -m "Commit $i"
done

# "Bug" introduced in commit 5
# Add marker
git checkout HEAD~5
echo "BUG" >> test.txt
git add test.txt
git commit --amend --no-edit

# Use bisect to find it
git checkout bisect-demo
git bisect start
git bisect bad
git bisect good HEAD~9

# Test each:
grep -q "BUG" test.txt && git bisect bad || git bisect good

git bisect reset
```

### Exercise 4: Using Reflog

```bash
# Make some changes
echo "Change 1" > temp.txt
git add temp.txt
git commit -m "Temp change"

# Reset
git reset --hard HEAD~1

# Recover using reflog
git reflog
# Find the commit hash
git checkout -b recovered <hash>
```

## Useful Aliases

Add these to your `.gitconfig`:

```bash
# Create aliases
git config --global alias.lg "log --graph --oneline --all"
git config --global alias.last "log -1 HEAD"
git config --global alias.unstage "reset HEAD --"
git config --global alias.who "shortlog -sn --"

# Beautiful log
git config --global alias.tree "log --graph --pretty=format:'%C(yellow)%h%C(reset) -%C(red)%d%C(reset) %s %C(green)(%cr) %C(blue)<%an>%C(reset)' --abbrev-commit --date=relative"

# Then use with:
git lg
git last
git who
git tree
```

## Best Practices

1. **Use --oneline for quick scans**: Get overview before deep diving
2. **Combine filters**: Narrow down to relevant commits
3. **Follow file renames**: Use `--follow` for moved files
4. **Use blame wisely**: Find context, not blame people
5. **Learn bisect**: Invaluable for finding bug origins
6. **Check reflog when lost**: Can recover almost anything
7. **Use GUI tools**: Sometimes visual is clearer
8. **Create aliases**: Save time on common commands

## Common Patterns

### Recent Work

```bash
# What did I do today?
git log --since="midnight" --author="Your Name"

# What did I do this week?
git log --since="1 week ago" --author="Your Name" --oneline
```

### Team Activity

```bash
# What happened recently?
git log --since="yesterday" --oneline

# Who's been active?
git shortlog --since="1 week ago" -sn

# What changed in main?
git log origin/main..main
```

### File Investigation

```bash
# When did this file last change?
git log -1 --format="%ar" -- filename.txt

# Who worked on this file most?
git log --format="%an" -- filename.txt | sort | uniq -c | sort -nr

# How has this file evolved?
git log -p --follow -- filename.txt
```

## Next Steps

You now know how to explore your project's history comprehensively! You can find any commit, track down bugs, and understand how your project evolved.

**Continue to**: [Chapter 6: Undoing Changes](../06-undoing-changes/README.md)

## Additional Resources

- [Git Log Documentation](https://git-scm.com/docs/git-log)
- [Git Show Documentation](https://git-scm.com/docs/git-show)
- [Git Blame Documentation](https://git-scm.com/docs/git-blame)
- [Git Bisect Tutorial](https://git-scm.com/docs/git-bisect)

---

**Quick Reference Commands**

```bash
# Basic log
git log                          # Full log
git log --oneline                # Compact
git log -5                       # Last 5 commits
git log --graph --all            # Visual graph

# Filtering
git log --author="Name"          # By author
git log --since="1 week ago"     # By date
git log --grep="term"            # By message
git log -- file.txt              # By file
git log -S "content"             # By content change

# Viewing commits
git show abc123                  # Show commit
git diff abc123..def456          # Compare commits
git blame file.txt               # Line history

# Finding
git bisect start                 # Find bad commit
git reflog                       # Reference log

# Statistics
git shortlog -sn                 # Contributors
git log --stat                   # With stats
```