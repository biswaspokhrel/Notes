# Chapter 6: Undoing Changes

## Introduction

Everyone makes mistakes. Git provides powerful tools to undo changes, recover lost work, and fix errors. This chapter teaches you how to safely undo changes at different stages of your workflow, from modifying uncommitted files to rewriting published history.

## Understanding Levels of Undoing

Git allows you to undo changes at different levels:

1. **Working Directory**: Unstaged changes in files
2. **Staging Area**: Changes staged for commit
3. **Local Repository**: Committed changes
4. **Remote Repository**: Published changes

The deeper the level, the more careful you need to be.

## Undoing Working Directory Changes

### Discard Changes in Files

```bash
# Discard changes in specific file
git restore file.txt

# Discard changes in all files
git restore .

# Discard changes in multiple files
git restore file1.txt file2.txt

# Old syntax (still works)
git checkout -- file.txt
```

**Warning**: This permanently deletes your uncommitted changes!

### Restore File from Specific Commit

```bash
# Restore file from specific commit
git restore --source=abc123 file.txt

# Restore file from HEAD (last commit)
git restore --source=HEAD file.txt

# Restore from 2 commits ago
git restore --source=HEAD~2 file.txt

# Old syntax
git checkout abc123 -- file.txt
```

### Discard All Local Changes

```bash
# Discard all changes and untracked files
git reset --hard HEAD
git clean -fd

# Just discard changes (keep untracked files)
git reset --hard HEAD

# Just remove untracked files
git clean -fd
```

**Warning**: This is destructive and permanent!

### Preview Clean Operation

```bash
# See what would be deleted
git clean -n

# Delete untracked files
git clean -f

# Delete untracked files and directories
git clean -fd

# Include ignored files
git clean -fdx
```

## Undoing Staged Changes

### Unstage Files

```bash
# Unstage specific file
git restore --staged file.txt

# Unstage all files
git restore --staged .

# Old syntax (still works)
git reset HEAD file.txt
git reset HEAD  # Unstage all
```

This keeps your changes in the working directory.

### Unstage and Discard

```bash
# Unstage and discard changes
git restore --staged file.txt
git restore file.txt

# Or in one command (old syntax)
git checkout HEAD -- file.txt
```

## Undoing Commits

### Amend Last Commit

Change the most recent commit (message or content):

```bash
# Change commit message
git commit --amend -m "New message"

# Add forgotten files to last commit
git add forgotten-file.txt
git commit --amend --no-edit

# Change commit and message
git add file.txt
git commit --amend -m "Updated commit"
```

**Warning**: Only amend commits that haven't been pushed!

### Revert a Commit

Create a new commit that undoes a previous commit:

```bash
# Revert specific commit (creates new commit)
git revert abc123

# Revert without committing (stage changes)
git revert -n abc123
git revert --no-commit abc123

# Revert multiple commits
git revert abc123 def456

# Revert a range
git revert HEAD~3..HEAD

# Revert merge commit
git revert -m 1 abc123
```

**Revert vs Reset**: Revert is safe for published commits because it creates a new commit instead of rewriting history.

### Reset to Previous Commit

Move the branch pointer backward:

```bash
# Three types of reset:

# 1. Soft reset - keep changes staged
git reset --soft HEAD~1

# 2. Mixed reset (default) - keep changes unstaged
git reset HEAD~1
git reset --mixed HEAD~1

# 3. Hard reset - discard all changes
git reset --hard HEAD~1
```

**Reset modes:**

| Mode | Moves HEAD | Changes staging | Changes working dir |
|------|------------|-----------------|---------------------|
| --soft | Yes | No | No |
| --mixed | Yes | Yes | No |
| --hard | Yes | Yes | Yes |

### Reset Use Cases

**Undo last commit, keep changes:**
```bash
git reset HEAD~1
```

**Completely remove last commit:**
```bash
git reset --hard HEAD~1
```

**Undo commits but keep work staged:**
```bash
git reset --soft HEAD~3
```

**Reset to specific commit:**
```bash
git reset --hard abc123
```

### Reset vs Revert Comparison

```bash
# REVERT - Safe for published history
git revert abc123
# Creates: abc123 → xyz789 (new commit undoing abc123)
# History: A → B → C → C'

# RESET - Rewrites history (dangerous if pushed)
git reset --hard abc123
# Moves: HEAD → abc123
# History: A → B (C is gone)
```

## Recovering Lost Commits

### Using Reflog

The reflog tracks where HEAD has been:

```bash
# View reflog
git reflog

# Shows entries like:
# abc123 HEAD@{0}: commit: Latest commit
# def456 HEAD@{1}: commit: Previous commit
# ghi789 HEAD@{2}: reset: moving to HEAD~1
```

### Recover from Reset

```bash
# You did a hard reset and want it back
git reset --hard HEAD~3

# Check reflog
git reflog

# Find the commit you want
# Recover it
git reset --hard abc123

# Or create a new branch at that commit
git branch recovered abc123
```

### Recover Deleted Branch

```bash
# You deleted a branch
git branch -D feature-branch

# Find it in reflog
git reflog

# Recreate it
git branch feature-branch abc123
```

### Find Orphaned Commits

```bash
# Find commits not reachable from any branch
git fsck --lost-found

# This creates files in .git/lost-found/
# You can view them with:
git show <hash>
```

## Undoing Multiple Commits

### Interactive Rebase

Rewrite commit history interactively:

```bash
# Rebase last 5 commits
git rebase -i HEAD~5

# Rebase from specific commit
git rebase -i abc123
```

Editor opens with:
```
pick abc123 Commit message 1
pick def456 Commit message 2
pick ghi789 Commit message 3

# Commands:
# p, pick = use commit
# r, reword = use commit, but edit message
# e, edit = use commit, but stop for amending
# s, squash = meld into previous commit
# f, fixup = like squash, but discard message
# d, drop = remove commit
```

### Common Rebase Operations

**Squash commits:**
```
pick abc123 Add feature
squash def456 Fix typo
squash ghi789 Add tests
```

**Reorder commits:**
```
pick ghi789 Add tests
pick abc123 Add feature
pick def456 Fix typo
```

**Remove commit:**
```
pick abc123 Add feature
drop def456 Bad commit
pick ghi789 Add tests
```

**Edit commit:**
```
pick abc123 Add feature
edit def456 Need to split this
pick ghi789 Add tests
```

When marked for edit:
```bash
# Git stops at the commit
# Make your changes
git add file.txt
git commit --amend

# Continue
git rebase --continue
```

### Abort Rebase

```bash
# If rebase goes wrong
git rebase --abort

# Returns to state before rebase started
```

## Undoing Published Changes

### Revert Public Commits

For commits that have been pushed:

```bash
# Safe - creates new commit
git revert abc123
git push origin main

# Don't use reset on published commits!
# This would cause problems for others:
git reset --hard abc123  # ❌ DON'T DO THIS
git push --force         # ❌ DANGEROUS
```

### Force Push (Use Carefully!)

Sometimes you need to rewrite published history:

```bash
# Standard force push (dangerous!)
git push --force origin main

# Safer force push (checks remote hasn't changed)
git push --force-with-lease origin main
```

**When force push is acceptable:**
- Your own feature branch
- Fixing a critical mistake quickly
- Everyone on team is aware and can re-clone

**When force push is NOT acceptable:**
- Shared branches (main, develop)
- Others are working on the branch
- Without team coordination

### Fix Force Push Problems

If someone force pushed and you need to update:

```bash
# Your push is rejected because of force push

# Option 1: Fetch and reset (lose your changes)
git fetch origin
git reset --hard origin/main

# Option 2: Fetch and rebase (keep your changes)
git fetch origin
git rebase origin/main

# Option 3: Fetch and create backup branch
git branch backup-branch
git fetch origin
git reset --hard origin/main
```

## Stashing Changes

Save changes temporarily without committing:

### Basic Stash

```bash
# Stash current changes
git stash

# Or with a message
git stash save "Work in progress on feature"

# Stash including untracked files
git stash -u
git stash --include-untracked

# Stash everything including ignored files
git stash -a
git stash --all
```

### Applying Stashes

```bash
# List stashes
git stash list
# Output:
# stash@{0}: WIP on main: abc123 Latest commit
# stash@{1}: On feature: def456 Feature work

# Apply most recent stash
git stash apply

# Apply specific stash
git stash apply stash@{1}

# Apply and remove from stash list
git stash pop

# Apply specific and remove
git stash pop stash@{0}
```

### Managing Stashes

```bash
# View stash contents
git stash show
git stash show -p  # With full diff

# View specific stash
git stash show stash@{1}

# Create branch from stash
git stash branch feature-name stash@{0}

# Delete specific stash
git stash drop stash@{1}

# Delete all stashes
git stash clear
```

### Partial Stashing

```bash
# Interactively choose what to stash
git stash -p
git stash --patch

# Stash only staged changes
git stash --staged
```

## Fixing Common Mistakes

### Committed to Wrong Branch

```bash
# You're on main but should be on feature

# Option 1: Move commits to new branch
git branch feature-branch  # Create branch at current commit
git reset --hard HEAD~3    # Remove commits from main
git checkout feature-branch

# Option 2: Move commits to existing branch
git checkout feature-branch
git merge main
git checkout main
git reset --hard HEAD~3
```

### Committed with Wrong Email/Name

```bash
# Fix last commit
git commit --amend --author="Correct Name <correct@email.com>"

# Fix multiple commits (interactive rebase)
git rebase -i HEAD~5
# Mark commits with 'edit'
# For each:
git commit --amend --author="Correct Name <correct@email.com>" --no-edit
git rebase --continue
```

### Merged Wrong Branch

```bash
# Just merged wrong branch
git reset --hard HEAD~1  # If not pushed

# If already pushed
git revert -m 1 HEAD  # Revert merge commit
```

### Committed Sensitive Data

```bash
# Just committed (not pushed)
git reset --hard HEAD~1

# Already pushed - nuclear option
# Use BFG Repo-Cleaner or git filter-branch
# Then force push (everyone must re-clone!)

# Better: Invalidate the sensitive data
# (Revoke API keys, change passwords, etc.)
```

### Pushed to Wrong Remote/Branch

```bash
# Pushed to wrong branch on same remote
# Delete the wrong branch
git push origin --delete wrong-branch

# Pushed to wrong remote
# Delete from wrong remote
git push wrong-remote --delete branch-name
```

## Practical Exercises

### Exercise 1: Unstaging and Discarding

```bash
# Make changes
echo "Change 1" > file1.txt
echo "Change 2" > file2.txt

# Stage
git add .

# Unstage one file
git restore --staged file1.txt

# Discard unstaged changes
git restore file1.txt

# Discard staged changes
git restore --staged file2.txt
git restore file2.txt
```

### Exercise 2: Amending Commits

```bash
# Create commit
echo "Content" > file.txt
git add file.txt
git commit -m "Add flie"  # Typo!

# Fix message
git commit --amend -m "Add file"

# Add forgotten file
echo "More" > file2.txt
git add file2.txt
git commit --amend --no-edit
```

### Exercise 3: Using Reset

```bash
# Create commits
echo "V1" > file.txt
git add file.txt
git commit -m "Version 1"

echo "V2" >> file.txt
git commit -am "Version 2"

echo "V3" >> file.txt
git commit -am "Version 3"

# Soft reset - keep changes staged
git reset --soft HEAD~2
git status  # Changes are staged

# Mixed reset - keep changes unstaged
git reset HEAD~1
git status  # Changes are unstaged

# Hard reset - discard changes
git reset --hard HEAD~1
git status  # Clean working directory
```

### Exercise 4: Recovering Commits

```bash
# Make commits
echo "Important" > important.txt
git add important.txt
git commit -m "Important work"

# Accidentally reset
git reset --hard HEAD~1

# Recover
git reflog
# Find the commit
git reset --hard abc123
# Or create recovery branch
git branch recovered abc123
```

### Exercise 5: Stashing

```bash
# Start working
echo "WIP" > file.txt
git add file.txt

# Need to switch branches
git stash save "Feature work"

# Do other work
git checkout other-branch
# ... work ...
git checkout main

# Resume work
git stash list
git stash pop
```

## Reset Cheat Sheet

```bash
# Undo last commit, keep changes
git reset HEAD~1

# Undo last commit, keep changes staged
git reset --soft HEAD~1

# Undo last commit, discard changes
git reset --hard HEAD~1

# Undo multiple commits
git reset --hard HEAD~3

# Reset to specific commit
git reset --hard abc123

# Reset single file
git restore file.txt
```

## Best Practices

1. **Commit often**: Easier to undo small commits
2. **Never rewrite public history**: Use revert instead of reset
3. **Check twice before hard reset**: No undo for this!
4. **Use --force-with-lease**: Safer than --force
5. **Stash instead of committing WIP**: Keep history clean
6. **Keep reflog**: Your safety net
7. **Test before force push**: Ensure you're not breaking others
8. **Communicate force pushes**: Let team know
9. **Create branches before experiments**: Easy to discard
10. **Use revert for collaboration**: Safe for team workflows

## Safety Checklist

Before destructive operations:

```bash
# 1. Check status
git status

# 2. View what would change
git diff

# 3. Create backup branch
git branch backup-$(date +%Y%m%d)

# 4. Verify current state
git log --oneline -5

# 5. Now perform operation

# 6. If mistake, recover from backup
git checkout backup-branch
```

## Next Steps

You now know how to undo changes at every level! You can fix mistakes, recover lost work, and handle accidents confidently.

**Continue to**: [Chapter 7: Collaboration Workflows](../07-collaboration-workflows/README.md)

## Additional Resources

- [Git Reset Documentation](https://git-scm.com/docs/git-reset)
- [Git Revert Documentation](https://git-scm.com/docs/git-revert)
- [Git Reflog Documentation](https://git-scm.com/docs/git-reflog)
- [Atlassian Git Reset Tutorial](https://www.atlassian.com/git/tutorials/undoing-changes)

---

**Quick Reference Commands**

```bash
# Discard changes
git restore file.txt             # Discard file changes
git restore .                    # Discard all changes
git clean -fd                    # Remove untracked files

# Unstage
git restore --staged file.txt    # Unstage file
git restore --staged .           # Unstage all

# Amend commit
git commit --amend               # Change last commit
git commit --amend --no-edit     # Add to last commit

# Reset
git reset HEAD~1                 # Undo commit, keep changes
git reset --soft HEAD~1          # Undo commit, keep staged
git reset --hard HEAD~1          # Undo commit, discard all

# Revert
git revert abc123                # Undo commit safely

# Recover
git reflog                       # View reflog
git reset --hard abc123          # Recover commit

# Stash
git stash                        # Save changes
git stash pop                    # Apply and remove
git stash list                   # List stashes
```