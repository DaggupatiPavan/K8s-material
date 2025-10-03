# Git - Version Control System

## Overview

Git is a distributed version control system that tracks changes in source code during software development. It is designed to handle everything from small to very large projects with speed and efficiency.

### Key Features
- **Distributed**: Every developer has a full copy of the repository
- **Fast**: Most operations are performed locally
- **Branching**: Lightweight branching mechanism
- **Staging Area**: Intermediate area between working directory and repository
- **Data Integrity**: Ensures code integrity through cryptographic hashing

## Core Concepts

### Repository
A Git repository is a complete directory structure containing:
- Working directory (your files)
- Staging area (index)
- Git directory (`.git` folder with all metadata)

### Three States
1. **Working Directory**: Files you see and edit
2. **Staging Area**: Files prepared for next commit
3. **Git Directory**: Database where project history is stored

### Branching Model
- **Main/Master**: Primary production branch
- **Feature Branches**: For new features
- **Develop**: Integration branch for features
- **Release**: Preparation for production release
- **Hotfix**: Emergency fixes to production

## Common Commands

### Setup & Configuration
```bash
# Configure Git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Check configuration
git config --list

# Initialize repository
git init

# Clone repository
git clone <repository-url>
```

### Basic Workflow
```bash
# Check status
git status

# Add files to staging area
git add <file-name>
git add .           # Add all files
git add -A          # Add all files including deleted

# Commit changes
git commit -m "Commit message"

# View commit history
git log
git log --oneline  # Compact view
git log --graph    # Visual branch history

# View changes
git diff                    # Working directory vs staging
git diff --staged          # Staging vs last commit
git diff <branch1> <branch2> # Compare branches
```

### Branching & Merging
```bash
# List branches
git branch

# Create new branch
git branch <branch-name>

# Switch to branch
git checkout <branch-name>

# Create and switch to branch
git checkout -b <branch-name>

# Merge branch
git merge <branch-name>

# Delete branch
git branch -d <branch-name>    # Safe delete
git branch -D <branch-name>    # Force delete
```

### Remote Operations
```bash
# Add remote repository
git remote add origin <repository-url>

# View remotes
git remote -v

# Push changes
git push origin <branch-name>

# Pull changes
git pull origin <branch-name>

# Fetch changes without merging
git fetch origin
```

### Advanced Commands
```bash
# Stash changes
git stash
git stash pop
git stash list

# Reset commands
git reset --soft HEAD~1    # Keep changes in staging
git reset --hard HEAD~1    # Remove changes completely

# Revert commit
git revert <commit-hash>

# Tagging
git tag -a v1.0 -m "Version 1.0"
git push origin --tags
```

## Git Workflow Strategies

### Centralized Workflow
- Single main branch
- All developers commit to main
- Simple but risky for larger teams

### Feature Branch Workflow
- Main branch remains stable
- Features developed in separate branches
- Merged via pull requests

### Gitflow Workflow
- Main: Production-ready code
- Develop: Integration branch
- Feature: New features
- Release: Preparation for release
- Hotfix: Emergency production fixes

### Forking Workflow
- Developers fork repository
- Work on personal copies
- Contribute via pull requests

## Interview Questions

### Beginner Level
1. **What is Git and why is it used?**
   - Git is a distributed version control system that tracks changes in source code
   - Used for collaboration, version tracking, and code management

2. **What is the difference between Git and GitHub?**
   - Git is the version control system
   - GitHub is a hosting service for Git repositories

3. **What are the main states in Git?**
   - Working directory, staging area, and Git directory

4. **What is a commit?**
   - A snapshot of changes with a unique identifier and message

### Intermediate Level
1. **What is the difference between `git pull` and `git fetch`?**
   - `git fetch` downloads changes without merging
   - `git pull` fetches and merges changes

2. **What is branching in Git?**
   - Branching allows parallel development by creating independent lines of development

3. **What is merging and what are the types of merge?**
   - Combining changes from different branches
   - Fast-forward merge and 3-way merge

4. **What is rebasing?**
   - Moving or combining sequence of commits to a new base commit

### Advanced Level
1. **What is the difference between `git reset` and `git revert`?**
   - `git reset` moves branch pointer (can rewrite history)
   - `git revert` creates new commit to undo changes (preserves history)

2. **What is cherry-picking?**
   - Applying a specific commit from one branch to another

3. **What is Git stash?**
   - Temporarily storing changes without committing

4. **What are Git hooks?**
   - Scripts that run automatically before or after events like commit, push, receive

## Best Practices

### Commit Guidelines
- **Atomic Commits**: One logical change per commit
- **Clear Messages**: Use imperative mood and be descriptive
- **Frequent Commits**: Small, regular commits
- **Review Before Push**: Always review changes before pushing

### Branch Management
- **Short-lived Branches**: Keep feature branches focused and short
- **Descriptive Names**: Use clear, descriptive branch names
- **Regular Cleanup**: Delete merged branches regularly
- **Protected Branches**: Protect main and develop branches

### Collaboration
- **Pull Requests**: Use PRs for code review
- **Code Reviews**: Review all code before merging
- **Conflict Resolution**: Resolve conflicts promptly
- **Documentation**: Document significant changes

## Troubleshooting

### Common Issues
```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Fix commit message
git commit --amend -m "New message"

# Recover deleted branch
git reflog
git checkout -b <branch-name> <commit-hash>

# Resolve merge conflicts
# Edit conflicted files
git add <resolved-files>
git commit
```

## Integration with DevOps Tools

### CI/CD Integration
- Jenkins, GitLab CI, GitHub Actions
- Automated testing on commit/push
- Automated deployment on merge

### Project Management
- Jira, Trello integration
- Branch-to-issue tracking
- Release management

### Code Quality
- SonarQube integration
- Automated code review
- Style checking

## Resources

### Official Documentation
- [Git Documentation](https://git-scm.com/doc)
- [Git Book](https://git-scm.com/book)

### Learning Resources
- [GitHub Learning Lab](https://lab.github.com/)
- [Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials)

### GUI Tools
- SourceTree
- GitKraken
- GitHub Desktop
- VS Code Git Integration