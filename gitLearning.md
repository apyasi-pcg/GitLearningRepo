List of GIT Commands

    git config --global user.name
    git config --global user.email

How to open the .gitconfig file in windows:
    Press Windows Key + R to open the Run dialog.
    Type %USERPROFILE% and hit Enter
    Look for a file named .gitconfig

## De-initialize Accidental Git Repo (OneDrive root, Windows)

Goal: Stop tracking C:\Users\Abhinav Pyasi\OneDrive - PCG without touching nested repos.

Prereqs:
- PowerShell
- Git installed (`git --version`)

1) Set root path:
    $root = "$env:USERPROFILE\OneDrive - PCG"

2) Verify current state:
    git -C $root rev-parse --is-inside-work-tree
    git -C $root rev-parse --is-bare-repository
    # Expect: true then false

3) Safe de-init by renaming `.git` to a timestamped backup (reversible):
    $stamp = Get-Date -Format yyyyMMdd_HHmmss
    Rename-Item -LiteralPath "$root\.git" -NewName "_git_backup_$stamp"

4) Confirm de-initialization (expect an error now):
    git -C $root rev-parse --is-inside-work-tree

5) Verify nested repos are unaffected:
    git -C "$env:USERPROFILE\OneDrive - PCG\Documents\GitHub\CombeRepo\OracleIntegrationRepo" status -sb
    git -C "$env:USERPROFILE\OneDrive - PCG\Documents\GitHub\RednersRepo\OracleIntegrationRepo" status -sb

6) Optional cleanup (after you're certain you don't need the backup/history):
    # Remove backup(s)
    Remove-Item -LiteralPath "$root\_git_backup_*" -Recurse -Force
    # Remove stray Git metadata files at OneDrive root if unneeded
    Remove-Item -LiteralPath "$root\.gitignore","$root\.gitattributes","$root\.gitmodules" -ErrorAction SilentlyContinue

7) Rollback (if needed):
    Rename-Item -LiteralPath "$root\_git_backup_YYYYMMDD_HHMMSS" -NewName ".git"
    git -C $root status -sb

Notes:
- Renaming/removing the OneDrive `.git` does not modify files or the inner repos.
- If `.git` is a file (linked worktree), the same rename works.
- If rename fails due to a lock, close apps indexing the folder (e.g., OneDrive/VS Code) and retry.

## Find All Git Repos (Windows, PowerShell)

Goal: Locate every Git repository by finding both `.git` directories and `.git` files (covers normal repos and linked worktrees).

Quick check (current folder):
```powershell
git rev-parse --is-inside-work-tree