# Chapter 8: Advanced Topics

## Introduction

You've mastered the fundamentals! This chapter covers advanced Git techniques used by power users. These tools solve specific problems and optimize workflows. Pick and choose what's relevant to your work—you don't need to master everything here immediately.

## Git Submodules

Submodules let you include one Git repository inside another.

### When to Use Submodules

- Shared libraries across multiple projects
- Vendor dependencies you want to track
- Separating large components

### Adding a Submodule

```bash
# Add submodule
git submodule add https://github.com/user/library.git libs/library

# This creates:
# - libs/library/ directory with the submodule
# - .gitmodules file tracking submodule config
# - Entry in .git/config

# Commit the submodule
git add .gitmodules libs/library
git commit -m "Add library submodule"
```

### Cloning Repository with Submodules

```bash
# Clone and initialize submodules
git clone https://github.com/user/project.git
cd project
git submodule init
git submodule update

# Or in one command
git clone --recursive https://github.com/user/project.git

# Or after cloning
git clone https://github.com/user/project.git
git submodule update --init --recursive
```

### Working with Submodules

```bash
# Update all submodules to latest
git submodule update --remote

# Update specific submodule
git submodule update --remote libs/library

# Work inside submodule
cd libs/library
git checkout main
git pull
cd ../..
git add libs/library
git commit -m "Update library submodule"

# View submodule status
git submodule status

# Execute command in all submodules
git submodule foreach git pull origin main
```

### Removing a Submodule

```bash
# 1. Deinitialize
git submodule deinit libs/library

# 2. Remove from .git/modules
rm -rf .git/modules/libs/library

# 3. Remove from working tree
git rm libs/library

# 4. Commit
git commit -m "Remove library submodule"
```

### Submodule Alternatives

Consider these instead:
- **Git Subtree**: Simpler, single repo approach
- **Package managers**: npm, pip, cargo, etc.
- **Mono-repo tools**: Lerna, Nx, Bazel

## Git Subtree

Subtree merges external repositories into subdirectories.

### Adding a Subtree

```bash
# Add remote
git remote add library-remote https://github.com/user/library.git

# Add subtree
git subtree add --prefix=libs/library library-remote main --squash

# This pulls library into libs/library/ and commits it
```

### Pulling Subtree Updates

```bash
# Pull updates from subtree
git subtree pull --prefix=libs/library library-remote main --squash
```

### Pushing Changes Back

```bash
# If you modify the subtree and want to contribute back
git subtree push --prefix=libs/library library-remote feature-branch
```

### Subtree vs Submodule

**Subtree advantages:**
- Simpler for team (no special commands)
- Code is actually in the repo
- Easier to clone

**Submodule advantages:**
- Clear separation
- Smaller repository size
- Each component tracked independently

## Git Worktrees

Work on multiple branches simultaneously in separate directories.

### Creating Worktrees

```bash
# Create worktree for branch
git worktree add ../project-feature feature-branch

# Create new branch in worktree
git worktree add ../project-hotfix -b hotfix/critical

# Now you have:
# project/ (main branch)
# project-feature/ (feature branch)
# project-hotfix/ (hotfix branch)
```

### Managing Worktrees

```bash
# List worktrees
git worktree list

# Remove worktree
git worktree remove ../project-feature

# Prune stale worktrees
git worktree prune

# Move worktree
git worktree move ../project-feature ../new-location
```

### Use Cases

- Review PR while working on feature
- Build/test one branch while developing another
- Hotfix production while developing new feature
- Multiple checkouts of the same repo

## Advanced Rebase Techniques

### Rebase onto Different Base

```bash
# Rebase feature onto develop instead of main
git rebase --onto develop main feature-branch

# Before:
# main: A---B
#            \
# feature:     C---D---E

# After:
# develop: X---Y
#               \
# feature:        C'---D'---E'
```

### Autosquash

```bash
# Create fixup commits
git commit --fixup=abc123
git commit --squash=abc123

# Later, autosquash during rebase
git rebase -i --autosquash main

# Git automatically marks fixup/squash commits
```

### Rebase with Exec

```bash
# Run tests after each commit during rebase
git rebase -i --exec "npm test" main

# Git will stop if tests fail at any commit
```

## Git Hooks

Automate workflows with scripts that run at specific Git events.

### Client-Side Hooks

Located in `.git/hooks/`:

**pre-commit** - Before commit is created
```bash
#!/bin/bash
# .git/hooks/pre-commit

npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed"
    exit 1
fi

npm test
if [ $? -ne 0 ]; then
    echo "Tests failed"
    exit 1
fi
```

**prepare-commit-msg** - Edit default commit message
```bash
#!/bin/bash
# .git/hooks/prepare-commit-msg

# Add issue number from branch name
BRANCH=$(git symbolic-ref --short HEAD)
ISSUE=$(echo $BRANCH | grep -oP '(?<=/)\d+(?=\-)')

if [ -n "$ISSUE" ]; then
    echo "Refs #$ISSUE" >> "$1"
fi
```

**commit-msg** - Validate commit message
```bash
#!/bin/bash
# .git/hooks/commit-msg

# Enforce conventional commits
if ! grep -qE "^(feat|fix|docs|style|refactor|test|chore)(\(.+\))?: .+" "$1"; then
    echo "Commit message must follow conventional commits format"
    exit 1
fi
```

**pre-push** - Before push
```bash
#!/bin/bash
# .git/hooks/pre-push

npm test
if [ $? -ne 0 ]; then
    echo "Tests failed, push rejected"
    exit 1
fi
```

### Server-Side Hooks

Located in `.git/hooks/` on server:

- **pre-receive**: Before accepting push
- **update**: Once per branch being updated
- **post-receive**: After successful push

### Managing Hooks

```bash
# Make hook executable
chmod +x .git/hooks/pre-commit

# Share hooks with team (they're not committed by default)
# Option 1: Use husky (npm package)
npm install husky --save-dev
npx husky install
npx husky add .git/hooks/pre-commit "npm test"

# Option 2: Store in repo and symlink
mkdir .githooks
cp .git/hooks/pre-commit .githooks/
git add .githooks/
git config core.hooksPath .githooks
```

## Git Attributes

Control how Git handles files via `.gitattributes`.

### Example .gitattributes

```gitattributes
# Ensure line endings
*.sh text eol=lf
*.bat text eol=crlf

# Mark as binary
*.png binary
*.jpg binary
*.pdf binary

# Custom diff driver
*.json diff=json

# Custom merge driver
database.sql merge=ours

# Mark generated files
package-lock.json -diff
yarn.lock -diff

# Mark as vendored (GitHub linguist)
vendor/* linguist-vendored

# Mark as documentation
docs/* linguist-documentation
```

### Common Attributes

- `text`: Treat as text file
- `binary`: Treat as binary
- `eol=lf/crlf`: Force line ending
- `-diff`: Don't show in diffs
- `merge=ours`: Always keep our version in merge
- `filter`: Run content through filter

## Git LFS (Large File Storage)

Track large files efficiently.

### Setup

```bash
# Install Git LFS
# macOS: brew install git-lfs
# Ubuntu: apt-get install git-lfs
# Windows: Download from git-lfs.github.com

# Initialize in repository
git lfs install

# Track file types
git lfs track "*.psd"
git lfs track "*.mp4"
git lfs track "design/**"

# This creates/updates .gitattributes
git add .gitattributes
git commit -m "Track large files with LFS"

# Add and commit large files normally
git add large-file.psd
git commit -m "Add design file"
git push
```

### Managing LFS Files

```bash
# View tracked patterns
git lfs track

# View tracked files
git lfs ls-files

# Fetch LFS files
git lfs fetch

# Pull LFS files
git lfs pull

# Untrack pattern
git lfs untrack "*.psd"

# Migrate existing files to LFS
git lfs migrate import --include="*.psd"
```

## Sparse Checkout

Check out only parts of a repository.

### Cone Mode (Simpler)

```bash
# Clone without checking out
git clone --filter=blob:none --no-checkout https://github.com/user/repo.git
cd repo

# Enable sparse checkout
git sparse-checkout init --cone

# Specify directories to include
git sparse-checkout set src tests

# Checkout files
git checkout main

# Add more directories
git sparse-checkout add docs
```

### Non-Cone Mode (More Control)

```bash
# Enable sparse checkout
git sparse-checkout init

# Edit .git/info/sparse-checkout
echo "src/" >> .git/info/sparse-checkout
echo "!src/deprecated/" >> .git/info/sparse-checkout
echo "README.md" >> .git/info/sparse-checkout

# Apply
git sparse-checkout reapply
```

## Advanced Git Configuration

### Useful Global Configs

```bash
# Better diffs
git config --global diff.algorithm histogram

# Auto-correct mistyped commands
git config --global help.autocorrect 10

# Always use rebase on pull
git config --global pull.rebase true

# Default push behavior
git config --global push.default current

# Prune on fetch
git config --global fetch.prune true

# Show original branch names on merge
git config --global merge.conflictStyle diff3

# Use VS Code as diff tool
git config --global diff.tool vscode
git config --global difftool.vscode.cmd 'code --wait --diff $LOCAL $REMOTE'

# Conditional includes (different config per folder)
git config --global includeIf.gitdir:~/work/.path ~/.gitconfig-work
git config --global includeIf.gitdir:~/personal/.path ~/.gitconfig-personal
```

### Repository-Specific Configs

```bash
# Set email for work projects
cd ~/work/project
git config user.email "work@company.com"

# Set different signing key
git config user.signingkey ABC123
```

## Commit Signing

Verify commit authenticity with GPG signatures.

### Setup GPG

```bash
# Generate GPG key
gpg --full-generate-key
# Choose RSA and RSA, 4096 bits

# List keys
gpg --list-secret-keys --keyid-format LONG

# Export public key
gpg --armor --export KEY_ID

# Add to GitHub: Settings → SSH and GPG keys

# Configure Git
git config --global user.signingkey KEY_ID
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

### Signing Commits

```bash
# Sign specific commit
git commit -S -m "Signed commit"

# With global config, all commits are signed
git commit -m "This will be signed"

# Verify signatures
git log --show-signature

# Verify specific commit
git verify-commit abc123
```

## Git Archive

Export project without Git history.

```bash
# Create zip archive
git archive --format=zip --output=project.zip HEAD

# Create tar.gz archive
git archive --format=tar.gz --output=project.tar.gz HEAD

# Archive specific branch
git archive --format=zip --output=develop.zip develop

# Archive with prefix
git archive --format=zip --prefix=project/ --output=project.zip HEAD

# Archive specific directory
git archive --format=zip --output=src.zip HEAD:src/

# Archive specific commit
git archive --format=zip --output=release.zip abc123
```

## Git Bundle

Share repository via file (useful for offline transfer).

```bash
# Create bundle of entire repository
git bundle create repo.bundle --all

# Create bundle of specific branch
git bundle create feature.bundle feature-branch

# Verify bundle
git bundle verify repo.bundle

# Clone from bundle
git clone repo.bundle my-repo

# Fetch from bundle
git fetch ../repo.bundle main:main
```

## Advanced Log and Search

### Pickaxe Search

```bash
# Find commits that added or removed string
git log -S "function_name" --source --all

# With regex
git log -G "def.*login" --source --all

# Show actual changes
git log -S "function_name" -p
```

### Advanced Filtering

```bash
# Commits touching specific lines
git log -L 10,20:filename.txt

# Commits in one branch but not another
git log main..feature --oneline

# Commits in either branch but not both
git log --left-right main...feature

# Find merge commits that introduced code
git log --grep="Merge" --merges

# Find commits that touch specific function
git log -L :function_name:file.js
```

## Performance Optimization

### Garbage Collection

```bash
# Manual garbage collection
git gc

# Aggressive GC (slower but more thorough)
git gc --aggressive --prune=now

# Auto GC (Git runs this automatically)
git gc --auto
```

### Repository Maintenance

```bash
# Check repository integrity
git fsck --full

# Verify objects
git fsck --unreachable

# Count objects
git count-objects -vH

# Prune unreachable objects
git prune

# Repack objects
git repack -a -d --depth=250 --window=250
```

### Shallow Clones

```bash
# Clone only recent history
git clone --depth 1 https://github.com/user/repo.git

# Clone specific number of commits
git clone --depth 50 https://github.com/user/repo.git

# Later fetch more history
git fetch --unshallow

# Deepen shallow clone
git fetch --depth=100
```

## Troubleshooting Advanced Issues

### Corrupted Repository

```bash
# Try to recover
git fsck --full

# If objects are corrupt
rm -rf .git/index
git reset

# Last resort - re-clone
mv repo repo-backup
git clone <url> repo
# Copy your uncommitted work from repo-backup
```

### Large Repository Issues

```bash
# Find large files
git rev-list --objects --all \
  | git cat-file --batch-check='%(objecttype) %(objectname) %(objectsize) %(rest)' \
  | awk '/^blob/ {print $3,$4}' \
  | sort -nr \
  | head -20

# Remove large file from history (use BFG Repo-Cleaner)
java -jar bfg.jar --delete-files large-file.zip
git reflog expire --expire=now --all
git gc --prune=now --aggressive
```

### Detached HEAD Recovery

```bash
# You're in detached HEAD state
git branch temp-branch
git checkout main
git merge temp-branch
git branch -d temp-branch
```

## Practical Exercises

### Exercise 1: Git Hooks

```bash
# Create pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
echo "Running pre-commit checks..."
# Prevent commits to main
branch=$(git symbolic-ref --short HEAD)
if [ "$branch" = "main" ]; then
    echo "Cannot commit directly to main!"
    exit 1
fi
EOF

chmod +x .git/hooks/pre-commit

# Test it
git checkout main
touch test.txt
git add test.txt
git commit -m "This will fail"
```

### Exercise 2: Git Worktrees

```bash
# Create worktrees for parallel work
git worktree add ../my-project-feature feature-branch
git worktree add ../my-project-hotfix -b hotfix/urgent

# Work in both simultaneously
cd ../my-project-feature
# ... develop feature ...

cd ../my-project-hotfix
# ... fix bug ...

# Clean up
cd ../my-project
git worktree remove ../my-project-feature
git worktree remove ../my-project-hotfix
```

### Exercise 3: Interactive Rebase

```bash
# Create messy history
git checkout -b cleanup
echo "1" > file.txt && git add . && git commit -m "WIP"
echo "2" >> file.txt && git commit -am "Fix"
echo "3" >> file.txt && git commit -am "Really fix"
echo "4" >> file.txt && git commit -am "Add feature"

# Clean it up
git rebase -i HEAD~4
# Squash first three commits, keep last one

# View clean history
git log --oneline
```

## Best Practices

1. **Don't overuse advanced features**: Simple is better
2. **Document complex setups**: Help future maintainers
3. **Test hooks thoroughly**: They can block work
4. **Use LFS for binaries**: Keep repo size manageable
5. **Sign important commits**: Especially releases
6. **Regular maintenance**: Run `git gc` occasionally
7. **Backup before experiments**: Advanced features can go wrong
8. **Know when to use what**: Right tool for the job

## Common Advanced Workflows

### Release Management

```bash
# Create release branch
git checkout -b release/1.2.0 develop

# Bump version, update changelog
# ... commits ...

# Merge to main and tag
git checkout main
git merge release/1.2.0
git tag -a v1.2.0 -m "Release 1.2.0"

# Merge back to develop
git checkout develop
git merge release/1.2.0

# Push everything
git push origin main develop --tags
```

### Maintaining Fork

```bash
# Setup
git remote add upstream https://github.com/original/repo.git

# Regular sync
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# Rebase your feature on latest upstream
git checkout feature-branch
git rebase upstream/main
git push --force-with-lease origin feature-branch
```

## Additional Tools

### Git Aliases

```bash
# Useful aliases
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.visual 'log --graph --oneline --all'
git config --global alias.aliases 'config --get-regexp alias'
```

### Third-Party Tools

- **lazygit**: Terminal UI for Git
- **tig**: Text-mode interface
- **GitHub CLI**: `gh` command-line tool
- **git-extras**: Extra Git utilities
- **git-flow**: Git Flow helper
- **BFG Repo-Cleaner**: Clean up repositories

## Conclusion

Congratulations! You've completed the Git learning guide. You now have:

✅ Understanding of version control fundamentals  
✅ Mastery of basic Git operations  
✅ Skills to work with branches and merges  
✅ Ability to collaborate via remote repositories  
✅ Knowledge of viewing and understanding history  
✅ Confidence in undoing mistakes  
✅ Experience with professional workflows  
✅ Advanced techniques for complex scenarios  

## Next Steps

1. **Practice regularly**: Use Git daily
2. **Contribute to open source**: Real-world experience
3. **Explore your specific needs**: Deep dive into relevant topics
4. **Stay updated**: Git evolves, follow new features
5. **Help others**: Teaching reinforces your knowledge

## Final Resources

- [Pro Git Book](https://git-scm.com/book) - Comprehensive reference
- [Git Documentation](https://git-scm.com/doc) - Official docs
- [Oh My Git!](https://ohmygit.org/) - Interactive game
- [Learn Git Branching](https://learngitbranching.js.org/) - Visual learning
- [Git Exercises](https://gitexercises.fracz.com/) - Practice problems

---

**Keep Learning and Happy Gitting!** 🎉

---

**Quick Reference**

```bash
# Submodules
git submodule add <url> <path>
git submodule update --init --recursive

# Worktrees
git worktree add <path> <branch>
git worktree list

# Hooks
chmod +x .git/hooks/pre-commit

# LFS
git lfs track "*.psd"
git lfs ls-files

# Archive
git archive --format=zip -o file.zip HEAD

# Advanced config
git config --global pull.rebase true
git config --global fetch.prune true
```