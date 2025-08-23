# Git Command Reference

## Basic Status and Changes
- `git status` - Shows what files have changed and what's staged
- `git diff` - Shows the actual changes in your files
- `git log` - Shows the commit history

## Making Changes
- `git add .` - Stages all changes
- `git add <filename>` - Stages a specific file
- `git commit -m "message"` - Commits staged changes
- `git commit -am "message"` - Adds and commits all tracked files in one step

## Branching
- `git branch` - Lists all branches
- `git branch <name>` - Creates a new branch
- `git checkout <branch>` - Switches to a branch
- `git checkout -b <name>` - Creates and switches to a new branch

## Remote Operations
- `git pull` - Fetches and merges changes from remote
- `git fetch` - Downloads changes but doesn't merge
- `git push` - Pushes your changes to remote
- `git remote -v` - Shows your remote repositories

## Undoing Things
- `git reset --hard HEAD` - Discards all uncommitted changes
- `git reset --soft HEAD~1` - Undoes last commit but keeps changes
- `git checkout -- <file>` - Discards changes to a specific file

## Useful Aliases
Add these to your `~/.gitconfig`:
```gitconfig
[alias]
    st = status
    co = checkout
    br = branch
    ci = commit
    unstage = reset HEAD --
    last = log -1 HEAD
```

## Common Workflows

### Starting a New Feature
```bash
git checkout -b feature/new-feature
# Make changes
git add .
git commit -m "Add new feature"
git push origin feature/new-feature
```

### Updating Your Local Repository
```bash
git pull
# or
git fetch
git merge
```

### Fixing Mistakes
```bash
# Discard changes to a file
git checkout -- <file>

# Undo last commit but keep changes
git reset --soft HEAD~1

# Discard all uncommitted changes
git reset --hard HEAD
``` 