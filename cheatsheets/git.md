# Git Cheatsheet

> Quick-reference for everyday Git commands and "oh no" recovery.

## 🔥 Most Used

| Command | What it does | Example |
|---|---|---|
| `git status` | Show working tree status | `git status -s` (short) |
| `git add .` | Stage all changes | `git add -p` (interactive) |
| `git commit -m "msg"` | Commit staged changes | `git commit -m "fix: auth bug"` |
| `git push` | Push to remote | `git push origin main` |
| `git pull --rebase` | Pull + rebase on top | `git pull --rebase origin main` |
| `git checkout -b <branch>` | Create + switch branch | `git checkout -b feat/auth` |
| `git log --oneline -10` | Recent history (compact) | `git log --oneline --graph` |
| `git diff` | Unstaged changes | `git diff --staged` (staged) |
| `git stash` | Stash working changes | `git stash pop` |
| `git rebase -i HEAD~N` | Interactive rebase last N | `git rebase -i HEAD~3` |

## Setup

| Command | What it does | Example |
|---|---|---|
| `git config --global user.name "name"` | Set name | `git config --global user.name "Jane"` |
| `git config --global user.email "e"` | Set email | `git config --global user.email "j@co.com"` |
| `git config --global init.defaultBranch main` | Default branch name | |
| `git config --global pull.rebase true` | Always rebase on pull | |
| `git config --global core.editor "vim"` | Set editor | `git config --global core.editor "code --wait"` |
| `git init` | New repo | `git init` |
| `git clone <url>` | Clone repo | `git clone --depth 1 <url>` (shallow) |

## Daily Workflow

| Command | What it does | Example |
|---|---|---|
| `git status` | What's changed | `git status -sb` |
| `git add <file>` | Stage file | `git add src/main.rs` |
| `git add -p` | Stage interactively (hunks) | Pick y/n per change |
| `git diff` | View unstaged changes | `git diff src/` |
| `git diff --staged` | View staged changes | `git diff --staged` |
| `git commit -m "msg"` | Commit | `git commit -m "feat: add login"` |
| `git commit --amend` | Amend last commit | `git commit --amend --no-edit` |
| `git push` | Push to remote | `git push -u origin feat/auth` |
| `git pull --rebase` | Pull + rebase | `git pull --rebase origin main` |
| `git fetch` | Fetch without merge | `git fetch --all --prune` |

## Branching

| Command | What it does | Example |
|---|---|---|
| `git branch` | List local branches | `git branch -a` (include remote) |
| `git branch <name>` | Create branch | `git branch feat/auth` |
| `git checkout <branch>` | Switch branch | `git checkout main` |
| `git checkout -b <name>` | Create + switch | `git checkout -b fix/bug-123` |
| `git switch <branch>` | Switch (modern) | `git switch main` |
| `git switch -c <name>` | Create + switch (modern) | `git switch -c feat/auth` |
| `git merge <branch>` | Merge into current | `git merge feat/auth` |
| `git merge --no-ff <branch>` | Merge with merge commit | `git merge --no-ff feat/auth` |
| `git rebase <branch>` | Rebase onto branch | `git rebase main` |
| `git rebase -i HEAD~N` | Interactive rebase | `git rebase -i HEAD~5` |
| `git branch -d <name>` | Delete merged branch | `git branch -D <name>` (force) |
| `git branch -m <old> <new>` | Rename branch | `git branch -m dev develop` |
| `git cherry-pick <sha>` | Apply a specific commit | `git cherry-pick abc1234` |
| `git cherry-pick <a>..<b>` | Apply range of commits | `git cherry-pick abc1234..def5678` |

### Rebase Workflow

```bash
# Rebase feature branch onto latest main
git checkout feat/auth
git fetch origin
git rebase origin/main

# If conflicts:
# 1. Fix conflicts in files
# 2. git add <resolved-files>
# 3. git rebase --continue
# To abort: git rebase --abort

# Interactive rebase — squash, reorder, edit commits
git rebase -i HEAD~3
# In editor: pick/squash/fixup/reword/drop
```

## Stashing

| Command | What it does | Example |
|---|---|---|
| `git stash` | Stash working changes | `git stash` |
| `git stash -u` | Include untracked files | `git stash -u` |
| `git stash -m "msg"` | Stash with message | `git stash -m "wip: auth flow"` |
| `git stash list` | List all stashes | `git stash list` |
| `git stash pop` | Apply + remove latest stash | `git stash pop` |
| `git stash apply` | Apply without removing | `git stash apply stash@{2}` |
| `git stash drop` | Remove a stash | `git stash drop stash@{0}` |
| `git stash clear` | Remove all stashes | `git stash clear` |
| `git stash show -p` | Show stash diff | `git stash show -p stash@{1}` |

## Undoing Things

| Command | What it does | Example |
|---|---|---|
| `git checkout -- <file>` | Discard unstaged changes | `git checkout -- src/main.rs` |
| `git restore <file>` | Discard changes (modern) | `git restore src/main.rs` |
| `git restore --staged <file>` | Unstage file | `git restore --staged src/main.rs` |
| `git reset HEAD <file>` | Unstage (classic) | `git reset HEAD src/main.rs` |
| `git reset --soft HEAD~1` | Undo commit, keep staged | Last commit → staging area |
| `git reset --mixed HEAD~1` | Undo commit, keep unstaged | Last commit → working dir |
| `git reset --hard HEAD~1` | Undo commit, discard all | ⚠️ Destructive |
| `git revert <sha>` | Create inverse commit | `git revert abc1234` (safe for shared branches) |
| `git commit --amend` | Fix last commit message/content | `git commit --amend -m "better msg"` |
| `git clean -fd` | Remove untracked files/dirs | `git clean -fdn` (dry run first!) |

## History & Log

| Command | What it does | Example |
|---|---|---|
| `git log --oneline` | Compact log | `git log --oneline -20` |
| `git log --oneline --graph --all` | Visual branch graph | `git log --oneline --graph --all` |
| `git log -p <file>` | File change history | `git log -p src/main.rs` |
| `git log --author="name"` | Filter by author | `git log --author="Jane"` |
| `git log --since="2 weeks ago"` | Filter by date | `git log --since="2024-01-01"` |
| `git log --grep="pattern"` | Search commit messages | `git log --grep="fix"` |
| `git log -S "string"` | Search for code change (pickaxe) | `git log -S "fn authenticate"` |
| `git blame <file>` | Who changed each line | `git blame src/auth.rs` |
| `git show <sha>` | Show commit details | `git show abc1234` |
| `git shortlog -sn` | Commit count by author | `git shortlog -sn --no-merges` |
| `git bisect start` | Binary search for bad commit | See bisect workflow below |

### Bisect Workflow

```bash
git bisect start
git bisect bad                   # Current commit is broken
git bisect good abc1234          # Last known good commit
# Git checks out a middle commit — test it, then:
git bisect good                  # or: git bisect bad
# Repeat until the culprit is found
git bisect reset                 # Return to original branch
```

## Remotes

| Command | What it does | Example |
|---|---|---|
| `git remote -v` | List remotes | `git remote -v` |
| `git remote add <name> <url>` | Add remote | `git remote add upstream <url>` |
| `git remote remove <name>` | Remove remote | `git remote remove upstream` |
| `git fetch <remote>` | Fetch from remote | `git fetch upstream` |
| `git push -u origin <branch>` | Push + set tracking | `git push -u origin feat/auth` |
| `git push origin --delete <branch>` | Delete remote branch | `git push origin --delete feat/old` |
| `git pull origin <branch>` | Pull from specific remote | `git pull upstream main` |

## Tags

| Command | What it does | Example |
|---|---|---|
| `git tag` | List tags | `git tag -l "v1.*"` |
| `git tag <name>` | Lightweight tag | `git tag v1.0.0` |
| `git tag -a <name> -m "msg"` | Annotated tag | `git tag -a v1.0.0 -m "Release 1.0"` |
| `git tag -a <name> <sha>` | Tag a past commit | `git tag -a v0.9 abc1234 -m "Beta"` |
| `git push origin <tag>` | Push specific tag | `git push origin v1.0.0` |
| `git push origin --tags` | Push all tags | `git push origin --tags` |
| `git tag -d <name>` | Delete local tag | `git tag -d v1.0.0` |
| `git push origin :refs/tags/<name>` | Delete remote tag | `git push origin :refs/tags/v1.0.0` |

## 🚨 "Oh No" Recovery Commands

| Situation | Fix |
|---|---|
| Committed to wrong branch | `git reset --soft HEAD~1` → switch branch → commit |
| Need to undo a public commit | `git revert <sha>` (creates inverse commit) |
| Accidentally deleted a branch | `git reflog` → find SHA → `git branch <name> <sha>` |
| Messed up a rebase | `git reflog` → `git reset --hard <sha-before-rebase>` |
| Lost commits after `reset --hard` | `git reflog` → `git cherry-pick <sha>` |
| Need to find a lost commit | `git reflog` — shows all HEAD movements |
| Pushed secrets to remote | `git filter-branch` or `git-filter-repo` → force push → rotate secrets |
| Merge conflict hell — start over | `git merge --abort` or `git rebase --abort` |
| Want to undo everything since last push | `git reset --hard origin/<branch>` |
| Accidentally `git add`ed big file | `git reset HEAD <file>` → add to `.gitignore` |

### Reflog — Your Safety Net

```bash
# Reflog records every HEAD change (even after reset --hard)
git reflog

# Output:
# abc1234 HEAD@{0}: reset: moving to HEAD~1
# def5678 HEAD@{1}: commit: feat: add auth     ← your "lost" commit

# Recover:
git cherry-pick def5678
# or restore entire state:
git reset --hard def5678
```

---

*See [Kubernetes 101](../tutorials/rust-k8s-operator/README.md#part-3-kubernetes-101--deploy-with-kubectl) for Git workflows in the context of deploying to K8s.*
