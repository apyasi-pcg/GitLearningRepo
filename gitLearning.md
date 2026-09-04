# Git Learning Notes

- Author: Infinity
- GitHub: https://github.com/apyasi-pcg
- Repo: https://github.com/apyasi-pcg/GitLearningRepo
- OS: Windows (PowerShell examples)
- Last Updated: 2026-09-04

This is a personal, professional-style reference for daily Git use on Windows. It focuses on clear steps, safe defaults, and quick copy-paste commands.

---

## Quick Start (New Repo → First Push)

```powershell
# 1) Initialize in the current folder (creates .git)
git init

# 2) See what changed (run this often)
git status

# 3) Stage your file(s)
git add gitLearning.md     # or: git add .  (stage all changes)

# 4) Commit with a clear message
git commit -m "Add gitLearning.md"

# 5) Point to your remote repository (GitHub in this example)
git remote add origin https://github.com/apyasi-pcg/GitLearningRepo.git

# 6) If your current branch should be 'main', rename it
git branch -m main

# 7) First push: set upstream so future 'git push' works without arguments
git push -u origin main
```

Notes:
- If the remote already has commits and different default branch names, fetch first and align (`git fetch --all`, then choose branch).
- Prefer `git push -u origin <branch>` the first time to set the upstream.

---

## Daily Basics

- `git status`: Shows staged/unstaged/untracked changes. Safe to run anytime; do it often.
- `git add <path>`: Stage a file or folder. `git add .` stages all changes in the current repo.
- `git commit -m "message"`: Save a snapshot. Keep messages concise and imperative (e.g., "Fix null check").
- `git push`: Push current branch to its configured upstream. Use `-u` the first time to set it.

Examples:
```powershell
git status
git add .
git commit -m "Implement feature X"
git push             # works when upstream is already set
```

---

## Branches and Renaming

- `git branch`: List local branches. Add `-a` for remotes.
- `git branch -m main`: Rename the current branch to `main`. Do this before the first push, or update remotes accordingly.

After renaming an already-pushed branch:
```powershell
git push origin -u main          # push new name and set upstream
git push origin :old-name        # optional: delete old remote branch
```

---

## Remotes

- `git remote add origin <url>`: Link your local repo to a remote named `origin`.
- `git remote -v`: Show configured remotes.
- `git remote set-url origin <url>`: Change the remote URL if needed.

Examples:
```powershell
git remote add origin https://github.com/apyasi-pcg/GitLearningRepo.git
git remote -v
```

---

## Config and Credentials (Windows)

- Identity (global):
```powershell
git config --global user.name "Infinity"
git config --global user.email "your.email@example.com"
```

- Credential helper:
```powershell
# Recommended on Windows: secure Windows Credential Manager
git config --global credential.helper manager

# Plaintext storage (not recommended):
git config --global credential.helper store

# Unset helper (this scope only)
git config --unset credential.helper

# Or unset globally
git config --global --unset credential.helper
```

Open your Git config quickly:
```powershell
# Open the global config in Notepad
notepad "$env:USERPROFILE\.gitconfig"

# If you use VS Code
code "$env:USERPROFILE\.gitconfig"

# Manual steps (File Explorer): Win + R → %USERPROFILE% → find .gitconfig
```

Notes:
- `credential.helper store` writes plaintext tokens to `%USERPROFILE%\.git-credentials`. Prefer `manager` on Windows for better security.

---

## Inspect and Undo (Safe First)

- See concise history: 
```powershell
git log --oneline --decorate --graph --all
```

- Unstage a file you accidentally added:
```powershell
git restore --staged <path>
```

- Discard local changes in a file (cannot be undone):
```powershell
git checkout -- <path>
# or (newer spelling)
git restore <path>
```

---

## De-initialize Accidental Git Repo (OneDrive root, Windows)

Goal: Stop tracking `C:\Users\Abhinav Pyasi\OneDrive - PCG` without touching nested repos.

Prereqs:
- PowerShell
- Git installed (`git --version`)

```powershell
# 1) Set root path
$root = "$env:USERPROFILE\OneDrive - PCG"

# 2) Verify current state (expect: true, then false)
git -C $root rev-parse --is-inside-work-tree
git -C $root rev-parse --is-bare-repository

# 3) Safe de-init by renaming .git to a timestamped backup (reversible)
$stamp = Get-Date -Format yyyyMMdd_HHmmss
Rename-Item -LiteralPath "$root\.git" -NewName "_git_backup_$stamp"

# 4) Confirm de-initialization (now expect an error)
git -C $root rev-parse --is-inside-work-tree

# 5) Verify nested repos are unaffected
git -C "$env:USERPROFILE\OneDrive - PCG\Documents\GitHub\CombeRepo\OracleIntegrationRepo" status -sb
git -C "$env:USERPROFILE\OneDrive - PCG\Documents\GitHub\RednersRepo\OracleIntegrationRepo" status -sb

# 6) Optional cleanup (only when you're sure backups are not needed)
Remove-Item -LiteralPath "$root\_git_backup_*" -Recurse -Force
Remove-Item -LiteralPath "$root\.gitignore","$root\.gitattributes","$root\.gitmodules" -ErrorAction SilentlyContinue

# 7) Rollback (if needed)
Rename-Item -LiteralPath "$root\_git_backup_YYYYMMDD_HHMMSS" -NewName ".git"
git -C $root status -sb
```

Notes:
- Renaming/removing the OneDrive `.git` does not modify files or inner repos.
- If `.git` is a file (linked worktree), the same rename works.
- If rename fails due to a lock, close apps indexing the folder (OneDrive/VS Code) and retry.

---

## Find All Git Repos (Windows, PowerShell)

Goal: Locate every Git repository by finding both `.git` directories and `.git` files (covers normal repos and linked worktrees).

Quick check in the current folder:
```powershell
git rev-parse --is-inside-work-tree
```

Search recursively under OneDrive:
```powershell
$root = "$env:USERPROFILE\OneDrive - PCG"

# Find .git directories
$gitDirs = Get-ChildItem -Path $root -Recurse -Force -Directory -Filter .git -ErrorAction SilentlyContinue |
  Select-Object -ExpandProperty FullName

# Find .git files (linked worktrees)
$gitFiles = Get-ChildItem -Path $root -Recurse -Force -File -Filter .git -ErrorAction SilentlyContinue |
  Select-Object -ExpandProperty FullName

$candidates = $gitDirs + $gitFiles | Sort-Object -Unique

# Show canonical repo roots
$tops = $candidates | ForEach-Object {
  try { git -C (Split-Path -Parent $_) rev-parse --show-toplevel } catch { $null }
} | Where-Object { $_ } | Sort-Object -Unique

$tops
```

---

## Handy Shortcuts (optional)

Add to your global `.gitconfig` under an `[alias]` section:

```ini
[alias]
  st = status -sb
  lg = log --oneline --decorate --graph --all
  co = checkout
  br = branch -a
  cm = commit -m
```

---

## Reference: Commands Listed Here, With Notes

- `git status`: Shows working tree status; zero risk.
- `git init`: Creates a new repo in the current folder; avoid running at a high-level folder like OneDrive root.
- `git branch`: Lists branches; add `-a` for remotes.
- `git branch -m main`: Renames current branch to `main`; do this before the first push, or update remotes and default branch after.
- `git add gitLearning.md`: Stages a specific file; use `git add .` to stage broadly; undo with `git restore --staged <path>`.
- `git commit -m "added gitLearning.md file"`: Creates a commit; keep messages concise and imperative.
- `git remote add origin <url>`: Adds the `origin` remote; verify with `git remote -v`; change with `git remote set-url origin <url>`.
- `git push`: Pushes current branch to its upstream; first-time use `git push -u origin main`.
- `git config credential.helper store`: Stores credentials in plaintext at `%USERPROFILE%\.git-credentials` (not recommended on shared machines).
- `git config --unset credential.helper`: Unsets the helper for the current scope; add `--global` to unset globally.
- `git config --global user.name` / `--global user.email`: Set your global identity; example shown above.
- `git -C <path> status -sb`: Run a Git command in another directory without leaving your shell; great for quick checks.
- `git rev-parse --is-inside-work-tree`: Returns `true` when run inside a working tree; useful for diagnostics.

---

If you want this document versioned, keep it at the repo root and update the "Last Updated" field when you change it.