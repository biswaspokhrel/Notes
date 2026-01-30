# Chapter 1: Getting Started

## Introduction

Welcome to your Git journey! This chapter covers the foundational concepts of version control and gets you set up with Git on your system. By the end of this chapter, you'll understand why version control matters and have Git configured and ready to use.

## What is Version Control?

Version control is a system that records changes to files over time so you can recall specific versions later. Think of it as an unlimited "undo" button for your entire project, plus the ability to collaborate with others without stepping on each other's toes.

### Why Use Version Control?

- **Track History**: See every change ever made, who made it, and why
- **Collaborate Safely**: Multiple people can work on the same project simultaneously
- **Experiment Freely**: Try new ideas in isolation without breaking working code
- **Backup & Recovery**: Never lose work, even if you delete files accidentally
- **Professional Standard**: Git is used by virtually every software team worldwide

### What Makes Git Special?

Git is a **distributed** version control system, meaning:
- Every developer has a complete copy of the project history
- You can work offline and sync later
- It's incredibly fast for most operations
- Branching and merging are lightweight and powerful

## Installing Git

### Windows

**Option 1: Git for Windows**
1. Download from [git-scm.com](https://git-scm.com/download/win)
2. Run the installer
3. Accept most defaults, but consider these settings:
   - Default editor: Choose Visual Studio Code or your preferred editor
   - Line ending conversions: "Checkout Windows-style, commit Unix-style"
4. Verify installation: Open Command Prompt and type `git --version`

**Option 2: GitHub Desktop**
- Includes Git and a graphical interface
- Download from [desktop.github.com](https://desktop.github.com)

### macOS

**Option 1: Homebrew (Recommended)**
```bash
brew install git
```

**Option 2: Xcode Command Line Tools**
```bash
xcode-select --install
```

**Option 3: Official Installer**
- Download from [git-scm.com](https://git-scm.com/download/mac)

Verify installation:
```bash
git --version
```

### Linux

**Debian/Ubuntu:**
```bash
sudo apt-get update
sudo apt-get install git
```

**Fedora:**
```bash
sudo dnf install git
```

**Arch:**
```bash
sudo pacman -S git
```

Verify installation:
```bash
git --version
```

## Initial Configuration

Before using Git, you need to set up your identity. This information gets embedded in every commit you make.

### Required: Set Your Name and Email

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

**Important**: Use the email associated with your GitHub/GitLab account if you plan to use those services.

### Recommended: Set Your Default Editor

```bash
# Visual Studio Code
git config --global core.editor "code --wait"

# Vim
git config --global core.editor "vim"

# Nano
git config --global core.editor "nano"

# Notepad++ (Windows)
git config --global core.editor "'C:/Program Files/Notepad++/notepad++.exe' -multiInst -notabbar -nosession -noPlugin"
```

### Recommended: Set Default Branch Name

Modern convention uses "main" instead of "master":

```bash
git config --global init.defaultBranch main
```

### Optional: Enable Color Output

```bash
git config --global color.ui auto
```

### View Your Configuration

```bash
# View all settings
git config --list

# View a specific setting
git config user.name
```

### Configuration Levels

Git has three configuration levels:

1. **System**: Applies to all users (`--system`)
2. **Global**: Applies to your user account (`--global`) - most common
3. **Local**: Applies to a specific repository (`--local` or no flag)

Local settings override global, and global overrides system.

## Understanding Git's Architecture

### The Three States

Files in Git can exist in three states:

1. **Modified**: You've changed the file but haven't committed it
2. **Staged**: You've marked a modified file to go into your next commit
3. **Committed**: The data is safely stored in your local database

### The Three Sections

This leads to three main sections of a Git project:

1. **Working Directory**: Where you modify files
2. **Staging Area (Index)**: Where you prepare changes for commit
3. **Git Directory (Repository)**: Where Git stores metadata and object database

### Basic Git Workflow

```
Working Directory → Staging Area → Git Repository
     (edit)       (git add)      (git commit)
```

## Getting Help

Git has extensive built-in documentation:

```bash
# Three equivalent ways to get help
git help <command>
git <command> --help
man git-<command>

# Quick reference
git <command> -h

# Examples:
git help commit
git commit --help
git commit -h
```

## Key Terms Glossary

- **Repository (Repo)**: A directory containing your project and its complete history
- **Commit**: A snapshot of your project at a specific point in time
- **Branch**: A parallel version of your repository
- **Clone**: A copy of a repository
- **Remote**: A version of your project hosted on the internet or network
- **Push**: Send your commits to a remote repository
- **Pull**: Fetch and merge changes from a remote repository
- **Merge**: Combine changes from different branches
- **Staging Area**: A holding area for changes before committing

## Practical Exercise

Let's verify your Git installation and configuration:

### Exercise 1: Check Your Setup

```bash
# Verify Git is installed
git --version

# Check your configuration
git config --list

# Verify your name and email are set
git config user.name
git config user.email
```

### Exercise 2: Explore Git Help

```bash
# Get help on the config command
git help config

# Get a quick reference for commit
git commit -h
```

### Exercise 3: Create Your First Repository

```bash
# Create a new directory
mkdir my-first-repo
cd my-first-repo

# Initialize a Git repository
git init

# Check the status
git status
```

You should see a message indicating you're on the main (or master) branch with no commits yet.

## Common Issues & Solutions

### Issue: Git command not found

**Solution**: Git isn't installed or isn't in your PATH
- Reinstall Git
- On Windows, ensure "Use Git from the Windows Command Prompt" was selected during installation
- Restart your terminal/command prompt

### Issue: "Please tell me who you are" error

**Solution**: You haven't configured your name and email
```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### Issue: Line ending warnings on Windows

**Solution**: Configure Git to handle line endings:
```bash
git config --global core.autocrlf true
```

## Best Practices

1. **Use meaningful configuration**: Your commits will be tied to your identity forever
2. **Choose an editor you're comfortable with**: You'll be writing commit messages frequently
3. **Keep your email consistent**: Especially important for GitHub contributions
4. **Use SSH keys**: More secure and convenient than passwords (we'll cover this in Chapter 4)

## Next Steps

Now that you have Git installed and configured, you're ready to learn the basic workflow! 

**Continue to**: [Chapter 2: Basic Workflow](../02-basic-workflow/README.md)

## Additional Resources

- [Official Git Documentation](https://git-scm.com/doc)
- [Pro Git Book](https://git-scm.com/book/en/v2) - Free and comprehensive
- [Git Reference Manual](https://git-scm.com/docs)
- [First-Time Git Setup Guide](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)

---

**Quick Reference Commands**

```bash
# Installation verification
git --version

# Configuration
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"

# View configuration
git config --list
git config user.name

# Get help
git help <command>
git <command> -h
```