# Chapter 3: Branching and Merging

## Introduction

Branching is one of Git's most powerful features. It allows you to diverge from the main line of development and work on different features or experiments in complete isolation, then bring those changes back together. This chapter will teach you how to create, switch between, and merge branches effectively.

## What are Branches?

A branch in Git is simply a lightweight movable pointer to a commit. Think of it as a parallel universe where you can make changes without affecting the main project.

### Why Use Branches?

- **Isolate features**: Work on new features without breaking working code
- **Experiment safely**: Try ideas without committing to them
- **Collaborate effectively**: Multiple people can work on different features simultaneously
- **Organize work**: Keep bug fixes, features, and experiments separate
- **Enable workflows**: Support processes like code review and continuous integration

### The Default Branch

When you initialize a repository, Git creates a default branch (usually `main` or `master`). This is typically where your stable, production-ready code lives.

## Branch Basics

### Viewing Branches

```bash
# List local branches
git branch

# List all branches (including remote)
git branch -a

# List remote branches only
git branch -r

# List with last commit info
git branch -v

# List merged branches
git branch --merged

# List unmerged branches
git branch --no-merged
```

The current branch is marked with an asterisk (*).

### Creating Branches

```bash
# Create a new branch (but stay on current branch)
git branch feature-login

# Create and switch to new branch
git checkout -b feature-login
# or (newer syntax)
git switch -c feature-login

# Create branch from specific commit
git branch feature-login abc123

# Create branch from another branch
git branch feature-login develop
```

### Switching Branches

```bash
# Switch to existing branch
git checkout feature-login
# or (newer, clearer syntax)
git switch feature-login

# Switch to previous branch
git checkout -
# or
git switch -

# Create and switch in one command
git checkout -b new-feature
# or
git switch -c new-feature
```

**Important**: Before switching branches, commit or stash your changes!

### Deleting Branches

```bash
# Delete a merged branch
git branch -d feature-login

# Force delete an unmerged branch (be careful!)
git branch -D feature-login

# Delete remote branch
git push origin --delete feature-login
# or
git push origin :feature-login
```

## Branch Workflows

### Feature Branch Workflow

The most common pattern:

```bash
# Start from main
git checkout main
git pull  # Get latest changes

# Create feature branch
git checkout -b feature-user-profile

# Make your changes
# ... edit files ...

# Commit your work
git add .
git commit -m "Add user profile page"

# More commits as needed
git commit -am "Add profile edit functionality"
git commit -am "Add profile image upload"

# Merge back to main
git checkout main
git merge feature-user-profile

# Delete feature branch
git branch -d feature-user-profile
```

### Topic Branch Best Practices

- **One branch per feature/fix**: Keep work isolated
- **Short-lived branches**: Merge quickly to avoid divergence
- **Descriptive names**: `feature-login`, `bugfix-404-error`, `refactor-database`
- **Keep branches updated**: Regularly merge from main to avoid conflicts

## Merging Branches

Merging combines changes from different branches. Git handles most merges automatically.

### Fast-Forward Merge

When the target branch hasn't changed, Git simply moves the pointer forward:

```bash
git checkout main
git merge feature-login  # Fast-forward merge
```

**Diagram:**
```
Before:           After:
main: A---B       main: A---B---C---D
               \                    
feature:         C---D        feature: (deleted)
```

### Three-Way Merge

When both branches have diverged, Git creates a new merge commit:

```bash
git checkout main
git merge feature-login  # Creates merge commit
```

**Diagram:**
```
Before:           After:
main: A---B---C   main: A---B---C---M
               \                    /
feature:         D---E---F         /
                                  /
                        feature: (can be deleted)
```

### Merge Command Options

```bash
# Standard merge
git merge feature-branch

# Merge without fast-forward (always create merge commit)
git merge --no-ff feature-branch

# Merge with custom message
git merge -m "Merge feature-login into main" feature-branch

# See what would be merged (don't actually merge)
git merge --no-commit --no-ff feature-branch
git diff --cached  # Review changes
git merge --abort  # Cancel the merge

# Squash commits into one
git merge --squash feature-branch
git commit -m "Add login feature"
```

### Aborting a Merge

If things go wrong during a merge:

```bash
# Cancel the merge
git merge --abort

# Return to state before merge
git reset --hard HEAD
```

## Merge Conflicts

Conflicts occur when Git can't automatically merge changes because the same lines were modified in both branches.

### Recognizing a Conflict

```bash
$ git merge feature-branch
Auto-merging file.txt
CONFLICT (content): Merge conflict in file.txt
Automatic merge failed; fix conflicts and then commit the result.
```

### Conflict Markers

Git marks conflicts in your files:

```
<<<<<<< HEAD (Current branch)
This is content from the main branch
=======
This is content from the feature branch
>>>>>>> feature-branch
```

### Resolving Conflicts - Step by Step

1. **Identify conflicted files**:
```bash
git status  # Lists files with conflicts
```

2. **Open each file and look for conflict markers** (`<<<<<<<`, `=======`, `>>>>>>>`)

3. **Edit the file to resolve the conflict**:
   - Choose one version
   - Combine both versions
   - Write something completely new
   - Remove the conflict markers

4. **Mark as resolved**:
```bash
git add resolved-file.txt
```

5. **Complete the merge**:
```bash
git commit  # Opens editor with default merge message
# or
git commit -m "Merge feature-branch, resolve conflicts in file.txt"
```

### Conflict Resolution Example

**Before (in file.txt):**
```
<<<<<<< HEAD
print("Hello from main!")
=======
print("Hello from feature!")
>>>>>>> feature-login
```

**After resolving (choose to keep both):**
```
print("Hello from main!")
print("Hello from feature!")
```

Then:
```bash
git add file.txt
git commit -m "Merge feature-login, combine greeting messages"
```

### Conflict Resolution Tools

```bash
# Use visual merge tool
git mergetool

# Configure your preferred tool
git config --global merge.tool vimdiff
# or meld, kdiff3, p4merge, etc.

# View conflicts with
git diff  # Shows conflicts
git diff --ours  # Our changes
git diff --theirs  # Their changes

# Choose one side entirely
git checkout --ours file.txt    # Keep our version
git checkout --theirs file.txt  # Keep their version
git add file.txt
git commit
```

### Preventing Conflicts

1. **Pull often**: Stay up-to-date with the main branch
2. **Small, focused commits**: Easier to merge
3. **Communicate**: Know what others are working on
4. **Merge main into your branch regularly**:
```bash
git checkout feature-branch
git merge main  # Bring main's changes into your branch
```

## Rebasing

Rebasing is an alternative to merging that creates a linear history.

### What is Rebasing?

Rebasing moves or combines commits to a new base commit. It rewrites history to make it cleaner.

```bash
# Rebase current branch onto main
git checkout feature-branch
git rebase main
```

**Diagram:**
```
Before:           After:
main: A---B---C   main: A---B---C
               \                  \
feature:         D---E             D'---E'
```

Notice commits D and E are "replayed" on top of C.

### When to Rebase

**Use rebase when:**
- You want a clean, linear history
- Working on a local branch that hasn't been shared
- Updating your feature branch with main's latest changes

**Don't rebase when:**
- The branch has been pushed and others are working on it
- You're unsure - merging is safer!

**Golden Rule**: Never rebase commits that have been pushed to a shared repository!

### Rebase vs Merge

**Merge:**
- Preserves complete history
- Shows when branches diverged and merged
- Safer - doesn't rewrite history
- Can create messy history with many merge commits

**Rebase:**
- Creates clean, linear history
- Easier to understand at a glance
- Rewrites history (can be dangerous)
- Loses information about when branches diverged

### Interactive Rebase

Powerful tool for cleaning up commits:

```bash
# Rebase last 3 commits
git rebase -i HEAD~3

# Rebase from specific commit
git rebase -i abc123
```

This opens an editor with your commits:

```
pick abc123 Add login form
pick def456 Fix typo
pick ghi789 Add validation

# Commands:
# p, pick = use commit
# r, reword = use commit, but edit message
# e, edit = use commit, but stop for amending
# s, squash = meld into previous commit
# f, fixup = like squash, but discard message
# d, drop = remove commit
```

**Example - Squash commits:**
```
pick abc123 Add login form
squash def456 Fix typo
squash ghi789 Add validation
```

This combines all three into one commit.

### Handling Rebase Conflicts

```bash
# If conflict occurs during rebase
# 1. Resolve conflicts in files
# 2. Stage resolved files
git add resolved-file.txt

# 3. Continue rebase
git rebase --continue

# Or skip this commit
git rebase --skip

# Or abort entirely
git rebase --abort
```

## Advanced Branching

### Checking Out Remote Branches

```bash
# List remote branches
git branch -r

# Create local branch tracking remote
git checkout -b feature-login origin/feature-login
# or (simpler)
git checkout feature-login  # Git auto-creates tracking branch

# or (newest syntax)
git switch feature-login
```

### Renaming Branches

```bash
# Rename current branch
git branch -m new-name

# Rename specific branch
git branch -m old-name new-name

# Update remote (if already pushed)
git push origin :old-name new-name
git push origin -u new-name
```

### Comparing Branches

```bash
# See commits in feature-branch not in main
git log main..feature-branch

# See commits in main not in feature-branch
git log feature-branch..main

# See all differences
git diff main...feature-branch

# See files changed between branches
git diff --name-status main..feature-branch
```

### Cherry-Picking

Apply specific commits from one branch to another:

```bash
# Apply commit abc123 to current branch
git cherry-pick abc123

# Cherry-pick multiple commits
git cherry-pick abc123 def456

# Cherry-pick without committing (stage only)
git cherry-pick -n abc123
```

## Practical Exercises

### Exercise 1: Basic Branching

```bash
# Initialize a repository
git init my-app
cd my-app

# Create initial commit
echo "# My App" > README.md
git add README.md
git commit -m "Initial commit"

# Create and switch to feature branch
git checkout -b feature-header

# Make changes
echo "## Header Component" >> README.md
git commit -am "Add header section"

# Switch back to main
git checkout main

# Merge feature
git merge feature-header

# View history
git log --oneline --graph
```

### Exercise 2: Handling Conflicts

```bash
# On main branch
echo "Main content" > file.txt
git add file.txt
git commit -m "Add content on main"

# Create feature branch from one commit back
git checkout -b feature HEAD~1
echo "Feature content" > file.txt
git add file.txt
git commit -m "Add content on feature"

# Try to merge
git checkout main
git merge feature  # Conflict!

# Resolve conflict
# Edit file.txt to resolve
git add file.txt
git commit -m "Merge feature, resolve conflict"
```

### Exercise 3: Feature Branch Workflow

```bash
# Start fresh
git checkout main

# Create feature branch
git checkout -b feature-login

# Make multiple commits
echo "Login page" > login.html
git add login.html
git commit -m "Add login page"

echo "function login() {}" > login.js
git add login.js
git commit -m "Add login logic"

# Switch to main and make change
git checkout main
echo "Updated readme" >> README.md
git commit -am "Update README"

# Merge feature back
git merge feature-login

# View result
git log --oneline --graph --all
```

### Exercise 4: Interactive Rebase

```bash
# Create messy commits
git checkout -b cleanup
echo "Change 1" >> file.txt
git commit -am "WIP change 1"

echo "Change 2" >> file.txt
git commit -am "Fix typo"

echo "Change 3" >> file.txt
git commit -am "Add real content"

# Clean up with interactive rebase
git rebase -i HEAD~3
# Squash the commits together

# View clean history
git log --oneline
```

## Common Mistakes & Solutions

### Mistake: Switched Branches with Uncommitted Changes

**Problem**: Changes moved to new branch

**Solution**: Switch back and commit or stash
```bash
git checkout original-branch
git stash  # or commit
git checkout new-branch
```

### Mistake: Merged Wrong Branch

**Solution**: Undo the merge
```bash
git reset --hard HEAD~1  # If not pushed
# or
git revert -m 1 HEAD  # If already pushed
```

### Mistake: Deleted Branch Too Soon

**Solution**: Recover it
```bash
# Find the commit
git reflog

# Recreate branch
git branch feature-login abc123
```

### Mistake: Committed to Wrong Branch

**Solution**: Move commit to correct branch
```bash
# Note the commit hash
git log -1

# Switch to correct branch
git checkout correct-branch

# Cherry-pick the commit
git cherry-pick abc123

# Switch back and remove commit
git checkout wrong-branch
git reset --hard HEAD~1
```

## Best Practices

1. **Keep main stable**: Only merge tested, working code
2. **One feature per branch**: Makes merging and reviewing easier
3. **Delete merged branches**: Keeps repository clean
4. **Pull before you push**: Avoid unnecessary merges
5. **Use descriptive branch names**: `feature/user-auth`, not `fix1`
6. **Merge or rebase frequently**: Don't let branches diverge too much
7. **Never rebase shared branches**: You'll cause problems for others
8. **Review before merging**: Check what you're bringing in

## Branch Naming Conventions

```bash
# Features
feature/user-authentication
feature/shopping-cart
feat/payment-integration

# Bug fixes
bugfix/login-redirect
fix/memory-leak
hotfix/security-patch

# Experiments
experiment/new-algorithm
spike/performance-test

# Releases
release/v1.2.0
release/2024-01-30
```

## Next Steps

You now understand how to work with branches, handle merges, and resolve conflicts. This opens up powerful collaboration possibilities!

**Continue to**: [Chapter 4: Working with Remote Repositories](../04-remote-repositories/README.md)

## Additional Resources

- [Git Branching - Basic Branching and Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
- [Git Branching - Rebasing](https://git-scm.com/book/en/v2/Git-Branching-Rebasing)
- [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)
- [Learn Git Branching (Interactive)](https://learngitbranching.js.org/)

---

**Quick Reference Commands**

```bash
# Viewing branches
git branch                  # List local branches
git branch -a               # List all branches
git branch -v               # Show last commit

# Creating/switching branches
git branch feature          # Create branch
git checkout feature        # Switch to branch
git checkout -b feature     # Create and switch
git switch feature          # Switch (newer)
git switch -c feature       # Create and switch (newer)

# Deleting branches
git branch -d feature       # Delete merged branch
git branch -D feature       # Force delete

# Merging
git merge feature           # Merge feature into current
git merge --no-ff feature   # No fast-forward
git merge --abort           # Cancel merge

# Rebasing
git rebase main             # Rebase onto main
git rebase -i HEAD~3        # Interactive rebase
git rebase --abort          # Cancel rebase

# Comparing
git diff main..feature      # Changes in feature
git log main..feature       # Commits in feature

# Cherry-picking
git cherry-pick abc123      # Apply specific commit
```