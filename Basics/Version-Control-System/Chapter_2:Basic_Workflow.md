# Chapter 2: Basic Workflow

## Introduction

This chapter covers the fundamental Git commands you'll use daily. These are the core operations that make up 90% of typical Git usage. By mastering this chapter, you'll be able to track changes, create commits, and maintain a clean project history.

## The Basic Git Workflow

The typical workflow follows this pattern:

1. **Modify** files in your working directory
2. **Stage** changes you want to include in your next commit
3. **Commit** staged changes to your repository
4. **Repeat**

```
Edit Files → Stage Changes → Commit → Repeat
   ↓            (git add)    (git commit)
Working Dir → Staging Area → Repository
```

## Creating a Repository

### Initialize a New Repository

Create a Git repository in a new or existing project:

```bash
# Navigate to your project directory
cd /path/to/your/project

# Initialize Git
git init

# Verify it worked
git status
```

This creates a hidden `.git` folder containing all version control information.

### Clone an Existing Repository

Copy a repository from another location:

```bash
# Clone from a URL
git clone https://github.com/username/repository.git

# Clone into a specific folder
git clone https://github.com/username/repository.git my-folder-name

# Clone a specific branch
git clone -b branch-name https://github.com/username/repository.git
```

## Checking Repository Status

The `git status` command is your best friend. Use it constantly.

```bash
git status
```

This shows:
- Which branch you're on
- Which files are modified
- Which files are staged for commit
- Untracked files
- Your status relative to the remote repository

**Pro tip**: Run `git status` before and after every other Git command while learning.

## The Staging Area (Index)

Git's staging area is unique and powerful. It lets you prepare commits with surgical precision.

### Why a Staging Area?

- **Selective commits**: Commit only some of your changes
- **Review before committing**: Double-check what you're about to commit
- **Logical grouping**: Group related changes into focused commits

### Adding Files to Staging

```bash
# Stage a specific file
git add filename.txt

# Stage multiple files
git add file1.txt file2.txt file3.txt

# Stage all files in current directory
git add .

# Stage all files in the repository
git add -A
# or
git add --all

# Stage all modified and deleted files (but not new files)
git add -u
# or
git add --update

# Interactively choose what to stage
git add -i

# Stage parts of files (hunks)
git add -p filename.txt
# or
git add --patch filename.txt
```

### Viewing Staged Changes

```bash
# See what's staged for commit
git diff --staged
# or
git diff --cached
```

### Unstaging Files

Made a mistake? Remove files from staging:

```bash
# Unstage a specific file
git restore --staged filename.txt

# Unstage all files
git restore --staged .

# Old method (still works)
git reset HEAD filename.txt
```

## Making Commits

A commit is a snapshot of your project at a specific point in time. Each commit should represent a logical, complete change.

### Basic Commit

```bash
# Commit staged changes with message
git commit -m "Add user authentication feature"

# Commit with multi-line message
git commit -m "Add user authentication" -m "- Implemented login form
- Added password validation
- Created session management"
```

### Opening an Editor for Commit Message

```bash
# Opens your configured editor
git commit
```

Your editor will open with a template. Write your message, save, and close.

### Commit Shortcuts

```bash
# Stage all modified/deleted files AND commit
# (doesn't include new files)
git commit -a -m "Update documentation"
# or
git commit -am "Update documentation"

# Amend the previous commit
# (useful for fixing typos or adding forgotten files)
git add forgotten-file.txt
git commit --amend

# Amend without changing the message
git commit --amend --no-edit
```

**Warning**: Only amend commits that haven't been pushed to a shared repository!

## Writing Good Commit Messages

Good commit messages are crucial for project history and collaboration.

### The Seven Rules

1. **Separate subject from body with a blank line**
2. **Limit the subject line to 50 characters**
3. **Capitalize the subject line**
4. **Do not end the subject line with a period**
5. **Use the imperative mood** ("Add feature" not "Added feature")
6. **Wrap the body at 72 characters**
7. **Use the body to explain what and why, not how**

### Examples

**Good:**
```
Add user authentication system

Implemented JWT-based authentication to secure API endpoints.
Users can now register, login, and maintain sessions.
This addresses issue #42.
```

**Bad:**
```
fixed stuff
```

**Good short commit:**
```
Fix typo in README
```

**Good multi-line commit:**
```
Refactor database connection logic

- Extract connection pooling into separate module
- Add connection retry logic with exponential backoff
- Improve error handling and logging

This improves reliability and makes the code more maintainable.
Closes #123
```

### Conventional Commits (Optional Standard)

Many teams use a structured format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

Example:
```
feat(auth): add password reset functionality

Users can now request a password reset link via email.
The link expires after 1 hour for security.

Closes #234
```

## Viewing Your Changes

### See Unstaged Changes

```bash
# View changes in working directory
git diff

# View changes for a specific file
git diff filename.txt

# View changes with word-level diff
git diff --word-diff
```

### See Staged Changes

```bash
# View changes staged for commit
git diff --staged
# or
git diff --cached
```

### See Both Staged and Unstaged

```bash
git diff HEAD
```

## Ignoring Files

Some files shouldn't be tracked (build artifacts, dependencies, secrets, etc.)

### Create .gitignore

Create a `.gitignore` file in your repository root:

```bash
# Create .gitignore
touch .gitignore
```

### .gitignore Syntax

```gitignore
# Ignore specific file
secrets.txt

# Ignore all files with extension
*.log
*.tmp

# Ignore directory
node_modules/
build/
dist/

# Ignore files in any directory
**/*.pyc

# Negative pattern (don't ignore)
!important.log

# Ignore in root only
/TODO.txt

# Comments start with #
# This is a comment
```

### Common .gitignore Examples

**Node.js project:**
```gitignore
node_modules/
npm-debug.log
.env
dist/
```

**Python project:**
```gitignore
__pycache__/
*.pyc
*.pyo
venv/
.env
*.egg-info/
```

**General:**
```gitignore
# OS generated files
.DS_Store
Thumbs.db

# IDE files
.vscode/
.idea/
*.swp
*.swo

# Environment variables
.env
.env.local
```

### Useful .gitignore Commands

```bash
# Check why a file is ignored
git check-ignore -v filename.txt

# Force add an ignored file (not recommended)
git add -f ignored-file.txt

# Remove file from tracking but keep in working directory
git rm --cached filename.txt
```

### gitignore.io

Generate .gitignore files for your stack: [gitignore.io](https://gitignore.io)

## Removing Files

### Remove File from Git and Filesystem

```bash
git rm filename.txt
git commit -m "Remove obsolete file"
```

### Remove File from Git Only (Keep Locally)

```bash
git rm --cached filename.txt
git commit -m "Stop tracking config file"
```

### Remove Directory

```bash
git rm -r directory-name/
git commit -m "Remove old directory"
```

## Moving/Renaming Files

### Rename or Move Files

```bash
# Git will track the rename
git mv old-name.txt new-name.txt
git commit -m "Rename file for clarity"

# Move to different directory
git mv file.txt subdirectory/file.txt
git commit -m "Reorganize project structure"
```

### Alternative Method

Git is smart enough to detect renames:

```bash
# Using regular shell commands
mv old-name.txt new-name.txt
git add -A
git status  # Git shows "renamed: old-name.txt -> new-name.txt"
git commit -m "Rename file"
```

## Viewing Commit History

```bash
# View commit history
git log

# Compact one-line format
git log --oneline

# View last N commits
git log -5

# View with file changes
git log --stat

# View with actual changes
git log -p

# View graphical representation
git log --graph --oneline --all

# View commits by specific author
git log --author="John Doe"

# View commits in date range
git log --since="2 weeks ago"
git log --after="2024-01-01" --before="2024-01-31"

# View commits affecting specific file
git log filename.txt

# Search commits by message
git log --grep="bug fix"
```

## Practical Exercises

### Exercise 1: First Commit

```bash
# Create a new project
mkdir my-project
cd my-project
git init

# Create a file
echo "# My Project" > README.md

# Check status
git status

# Stage the file
git add README.md

# Check status again
git status

# Commit
git commit -m "Initial commit: Add README"

# View history
git log
```

### Exercise 2: Making Changes

```bash
# Edit the README
echo "This is my first Git project" >> README.md

# View changes
git diff

# Stage and commit
git add README.md
git commit -m "Add project description"

# View history
git log --oneline
```

### Exercise 3: Staging Selectively

```bash
# Create multiple files
echo "print('Hello')" > script.py
echo "# TODO" > TODO.txt

# View status
git status

# Stage only one file
git add script.py

# Check what's staged
git status

# Commit
git commit -m "Add Python script"

# Stage and commit the other file
git add TODO.txt
git commit -m "Add TODO list"
```

### Exercise 4: Working with .gitignore

```bash
# Create files that shouldn't be tracked
echo "secret" > .env
echo "temp data" > temp.log

# Create .gitignore
echo ".env" > .gitignore
echo "*.log" >> .gitignore

# Check status
git status  # Only .gitignore should show

# Commit .gitignore
git add .gitignore
git commit -m "Add .gitignore"
```

## Common Mistakes & Solutions

### Mistake: Committed to Wrong Files

**Solution**: Amend the commit
```bash
git add forgotten-file.txt
git commit --amend --no-edit
```

### Mistake: Wrong Commit Message

**Solution**: Amend with new message
```bash
git commit --amend -m "Correct message"
```

### Mistake: Accidentally Staged Wrong File

**Solution**: Unstage it
```bash
git restore --staged wrong-file.txt
```

### Mistake: Want to Undo Changes in Working Directory

**Solution**: Restore from last commit
```bash
git restore filename.txt  # Discard changes

# Old method (still works)
git checkout -- filename.txt
```

**Warning**: This permanently discards your changes!

### Mistake: Committed Sensitive Information

**Solution**: Remove from history (advanced - be careful!)
```bash
# Remove file from tracking
git rm --cached secrets.txt

# Add to .gitignore
echo "secrets.txt" >> .gitignore

# Commit
git add .gitignore
git commit -m "Remove secrets from tracking"

# For committed sensitive data, you may need:
# - git filter-branch (complex)
# - BFG Repo-Cleaner (easier tool)
# - Recreate repository (simplest for small projects)
```

## Best Practices

1. **Commit often**: Small, frequent commits are better than large, infrequent ones
2. **Make atomic commits**: Each commit should represent one logical change
3. **Write meaningful messages**: Future you will thank present you
4. **Review before committing**: Use `git status` and `git diff --staged`
5. **Don't commit generated files**: Use .gitignore for build artifacts
6. **Keep commits focused**: One commit = one purpose
7. **Test before committing**: Ensure code works before committing

## Daily Workflow Checklist

```bash
# 1. Check status
git status

# 2. Review your changes
git diff

# 3. Stage your changes
git add <files>

# 4. Review staged changes
git diff --staged

# 5. Commit with meaningful message
git commit -m "Your message"

# 6. View your commit
git log -1
```

## Next Steps

You now understand the basic Git workflow! You can track changes, create commits, and maintain project history. 

**Continue to**: [Chapter 3: Branching and Merging](../03-branching-and-merging/README.md)

## Additional Resources

- [Git Basics - Recording Changes](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [gitignore.io](https://gitignore.io)
- [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)

---

**Quick Reference Commands**

```bash
# Repository setup
git init                    # Initialize repository
git clone <url>             # Clone repository

# Status and diffs
git status                  # Check status
git diff                    # Unstaged changes
git diff --staged           # Staged changes

# Staging
git add <file>              # Stage file
git add .                   # Stage all in current dir
git add -A                  # Stage all changes
git restore --staged <file> # Unstage file

# Committing
git commit -m "message"     # Commit with message
git commit -am "message"    # Stage & commit modified
git commit --amend          # Modify last commit

# History
git log                     # View history
git log --oneline           # Compact history
git log --graph --all       # Graphical history

# File operations
git rm <file>               # Remove file
git mv <old> <new>          # Rename/move file
git restore <file>          # Discard changes
```