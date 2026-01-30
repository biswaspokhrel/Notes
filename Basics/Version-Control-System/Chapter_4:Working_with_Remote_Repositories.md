# Chapter 4: Working with Remote Repositories

## Introduction

Remote repositories enable collaboration. They're versions of your project hosted on the internet or network, allowing you to share code, back up your work, and collaborate with others. This chapter covers everything you need to work effectively with remote repositories like GitHub, GitLab, and Bitbucket.

## What is a Remote Repository?

A remote is a version of your repository hosted elsewhere. Think of it as a shared copy that everyone on your team can access. While your local repository is on your computer, remotes live on servers.

### Why Use Remotes?

- **Collaboration**: Multiple people can work on the same project
- **Backup**: Your code is safe even if your computer dies
- **Sharing**: Easy to share code with the world
- **Deployment**: Many hosting services deploy from Git repositories
- **Code Review**: Platforms like GitHub enable pull request workflows

## Popular Remote Hosting Services

### GitHub
- Most popular platform
- Free public and private repositories
- Strong community and social features
- GitHub Actions for CI/CD
- [github.com](https://github.com)

### GitLab
- Open source alternative
- Self-hosted option available
- Built-in CI/CD
- Free private repositories
- [gitlab.com](https://gitlab.com)

### Bitbucket
- Integrates with Atlassian tools (Jira, Confluence)
- Free for small teams
- Strong support for Git and Mercurial
- [bitbucket.org](https://bitbucket.org)

## Setting Up SSH Keys (Recommended)

SSH keys provide secure authentication without entering passwords constantly.

### Check for Existing SSH Keys

```bash
ls -la ~/.ssh
# Look for id_rsa.pub, id_ed25519.pub, or similar
```

### Generate SSH Key

```bash
# Generate new SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Or for older systems
ssh-keygen -t rsa -b 4096 -C "your.email@example.com"

# Press Enter to accept default location
# Enter a passphrase (recommended) or leave empty
```

### Add SSH Key to SSH Agent

**macOS/Linux:**
```bash
# Start ssh-agent
eval "$(ssh-agent -s)"

# Add your key
ssh-add ~/.ssh/id_ed25519
```

**Windows (Git Bash):**
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

### Add SSH Key to GitHub/GitLab

1. Copy your public key:
```bash
# macOS
pbcopy < ~/.ssh/id_ed25519.pub

# Linux
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard

# Windows (Git Bash)
cat ~/.ssh/id_ed25519.pub | clip

# Or just display it
cat ~/.ssh/id_ed25519.pub
```

2. **GitHub**: Settings → SSH and GPG keys → New SSH key → Paste → Add
3. **GitLab**: Preferences → SSH Keys → Paste → Add key

### Test SSH Connection

```bash
# GitHub
ssh -T git@github.com

# GitLab
ssh -T git@gitlab.com

# You should see a success message
```

## Working with Remotes

### Viewing Remotes

```bash
# List remotes
git remote

# List remotes with URLs
git remote -v

# Detailed info about a remote
git remote show origin
```

### Adding a Remote

```bash
# Add a remote named 'origin'
git remote add origin git@github.com:username/repository.git

# Or with HTTPS
git remote add origin https://github.com/username/repository.git

# Add additional remotes
git remote add upstream git@github.com:original-owner/repository.git
```

**Convention**: The default remote is typically named `origin`.

### Changing Remote URL

```bash
# Change to SSH
git remote set-url origin git@github.com:username/repository.git

# Change to HTTPS
git remote set-url origin https://github.com/username/repository.git

# Verify the change
git remote -v
```

### Removing a Remote

```bash
git remote remove origin
# or
git remote rm origin
```

### Renaming a Remote

```bash
git remote rename old-name new-name
```

## Cloning Repositories

Cloning creates a local copy of a remote repository.

### Basic Clone

```bash
# Clone with SSH (recommended)
git clone git@github.com:username/repository.git

# Clone with HTTPS
git clone https://github.com/username/repository.git

# Clone into specific folder
git clone git@github.com:username/repository.git my-folder

# Clone specific branch
git clone -b develop git@github.com:username/repository.git

# Shallow clone (only recent history)
git clone --depth 1 git@github.com:username/repository.git
```

When you clone, Git automatically:
- Creates a remote named `origin`
- Sets up tracking for the default branch
- Checks out the default branch

## Fetching Changes

Fetching downloads commits, files, and refs from a remote repository without merging.

### Basic Fetch

```bash
# Fetch from origin
git fetch origin

# Fetch from all remotes
git fetch --all

# Fetch specific branch
git fetch origin main

# Fetch and prune deleted branches
git fetch --prune
```

**Fetch vs Pull**: Fetch downloads changes but doesn't merge them. Pull fetches and merges.

### Viewing Fetched Changes

```bash
# See what was fetched
git log origin/main

# Compare with your branch
git log main..origin/main

# See diff
git diff main origin/main
```

## Pulling Changes

Pulling fetches changes and merges them into your current branch.

### Basic Pull

```bash
# Pull from tracked remote branch
git pull

# Pull from specific remote and branch
git pull origin main

# Pull with rebase instead of merge
git pull --rebase

# Pull all branches
git pull --all
```

**Pull = Fetch + Merge**:
```bash
# These are equivalent
git pull origin main

git fetch origin
git merge origin/main
```

### Pull with Rebase

```bash
# Rebase instead of merge
git pull --rebase origin main

# Make rebase the default
git config --global pull.rebase true
```

This creates a cleaner history by replaying your commits on top of the remote changes.

### Handling Pull Conflicts

If a pull creates conflicts:

```bash
# Conflicts appear - resolve them
git status  # See conflicted files
# ... edit files to resolve conflicts ...
git add resolved-file.txt

# If merging
git commit

# If rebasing
git rebase --continue
```

## Pushing Changes

Pushing uploads your local commits to a remote repository.

### Basic Push

```bash
# Push to tracked remote branch
git push

# Push to specific remote and branch
git push origin main

# Push and set upstream tracking
git push -u origin main
# or
git push --set-upstream origin main

# Push all branches
git push --all origin

# Push tags
git push --tags
```

### First Push of New Branch

```bash
# Create and push new branch
git checkout -b feature-login
# ... make commits ...
git push -u origin feature-login

# Now just 'git push' will work for this branch
```

### Force Push (Use with Caution!)

```bash
# Overwrite remote history (DANGEROUS!)
git push --force origin main

# Safer force push (fails if remote has changes you don't have)
git push --force-with-lease origin main
```

**Warning**: Force pushing rewrites remote history. Only do this on branches you own!

### Deleting Remote Branches

```bash
# Delete remote branch
git push origin --delete feature-old

# Alternative syntax
git push origin :feature-old
```

## Tracking Branches

Tracking branches are local branches that have a direct relationship to a remote branch.

### Set Upstream Tracking

```bash
# When pushing
git push -u origin feature-login

# For existing branch
git branch --set-upstream-to=origin/feature-login feature-login
# or shorter
git branch -u origin/feature-login

# Check tracking
git branch -vv
```

### Checking Out Remote Branches

```bash
# List remote branches
git branch -r

# Create local branch tracking remote
git checkout -b feature-login origin/feature-login

# Simpler (Git auto-creates tracking)
git checkout feature-login

# or
git switch feature-login
```

## Working with Forks

Forks are personal copies of someone else's repository.

### Fork Workflow

1. **Fork on GitHub** (use website button)

2. **Clone your fork**:
```bash
git clone git@github.com:your-username/repository.git
cd repository
```

3. **Add upstream remote**:
```bash
git remote add upstream git@github.com:original-owner/repository.git
```

4. **Verify remotes**:
```bash
git remote -v
# origin    git@github.com:your-username/repository.git
# upstream  git@github.com:original-owner/repository.git
```

5. **Keep fork updated**:
```bash
# Fetch upstream changes
git fetch upstream

# Merge into your main
git checkout main
git merge upstream/main

# Push to your fork
git push origin main
```

6. **Create feature branch**:
```bash
git checkout -b feature-awesome
# ... make changes and commit ...
git push -u origin feature-awesome
```

7. **Create Pull Request** on GitHub

## Pull Requests / Merge Requests

Pull Requests (GitHub) or Merge Requests (GitLab) are requests to merge your changes into another branch.

### Creating a Pull Request (GitHub)

1. Push your branch:
```bash
git push -u origin feature-login
```

2. Go to repository on GitHub

3. Click "Pull Request" or "Compare & pull request"

4. Fill in:
   - Title (descriptive)
   - Description (what changed and why)
   - Base branch (usually `main`)
   - Compare branch (your feature branch)

5. Click "Create Pull Request"

### PR Best Practices

- **Descriptive title**: "Add user authentication" not "Updates"
- **Clear description**: What, why, how
- **Link issues**: "Closes #123"
- **Keep focused**: One feature/fix per PR
- **Small PRs**: Easier to review
- **Update regularly**: Keep in sync with base branch
- **Respond to feedback**: Address review comments

### Updating a Pull Request

```bash
# Make changes based on feedback
git add .
git commit -m "Address review comments"

# Push to same branch
git push origin feature-login

# PR updates automatically
```

### Keeping PR Updated with Main

```bash
git checkout main
git pull origin main

git checkout feature-login
git merge main
# or
git rebase main

git push origin feature-login
```

## Tags

Tags mark specific points in history, often used for releases.

### Creating Tags

```bash
# Lightweight tag
git tag v1.0.0

# Annotated tag (recommended)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag specific commit
git tag -a v0.9.0 abc123 -m "Beta release"
```

### Listing Tags

```bash
# List all tags
git tag

# Search for tags
git tag -l "v1.*"

# Show tag details
git show v1.0.0
```

### Pushing Tags

```bash
# Push specific tag
git push origin v1.0.0

# Push all tags
git push origin --tags

# Push annotated tags only
git push --follow-tags
```

### Deleting Tags

```bash
# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0
# or
git push origin :refs/tags/v1.0.0
```

### Checking Out Tags

```bash
# View tag (detached HEAD state)
git checkout v1.0.0

# Create branch from tag
git checkout -b hotfix-v1.0.0 v1.0.0
```

## Practical Exercises

### Exercise 1: Create and Push to GitHub

```bash
# Create local repository
mkdir my-project
cd my-project
git init
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit"

# Create repository on GitHub (via website)
# Then add remote and push
git remote add origin git@github.com:username/my-project.git
git branch -M main
git push -u origin main
```

### Exercise 2: Fork and Contribute

```bash
# Fork a repository on GitHub

# Clone your fork
git clone git@github.com:your-username/repository.git
cd repository

# Add upstream
git remote add upstream git@github.com:original-owner/repository.git

# Create feature branch
git checkout -b fix-typo

# Make changes
echo "Fixed content" > file.txt
git commit -am "Fix typo in documentation"

# Push to your fork
git push -u origin fix-typo

# Create Pull Request on GitHub
```

### Exercise 3: Sync Fork with Upstream

```bash
# Fetch upstream changes
git fetch upstream

# Switch to main
git checkout main

# Merge upstream changes
git merge upstream/main

# Push to your fork
git push origin main
```

### Exercise 4: Working with Tags

```bash
# Create annotated tag
git tag -a v1.0.0 -m "First release"

# Push tag
git push origin v1.0.0

# List tags
git tag

# Create branch from tag
git checkout -b hotfix-v1.0.0 v1.0.0
```

## Common Remote Workflows

### Centralized Workflow

Everyone pushes to a central repository:

```bash
# Get latest changes
git pull origin main

# Make changes and commit
git commit -am "Add feature"

# Push
git push origin main
```

### Feature Branch Workflow

Each feature gets its own branch:

```bash
# Create feature branch
git checkout -b feature-awesome

# Work and commit
git commit -am "Build awesome feature"

# Push feature branch
git push -u origin feature-awesome

# Create Pull Request
# After merge, delete branch
git checkout main
git pull origin main
git branch -d feature-awesome
git push origin --delete feature-awesome
```

### Forking Workflow

For open source contributions:

```bash
# Keep fork synced
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# Create feature branch
git checkout -b contribution

# Make changes and push
git push -u origin contribution

# Create Pull Request to upstream
```

## Common Mistakes & Solutions

### Mistake: Pushed to Wrong Branch

**Solution**: Revert on remote
```bash
# Revert the commits
git revert abc123

# Or reset and force push (if no one pulled yet)
git reset --hard HEAD~1
git push --force-with-lease
```

### Mistake: Diverged Branches

```bash
# Your branch and remote have diverged
# Option 1: Pull and merge
git pull origin main

# Option 2: Pull and rebase
git pull --rebase origin main
```

### Mistake: Committed Sensitive Data

**Solution**: Remove from history
```bash
# Remove file
git rm --cached secrets.txt
git commit -m "Remove secrets"

# For all history, use BFG Repo-Cleaner
# https://rtyley.github.io/bfg-repo-cleaner/

# Force push (everyone must re-clone!)
git push --force-with-lease
```

### Mistake: Rejected Push

```bash
# Remote has changes you don't have
! [rejected]        main -> main (non-fast-forward)

# Solution: Pull first
git pull origin main
git push origin main
```

## Best Practices

1. **Pull before you push**: Avoid merge conflicts
2. **Use SSH keys**: More secure and convenient
3. **Commit before pulling**: Don't pull into dirty working directory
4. **Feature branches**: Keep main clean
5. **Small, frequent pushes**: Easier collaboration
6. **Descriptive commit messages**: Help collaborators understand changes
7. **Never force push shared branches**: You'll break others' work
8. **Review before pushing**: Use `git log` and `git diff`
9. **Use pull requests**: Enable code review
10. **Keep forks synced**: Stay up-to-date with upstream

## Troubleshooting

### Can't Push - Permission Denied

```bash
# Check remote URL
git remote -v

# Ensure you have SSH key set up
ssh -T git@github.com

# Or switch to HTTPS
git remote set-url origin https://github.com/username/repo.git
```

### Large Files Problem

```bash
# File too large for GitHub (>100MB)
# Use Git LFS
git lfs install
git lfs track "*.psd"
git add .gitattributes
git add large-file.psd
git commit -m "Add large file with LFS"
git push
```

### Rejected Push - Out of Date

```bash
# Pull changes first
git pull origin main

# Resolve any conflicts
# Then push
git push origin main
```

## Next Steps

You now know how to collaborate with others using remote repositories! You can clone, push, pull, fork, and contribute to projects.

**Continue to**: [Chapter 5: Viewing History](../05-viewing-history/README.md)

## Additional Resources

- [GitHub Guides](https://guides.github.com/)
- [GitLab Documentation](https://docs.gitlab.com/)
- [Setting up SSH Keys](https://docs.github.com/en/authentication/connecting-to-github-with-ssh)
- [About Pull Requests](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)

---

**Quick Reference Commands**

```bash
# Remotes
git remote -v                          # List remotes
git remote add origin <url>            # Add remote
git remote set-url origin <url>        # Change URL
git remote remove origin               # Remove remote

# Cloning
git clone <url>                        # Clone repository
git clone -b branch <url>              # Clone specific branch

# Fetching
git fetch origin                       # Fetch from origin
git fetch --all                        # Fetch from all remotes

# Pulling
git pull                               # Fetch and merge
git pull --rebase                      # Fetch and rebase

# Pushing
git push origin main                   # Push to main
git push -u origin feature             # Push and track
git push --force-with-lease            # Safe force push
git push origin --delete branch        # Delete remote branch

# Tags
git tag v1.0.0                         # Create tag
git push origin v1.0.0                 # Push tag
git push --tags                        # Push all tags

# Tracking
git branch -u origin/main              # Set upstream
git branch -vv                         # Show tracking
```