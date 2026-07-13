## Backup Tools Overview
| Tool | What It Does |
|------|--------------|
| Rsync | fast, efficient sync — only transfers changed parts |
| Duplicity | rsync + encryption for secure remote backups |
| Deja Dup | GUI wrapper around rsync, beginner friendly |

## Additional Encryption Tools
| Tool | Purpose |
|------|---------|
| GnuPG | encrypt backup files |
| eCryptfs | filesystem-level encryption |
| LUKS | full disk encryption |

---

## rsync — Install
```bash
sudo apt install rsync -y
```

## rsync — Common Flags
| Flag | Meaning |
|------|---------|
| -a | archive mode — preserves permissions, timestamps, ownership |
| -v | verbose — show progress |
| -z | compress data during transfer |
| -e ssh | use SSH as transfer method (encrypted) |
| --delete | remove files on destination no longer on source |
| --backup | keep old versions instead of overwriting |
| --backup-dir | where to store old versions |

## CRITICAL — Trailing Slash Matters
```bash
rsync -av ~/to_backup ~/synced_backup       # copies FOLDER itself into destination
rsync -av ~/to_backup/ ~/synced_backup      # copies CONTENTS of folder into destination
```
Without slash → synced_backup/to_backup/files...
With slash    → synced_backup/files...
> Classic gotcha — always be deliberate about this

---

## Basic Backup
```bash
rsync -av /path/to/mydirectory user@backup_server:/path/to/backup/directory
```

## Advanced Backup (compressed, incremental, cleaned)
```bash
rsync -avz --backup --backup-dir=/path/to/backup/folder --delete /path/to/mydirectory user@backup_server:/path/to/backup/directory
```

## Restore (reverse direction)
```bash
rsync -av user@remote_host:/path/to/backup/directory /path/to/mydirectory
```

## Encrypted Transfer
```bash
rsync -avz -e ssh /path/to/mydirectory user@backup_server:/path/to/backup/directory
```
> -e ssh tunnels transfer through SSH — encrypted instead of plaintext

---

## SSH Key-Based Authentication (for automation)

### Why
Cron jobs can't respond to password prompts — need passwordless auth for automated rsync.

### Step 1 — Generate Key Pair
```bash
ssh-keygen -t rsa -b 2048
```

| Flag | Meaning |
|------|---------|
| -t rsa | key type |
| -b 2048 | key size in bits |

Creates:
~/.ssh/id_rsa ← PRIVATE key — never share  
~/.ssh/id_rsa.pub ← PUBLIC key — safe to share

> Public key = lock you give the server
> Private key = key only you hold

### Step 2 — Copy Public Key to Server
```bash
ssh-copy-id user@backup_server
```
Appends your public key to server's ~/.ssh/authorized_keys
Now connects WITHOUT password:
```bash
ssh user@backup_server
```

### Security — Key Permissions
```bash
chmod 600 ~/.ssh/id_rsa        # private key readable ONLY by you
```
> If permissions are too loose, SSH refuses to use the key

---

## Automated Backup Script

### The Script
```bash
#!/bin/bash
rsync -avz -e ssh /path/to/mydirectory user@backup_server:/path/to/backup/directory
```

### Make Executable
```bash
chmod +x RSYNC_Backup.sh
```

### Schedule via Cron
```bash
crontab -e
```
```bash
0 * * * * /path/to/RSYNC_Backup.sh    # every hour, minute 0
```

---

## Practice Exercise (Local Test)
```bash
mkdir ~/to_backup
mkdir ~/synced_backup

# manual test
rsync -av ~/to_backup/ ~/synced_backup/

# automate every minute via cron
crontab -e
```
```bash
* * * * * rsync -av ~/to_backup/ ~/synced_backup/
```
> Local test doesn't need SSH — direct paths work fine

---

## Security Angle
This exact workflow (ssh-keygen + ssh-copy-id + rsync + cron) shows up in:
- Setting up persistent access during authorized pentests
- Automating data exfiltration in red team engagements
- Secure CI/CD pipeline setups

Previous:
[[Day -12 Working with Web Services]]

Next:
[[Day -14 File System Management]]