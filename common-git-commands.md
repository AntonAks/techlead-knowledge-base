
# Common Git Commands

## 1. Initial Setup

### Initialize a Git Repository
```bash
git init
```
Creates a new Git repository in the current directory.

### Set Global Username and Email
```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

### Set Local Username and Email (for specific repo)
```bash
git config user.name "Your Name"
git config user.email "you@example.com"
```

## 2. Checking Configuration
```bash
git config --list            # View all current configurations
git config user.name         # Check specific config
```

## 3. Cloning a Repository
```bash
git clone <repository_url>
```
Creates a local copy of a remote repository.

## 4. Checking Status
```bash
git status
```
Shows the status of changes in the working directory.

## 5. Adding Changes
```bash
git add <file>              # Stage specific file
git add .                   # Stage all changes
```

## 6. Committing Changes
```bash
git commit -m "Your commit message"
```

## 7. Viewing Commit History
```bash
git log
git log --oneline           # Concise view
```

## 8. Branching

### Create a New Branch
```bash
git branch <branch-name>
```

### Switch to a Branch
```bash
git checkout <branch-name>
```

### Create and Switch
```bash
git checkout -b <branch-name>
```

## 9. Merging
```bash
git merge <branch-name>
```
Merges the specified branch into the current one.

## 10. Pushing to Remote
```bash
git push origin <branch-name>
```

## 11. Pulling from Remote
```bash
git pull
```

## 12. Stashing Changes
```bash
git stash                  # Save changes temporarily
git stash pop              # Reapply stashed changes
```

## 13. Viewing Remote
```bash
git remote -v
```

## 14. Add a Remote Repository
```bash
git remote add origin <url>
```

## 15. Remove a File from Git (keep locally)
```bash
git rm --cached <file>
```

## 16. Tags

### Create a Lightweight Tag
```bash
git tag <tag-name>
```
Creates a tag that just points to the current commit.

### Create an Annotated Tag
```bash
git tag -a <tag-name> -m "Tag message"
```
Creates a tag with a message, author, and date. Recommended for releases.

### List All Tags
```bash
git tag
```

### View Tag Details
```bash
git show <tag-name>
```

### Tag a Specific Commit
```bash
git tag -a <tag-name> <commit-hash> -m "Tagging a specific commit"
```

### Delete a Local Tag
```bash
git tag -d <tag-name>
```

### Push a Tag to Remote
```bash
git push origin <tag-name>
```

### Push All Tags
```bash
git push origin --tags
```

### Delete a Remote Tag
```bash
git push origin --delete <tag-name>
```