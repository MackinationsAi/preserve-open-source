# Complete GitHub Repository Archiving Guide

A guide for creating a full offline backup of a GitHub repository w/ all branches, tags, & LFS files.

## Example Repository
This guide uses [InvokeAI](https://github.com/invoke-ai/InvokeAI) as an example, but works for any GitHub repository.

## Prerequisites
- Git installed w/ Git LFS support
- PowerShell (Windows) or adapt commands for bash (Linux/Mac)
- Sufficient disk space (check repo size on GitHub)

## Step 1: Initial Clone & Branch Checkout

Clone the repository & check out all branches:
```powershell
cd X:/     # Replace X: w/ your target drive/path
git clone https://github.com/invoke-ai/InvokeAI.git; cd InvokeAI; git branch -r | ForEach-Object { $_.Trim() } | Where-Object { $_ -notmatch '->' } | ForEach-Object { $branch = $_ -replace 'origin/', ''; git checkout $branch }
```

**What this does:**
- Clones the repository
- Navigates into the repo directory
- Creates local tracking branches for all remote branches

**Note:** You may encounter "Aborting" messages during this step due to Git LFS or lock file issues. This is normal. If this happens, close PowerShell, reopen it, & proceed to Step 2 to resolve these issues & complete the process.

## Step 2: Fix Any Issues & Complete LFS Download

If you encounter any errors (like Git lock files or LFS issues), run this cleanup command:
```powershell
cd X:\InvokeAI; Remove-Item -Force .git/index.lock -ErrorAction SilentlyContinue; git checkout main -f; Remove-Item -Force tests/test_model_probe/stripped_models/peft_adapter_model.safetensors -ErrorAction SilentlyContinue; git branch -r | ForEach-Object { $_.Trim() } | Where-Object { $_ -notmatch '->' } | ForEach-Object { $branch = $_ -replace 'origin/', ''; if (-not (git branch --list $branch)) { git checkout $branch -f } }; git checkout main -f; git lfs fetch --all; git lfs checkout; git fetch --tags
```

**What this does:**
- Removes any Git lock files
- Forces checkout to main branch
- Removes problematic LFS files (if any)
- Checks out any remaining branches
- Fetches all LFS objects
- Fetches all tags/releases
- Returns to main branch

## Step 3: Update Your Archive (Optional - Run Periodically)

To keep your archive up-to-date w/ the latest changes:
```powershell
cd X:\InvokeAI; git fetch --all --tags; git checkout main; git pull; git branch -r | ForEach-Object { $_.Trim() } | Where-Object { $_ -notmatch '->' } | ForEach-Object { $branch = $_ -replace 'origin/', ''; if (-not (git branch --list $branch)) { git checkout $branch } else { git checkout $branch; git pull } }; git checkout main; git branch > branches-list.txt; git tag > tags-list.txt; git remote -v > remote-info.txt
```
**What this does:**
- Fetches all new remote branches & tags
- Updates main branch
- Updates all existing local branches
- Creates local branches for any new remote branches
- Returns to main branch
- Creates/updates 3 .txt files w/ the names of all branches, tags & remote URLs

## Verification

Check how many branches you have:
```powershell
git branch | Measure-Object -Line
```

View all branches:
```powershell
git branch -a
```

Check disk usage:
```powershell
Get-ChildItem .git -Recurse | Measure-Object -Property Length -Sum
```

## What You Get

- [x] Complete offline copy of the repository
- [x] All branches (development, feature, release, etc.)
- [x] All Git history & commits
- [x] All tags & releases
- [x] All LFS files (large binary files)
- [x] No internet required to browse, switch branches, or view history 

## Use Cases

- **Preservation**: Archive open-source projects before acquisitions or shutdowns
- **Offline Development**: Work without internet access
- **Backup**: Personal backup of important repositories
- **Research**: Study complete development history

## Important Notes

- Replace `X:/` w/ your actual desired drive/path
- For InvokeAI specifically, some file paths may need adjustment based on your system
- The repository will be fully independent from GitHub after cloning
- You can copy the entire folder to other locations or drives
- Consider making multiple backup copies on different drives

## Adapting for Other Repositories

Replace the repository URL in Step 1:
```powershell
git clone https://github.com/username/repository.git
```

Adjust file paths in Step 2 if you encounter different LFS or lock file issues specific to that repository.

---

**Storage Tip**: External drives are ideal for archiving. The complete repository w/ all branches takes up the same space as a single clone plus the delta differences between branches.
