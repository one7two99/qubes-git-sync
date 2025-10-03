# Qubes Git Sync

`qubes-git-sync` is a simple command-line tool to synchronize Git repositories between **dom0**, a dedicated **SyncVM** (e.g., `my-git-sync`), and **GitHub** in a Qubes OS environment.  

Since dom0 must not connect to the internet directly, this script provides a secure workflow:  
- dom0 → SyncVM → GitHub  
- GitHub → SyncVM → dom0  

It uses `qvm-run --pass-io` to transfer repository data and delegates all GitHub interaction (SSH keys, network) to the SyncVM.  

Hints:
Please review and understand what the script does before adding it to your own qubes installation (as with every other piece of code you get from the internet :-)

Attention:
The script will assume that the changes will be done in dom0 and therefore resets SyncVM to a clean `main`. This ensures Github is always a clear mirror of dom0.

TODO v2: Fix documentation after moving from tar to git bundle to transfer data from dom0 to SyncVM

---

## Features

- **Push / Pull** repositories between dom0 and SyncVM  
- **gitpush**: dom0 → SyncVM → GitHub  
- **gitpull**: GitHub → SyncVM → dom0  
- **--all** flag: run actions for multiple repositories defined in the script  
- Logging with timestamps to `~/.qubes-git-sync.log`  
- Clear `[OK]`, `[INFO]`, and `[FAIL]` messages in both terminal and logfile  
- End-of-run summary when using `--all`  

---

## Requirements

- Qubes OS  
- A dedicated AppVM for Git sync, e.g., `my-git-sync`, with:
  - Internet access  
  - `git` and `openssh-client` installed in its template  
  - Your GitHub SSH key stored in `~/.ssh/` (inside the SyncVM, **not** in dom0)  
- The repositories cloned (or auto-cloned by the script) in `~/repos/<repo>` inside the SyncVM  
- Make sure that user.email and user.name for git commits are setup correctly in the SyncVM! This is required so that pushing commits will work. To do so run the following command in the SyncVM:  
  - `git config --global user.name 'NAME'
  - `git config --global user.email 'EMAIL'

---

## Installation

1. Copy the script into dom0:  

   ```bash
   sudo nano /usr/local/bin/qubes-git-sync
   ```

   Paste the script content.  

2. Make it executable:  

   ```bash
   sudo chmod +x /usr/local/bin/qubes-git-sync
   ```

3. Edit the variables inside the script:  
   - `SYNCVM="my-git-sync"` → your chosen sync VM  
   - `GITHUB_USER="your-github-username"`  
   - `ALL_REPOS="repo1 repo2 repo3"` → list of default repositories  

---

## Usage

```bash
qubes-git-sync {push|pull|gitpush|gitpull} <repository>|--all
```

### Commands

- **push `<repo>`**  
  Copy repository from dom0 → SyncVM  

- **pull `<repo>`**  
  Copy repository from SyncVM → dom0  

- **gitpush `<repo>`**  
  Sync dom0 → SyncVM and then run `git add/commit/push` from the SyncVM to GitHub  

- **gitpull `<repo>`**  
  Run `git pull --ff-only` in SyncVM and then copy the updated repository to dom0  

### Options

- `--all` → Perform the action for all repositories in the `ALL_REPOS` list  
- `-h` or `--help` → Show usage  

---

## Examples

Push a single repository from dom0 to SyncVM:  

```bash
qubes-git-sync push my-qubes
```

Pull from SyncVM to dom0:  

```bash
qubes-git-sync pull my-qubes
```

Push local changes from dom0 to GitHub in one step:  

```bash
qubes-git-sync gitpush my-qubes
```

Pull the latest changes from GitHub into dom0:  

```bash
qubes-git-sync gitpull my-qubes
```

Run an operation for all default repositories:  

```bash
qubes-git-sync gitpush --all
qubes-git-sync gitpull --all
```

---

## Logging

All operations are logged to `~/.qubes-git-sync.log` with timestamps.  
Sample log output:  

```
[2025-10-03 16:02:11] [INFO] Repo 'my-qubes' not found, attempting git clone ...
[2025-10-03 16:02:15] [OK] Repo 'my-qubes' successfully cloned.
[2025-10-03 16:02:18] [OK] Repo 'my-qubes' successfully pushed from dom0 to GitHub.
[2025-10-03 16:02:20] [FAIL] Git pull for 'another-repo' failed (non-fast-forward or network issue).
[2025-10-03 16:02:20] [INFO] Summary: 1 successful, 1 failed, total 2.
```

---

## Security Notes

- dom0 never connects to the internet directly.  
- SSH keys are stored only in the SyncVM.  
- dom0 communicates with SyncVM via `qvm-run --pass-io` (data streams only).  
- You can control access by limiting which repos are in `ALL_REPOS`.  

---

## License

GPLv3 License – see [LICENSE](LICENSE) file for details.  

---

👉 With this setup, you can safely manage your configuration and documentation repos in dom0 while still keeping them synchronized with GitHub.  
