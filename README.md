# qubes-git-sync

Synchronize Git repositories between **dom0**, a dedicated **SyncVM**, and **GitHub** in [Qubes OS](https://www.qubes-os.org/).  

This script enables you to keep full Git history while respecting Qubes’ strict isolation model:
- **dom0**: edit files, stage, and commit (but never connect to the network).
- **SyncVM**: acts as a proxy — it receives commit bundles from dom0 and pushes them to GitHub.
- **GitHub**: stores your full repository history.

**To stay reasonable secure:**
Please review and understand what the script does before adding it to your own qubes installation (as with every other piece of code you get from the internet :-)

---

## Why?

Qubes OS intentionally disconnects dom0 from the network for security.  
That means you cannot `git push` from dom0 directly.  
This script provides a secure workflow:

- Keep **real Git history** in dom0.  
- Use `git bundle` to export commits from dom0.  
- Transfer the bundle to SyncVM via `qvm-run --pass-io`.  
- SyncVM imports the commits and pushes to GitHub.  
- Optionally, SyncVM can pull updates from GitHub and copy them back to dom0.  

---

## Installation

1. Copy the script into dom0:

   ```bash
   sudo install -m 755 qubes-git-sync /usr/local/bin/qubes-git-sync
   ```

2. Adjust these variables in the script if needed:
   - `SYNCVM="my-git-sync"` → name of your dedicated AppVM that has network access to GitHub.
   - `GITHUB_USER="your-github-username"` → your GitHub username.
   - `ALL_REPOS="my-qubes another-repo scripts-docs"` → list of repositories managed by `--all`.

3. In your SyncVM:
   - Configure SSH access to GitHub (e.g. deploy your SSH keys).
   - Ensure `git` is installed.

4. Make sure that user.email and user.name for git commits are setup correctly in the SyncVM an dom0! This is required so that pushing commits will work. To do so run the following commands:
   - `git config --global user.name 'NAME'`
   - `git config --global user.email 'EMAIL'`

---

## Usage

### Daily Workflow

1. **In dom0**
   - Edit your files as usual.
   - Stage and commit changes:
     ```bash
     git add file1 file2
     git commit -m "Describe your changes"
     ```

2. **Sync changes to GitHub**
   ```bash
   qubes-git-sync gitpush <repository>
   ```

   This will:
   - Create a `git bundle` in dom0.
   - Transfer it securely to your SyncVM.
   - Import the bundle and push commits to GitHub.

3. **Pull updates from GitHub into dom0**
   ```bash
   qubes-git-sync gitpull <repository>
   ```
   This will:
   - `git pull` in SyncVM from GitHub.
   - Copy the updated repository tree back to dom0.

---

### Examples

Push one repository:
```bash
qubes-git-sync gitpush my-qubes
```

Push all repositories defined in `ALL_REPOS`:
```bash
qubes-git-sync gitpush --all
```

Pull latest changes from GitHub into dom0:
```bash
qubes-git-sync gitpull my-qubes
```

---

## Logging

All actions are logged in dom0 at:
```
~/.qubes-git-sync.log
```

---

## Security Notes

- dom0 never connects to the network; only SyncVM interacts with GitHub.  
- Only committed changes are synchronized. Uncommitted edits in dom0 will not be included.  
- Temporary bundle files are removed after each sync (both in dom0 and SyncVM).  

---

## Limitations

- Only the `main` branch is synchronized.  
- Merge conflicts are not automatically resolved — the script enforces fast-forward merges only.  
- If you forget to commit in dom0 before running `gitpush`, no changes will be transferred.  

---

## License

GPLv3 License – see [LICENSE](LICENSE) file for details.  

---

👉 With this setup, you can safely manage your configuration and documentation repos in dom0 while still keeping them synchronized with GitHub.  
