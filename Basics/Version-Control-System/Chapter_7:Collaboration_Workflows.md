# Chapter 7: Collaboration Workflows

## Introduction

Git shines when working with teams. This chapter covers collaboration workflows, best practices for working with others, pull request processes, code review, and how to maintain a healthy team repository. You'll learn established patterns that professional teams use daily.

## Common Collaboration Workflows

### Centralized Workflow

Simplest workflow - everyone pushes to a single branch (usually `main`).

**Process:**
```bash
# 1. Pull latest changes
git pull origin main

# 2. Make your changes
# ... edit files ...
git add .
git commit -m "Add feature"

# 3. Pull again (in case others pushed)
git pull origin main

# 4. Push your changes
git push origin main
```

**Pros:**
- Simple to understand
- Good for small teams
- Minimal overhead

**Cons:**
- No code review by default
- Direct commits to main
- Can have conflicts if many people work simultaneously

### Feature Branch Workflow

Each new feature gets its own branch. Merge to main when complete.

**Process:**
```bash
# 1. Update main
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/user-authentication

# 3. Work on feature
# ... make commits ...
git commit -am "Add login form"
git commit -am "Add authentication logic"

# 4. Push feature branch
git push -u origin feature/user-authentication

# 5. Create Pull Request on GitHub/GitLab

# 6. After review and approval, merge to main
git checkout main
git pull origin main
git merge feature/user-authentication

# 7. Push merged main
git push origin main

# 8. Delete feature branch
git branch -d feature/user-authentication
git push origin --delete feature/user-authentication
```

**Pros:**
- Isolated development
- Code review via pull requests
- Main branch stays stable
- Easy to see what everyone is working on

**Cons:**
- Slightly more complex
- Requires discipline

**Best for:** Most teams, most projects

### Git Flow

Structured workflow with multiple long-lived branches.

**Branches:**
- `main` - Production code
- `develop` - Integration branch
- `feature/*` - Feature branches (from develop)
- `release/*` - Release preparation (from develop)
- `hotfix/*` - Emergency fixes (from main)

**Process:**

**Feature development:**
```bash
# Start feature
git checkout develop
git checkout -b feature/new-feature

# Work on feature
git commit -am "Build feature"

# Finish feature
git checkout develop
git merge feature/new-feature
git push origin develop
git branch -d feature/new-feature
```

**Release:**
```bash
# Start release
git checkout develop
git checkout -b release/1.2.0

# Prepare release (version bumps, changelog)
git commit -am "Bump version to 1.2.0"

# Merge to main
git checkout main
git merge release/1.2.0
git tag -a v1.2.0 -m "Release 1.2.0"
git push origin main --tags

# Merge back to develop
git checkout develop
git merge release/1.2.0
git push origin develop

# Delete release branch
git branch -d release/1.2.0
```

**Hotfix:**
```bash
# Create hotfix from main
git checkout main
git checkout -b hotfix/critical-bug

# Fix bug
git commit -am "Fix critical bug"

# Merge to main
git checkout main
git merge hotfix/critical-bug
git tag -a v1.2.1 -m "Hotfix 1.2.1"
git push origin main --tags

# Merge to develop
git checkout develop
git merge hotfix/critical-bug
git push origin develop

# Delete hotfix branch
git branch -d hotfix/critical-bug
```

**Pros:**
- Very structured
- Clear release process
- Good for scheduled releases

**Cons:**
- Complex for beginners
- Overhead for simple projects

**Best for:** Large teams, products with scheduled releases

### Forking Workflow

Used in open source - contributors fork the repository.

**Process:**

**Initial setup:**
```bash
# 1. Fork repository on GitHub (web interface)

# 2. Clone your fork
git clone git@github.com:your-username/project.git
cd project

# 3. Add upstream remote
git remote add upstream git@github.com:original-owner/project.git

# 4. Verify remotes
git remote -v
# origin    git@github.com:your-username/project.git
# upstream  git@github.com:original-owner/project.git
```

**Making contributions:**
```bash
# 1. Sync with upstream
git fetch upstream
git checkout main
git merge upstream/main
git push origin main

# 2. Create feature branch
git checkout -b fix/documentation-typo

# 3. Make changes
git commit -am "Fix typo in README"

# 4. Push to your fork
git push -u origin fix/documentation-typo

# 5. Create Pull Request on GitHub
# From: your-username:fix/documentation-typo
# To: original-owner:main
```

**Pros:**
- No write access needed to main repo
- Safe for open source projects
- Clear ownership

**Cons:**
- More complex setup
- Extra remote to manage

**Best for:** Open source projects, external contributors

### Trunk-Based Development

Everyone commits to main (trunk) frequently, using feature flags.

**Process:**
```bash
# 1. Pull latest
git pull origin main

# 2. Make small change
# ... edit files ...
git commit -am "Add feature (behind feature flag)"

# 3. Push directly to main
git push origin main

# Or use very short-lived branches (<1 day)
git checkout -b temp-feature
git commit -am "Quick feature"
git checkout main
git merge temp-feature
git push origin main
git branch -d temp-feature
```

**Key practices:**
- Commit to main multiple times per day
- Use feature flags for incomplete features
- Rely on automated testing
- Short-lived branches (hours, not days)

**Pros:**
- Fast integration
- Fewer merge conflicts
- Encourages small changes
- Continuous integration

**Cons:**
- Requires excellent CI/CD
- Needs feature flag discipline
- Less code review opportunity

**Best for:** Teams with strong CI/CD, fast-moving projects

## Pull Request Best Practices

### Creating Good Pull Requests

**Before creating:**
```bash
# 1. Update your branch with main
git checkout main
git pull origin main
git checkout feature-branch
git merge main

# Or rebase
git rebase main

# 2. Ensure tests pass
npm test  # or your test command

# 3. Review your own changes
git diff main..feature-branch

# 4. Clean up commits if needed
git rebase -i main
```

**PR Description Template:**
```markdown
## Description
Brief description of what this PR does.

## Changes
- Added user authentication
- Updated user model
- Added authentication tests

## Type of change
- [ ] Bug fix
- [x] New feature
- [ ] Breaking change
- [ ] Documentation update

## Testing
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Related Issues
Closes #123
Related to #456

## Screenshots (if applicable)
[Add screenshots here]

## Checklist
- [x] Code follows project style guidelines
- [x] Self-review completed
- [x] Comments added for complex code
- [x] Documentation updated
- [x] No new warnings generated
```

### PR Size Guidelines

**Ideal PR:**
- 100-400 lines changed
- Single focused purpose
- Can be reviewed in < 30 minutes

**Too large:**
- > 1000 lines
- Multiple features
- Consider splitting

**How to keep PRs small:**
```bash
# Break work into smaller pieces
git checkout -b feature/auth-part1-models
# ... implement just models ...
git push -u origin feature/auth-part1-models
# Create PR, get it merged

git checkout main
git pull origin main
git checkout -b feature/auth-part2-controllers
# ... implement controllers ...
```

### Responding to Review Feedback

```bash
# 1. Address feedback
# ... make changes ...
git add .
git commit -m "Address review feedback"

# 2. Push to same branch
git push origin feature-branch
# PR updates automatically

# 3. Mark conversations as resolved on GitHub

# 4. Request re-review
```

**If reviewer suggests major changes:**
```bash
# Option 1: Amend commits (if not pushed)
git add .
git commit --amend --no-edit

# Option 2: Make new commit (if pushed)
git add .
git commit -m "Refactor based on feedback"

# Option 3: Interactive rebase to clean up
git rebase -i main
# Squash feedback commits
```

## Code Review

### As a Reviewer

**What to look for:**
- Functionality: Does it work?
- Testing: Are there tests? Do they pass?
- Security: Any security concerns?
- Performance: Any performance issues?
- Maintainability: Is it readable and maintainable?
- Style: Follows project conventions?

**Good review comments:**
```markdown
# ✅ Good - Specific and constructive
Could we extract this logic into a separate function? It would make it 
easier to test and reuse.

# ✅ Good - Asks questions
Why did we choose this approach over using the existing utility function?

# ✅ Good - Offers suggestions
```suggestion
const result = data.map(item => item.value);
```

# ❌ Bad - Vague
This is confusing.

# ❌ Bad - Personal attack
This is terrible code.
```

**Review workflow:**
```bash
# 1. Check out the PR branch
git fetch origin
git checkout feature-branch

# 2. Run tests
npm test

# 3. Try it locally
npm start

# 4. Review the code
git diff main..feature-branch

# 5. Leave review on GitHub
```

### As an Author

**Preparing for review:**
```bash
# Self-review first
git diff main..feature-branch

# Run linter
npm run lint

# Run tests
npm test

# Check for debugging code
git diff main..feature-branch | grep -i "console.log\|debugger\|TODO"
```

**Handling feedback:**
- Thank reviewers
- Ask for clarification if needed
- Don't take it personally
- Learn from feedback
- Apply lessons to future PRs

## Handling Merge Conflicts

### During Pull/Merge

```bash
# Conflict occurs
$ git pull origin main
Auto-merging file.txt
CONFLICT (content): Merge conflict in file.txt
Automatic merge failed; fix conflicts and then commit the result.

# 1. View conflicted files
git status

# 2. Open each file and resolve
# Look for conflict markers:
<<<<<<< HEAD
Your changes
=======
Their changes
>>>>>>> origin/main

# 3. Edit to resolve, remove markers

# 4. Mark as resolved
git add file.txt

# 5. Complete merge
git commit  # Editor opens with merge message

# Or if pulling
git pull origin main  # Will complete automatically
```

### During Rebase

```bash
# Conflict during rebase
$ git rebase main
CONFLICT (content): Merge conflict in file.txt

# 1. Resolve conflict in file

# 2. Stage resolved file
git add file.txt

# 3. Continue rebase
git rebase --continue

# If you want to skip this commit
git rebase --skip

# If you want to abort
git rebase --abort
```

### Preventing Conflicts

**Best practices:**
```bash
# 1. Pull frequently
git pull origin main  # Daily or more

# 2. Keep branches short-lived
# Merge within a few days

# 3. Update feature branch regularly
git checkout feature-branch
git merge main

# 4. Communicate with team
# Know who's working on what files

# 5. Small, focused commits
# Easier to resolve conflicts
```

### Complex Conflicts

**Use merge tools:**
```bash
# Configure merge tool
git config --global merge.tool meld
# or vimdiff, kdiff3, p4merge, etc.

# Use it
git mergetool

# Common tools:
# - VS Code (built-in)
# - Meld (visual, cross-platform)
# - P4Merge (free from Perforce)
# - KDiff3 (free, cross-platform)
```

**Choose a version entirely:**
```bash
# Keep their version
git checkout --theirs file.txt
git add file.txt

# Keep our version  
git checkout --ours file.txt
git add file.txt
```

## Branch Protection

Set up rules on GitHub/GitLab to enforce quality.

### Common Protection Rules

**Via GitHub UI:**

Settings → Branches → Add rule

- **Require pull request reviews**: N reviewers must approve
- **Require status checks**: Tests must pass
- **Require signed commits**: Commits must be signed
- **Include administrators**: Rules apply to admins too
- **Restrict who can push**: Only certain people can merge

### Local Branch Policies

Set up pre-commit hooks:

```bash
# .git/hooks/pre-commit
#!/bin/bash

# Run tests
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed. Commit rejected."
    exit 1
fi

# Run linter
npm run lint
if [ $? -ne 0 ]; then
    echo "Linting failed. Commit rejected."
    exit 1
fi

exit 0
```

Make it executable:
```bash
chmod +x .git/hooks/pre-commit
```

## Continuous Integration

### Basic CI Setup (GitHub Actions)

`.github/workflows/test.yml`:
```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v2
    
    - name: Setup Node.js
      uses: actions/setup-node@v2
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: npm install
    
    - name: Run tests
      run: npm test
    
    - name: Run linter
      run: npm run lint
```

### Integrating with PRs

```bash
# 1. Create PR
git push -u origin feature-branch

# 2. CI runs automatically

# 3. If CI fails, fix issues
git commit -am "Fix failing tests"
git push origin feature-branch

# 4. CI runs again

# 5. Merge when green ✅
```

## Team Communication

### Commit Message Conventions

**Conventional Commits:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting, no code change
- `refactor`: Code restructuring
- `test`: Adding tests
- `chore`: Build process, tools

**Examples:**
```bash
git commit -m "feat(auth): add password reset functionality"
git commit -m "fix(api): handle null response from database"
git commit -m "docs(readme): update installation instructions"
git commit -m "refactor(utils): simplify date formatting logic"
```

### PR Title Conventions

```markdown
# Good titles
feat: Add user profile page
fix: Resolve login redirect issue
docs: Update API documentation
refactor: Simplify authentication logic

# Bad titles
Updates
Fix stuff
WIP
Changes
```

### Using Issues

Link PRs to issues:
```markdown
## PR Description
This PR implements user authentication.

Closes #123
Fixes #124
Related to #125
```

Git/GitHub keywords:
- `Closes #123` - Closes issue when PR merges
- `Fixes #123` - Same as closes
- `Resolves #123` - Same as closes
- `Related to #123` - Links without closing

## Practical Exercises

### Exercise 1: Feature Branch Workflow

```bash
# Simulate team workflow
git init team-project
cd team-project

# Setup
echo "# Project" > README.md
git add README.md
git commit -m "Initial commit"

# Create feature
git checkout -b feature/add-login
echo "Login page" > login.html
git add login.html
git commit -m "Add login page"

# Meanwhile, main updated (simulate teammate)
git checkout main
echo "Update" >> README.md
git commit -am "Update README"

# Update feature branch
git checkout feature/add-login
git merge main

# Merge feature
git checkout main
git merge feature/add-login

# Clean up
git branch -d feature/add-login
```

### Exercise 2: Handling Conflicts

```bash
# Create conflict scenario
git checkout -b branch-a
echo "Version A" > conflict.txt
git add conflict.txt
git commit -m "Add version A"

git checkout main
git checkout -b branch-b
echo "Version B" > conflict.txt
git add conflict.txt
git commit -m "Add version B"

# Merge branch-a first
git checkout main
git merge branch-a

# Try merge branch-b (conflict!)
git merge branch-b

# Resolve
# Edit conflict.txt
git add conflict.txt
git commit -m "Merge branch-b, resolve conflict"
```

### Exercise 3: Fork Workflow

```bash
# Simulate fork (use two directories)
mkdir original-repo
cd original-repo
git init
echo "# Original" > README.md
git add README.md
git commit -m "Initial commit"

# Simulate fork
cd ..
git clone original-repo forked-repo
cd forked-repo

# Add upstream
git remote add upstream ../original-repo

# Make contribution
git checkout -b fix/typo
echo "Fixed" >> README.md
git commit -am "Fix typo"
git push origin fix/typo

# Simulate upstream changes
cd ../original-repo
echo "Upstream change" >> README.md
git commit -am "Upstream update"

# Sync fork
cd ../forked-repo
git fetch upstream
git checkout main
git merge upstream/main
```

## Best Practices Summary

1. **Pull before push**: Always update before pushing
2. **Small PRs**: Easier to review and merge
3. **Descriptive commits**: Help future developers understand
4. **Test before PR**: Don't submit broken code
5. **Review your own PR first**: Catch obvious issues
6. **Respond to feedback quickly**: Keep PRs moving
7. **Update branches regularly**: Avoid large conflicts
8. **Delete merged branches**: Keep repo clean
9. **Communicate**: Let team know what you're working on
10. **Document conventions**: Write them down

## Next Steps

You now understand professional Git workflows! You can collaborate effectively, handle pull requests, and work with teams.

**Continue to**: [Chapter 8: Advanced Topics](../08-advanced-topics/README.md)

## Additional Resources

- [Git Flow](https://nvie.com/posts/a-successful-git-branching-model/)
- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [Trunk Based Development](https://trunkbaseddevelopment.com/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)

---

**Quick Reference**

```bash
# Feature Branch Workflow
git checkout main
git pull origin main
git checkout -b feature/name
# ... work ...
git push -u origin feature/name
# Create PR, merge, then:
git checkout main
git pull origin main
git branch -d feature/name

# Keep branch updated
git checkout feature-branch
git merge main
# or
git rebase main

# Resolve conflicts
# Edit files
git add resolved-file.txt
git commit  # or git rebase --continue
```