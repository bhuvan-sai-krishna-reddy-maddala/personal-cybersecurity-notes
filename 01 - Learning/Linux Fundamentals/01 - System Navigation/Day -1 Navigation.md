## Theory — Why Navigation Matters
Navigation is the foundation of everything in Linux.
Before you can hack, defend, investigate, or automate — you need to move around the system confidently.
Every pentest, every CTF, every incident response starts with knowing where you are and finding what you need.

---
## Where Am I?
```bash
pwd                    # Print Working Directory — shows your current location
```

---
## Listing Contents
```bash
ls                     # basic list — files and directories only
ls -l                  # long format — detailed info
ls -a                  # show hidden files (starting with .)
ls -la                 # long format + hidden files (most common combo)
ls -l /var/            # list contents of a SPECIFIC path without navigating there
```

### Reading ls -l Output
```
drwxr-xr-x 2 cry0l1t3 htbacademy 4096 Nov 13 17:37 Desktop  
│ │ │ │ │ │ │  
│ │ │ │ │ │ └── Name  
│ │ │ │ │ └── Date/time modified  
│ │ │ │ └── Size in bytes (or blocks for dirs)  
│ │ │ └── Group owner  
│ │ └── Owner  
│ └── Number of hard links  
└── Type and permissions

```

### The `total` Line
```
total 32
```
32 blocks × 1024 bytes = 32,768 bytes (32KB) of disk space used
### Hidden Files
Files starting with `.` are hidden:
```
.bashrc ← shell configuration  
.bash_history ← command history

```
> Use ls -la to see them — ls alone won't show them

---
## Moving Around
```bash
cd /dev/shm              # navigate to absolute path directly
cd ..                     # go up one level to parent directory
cd -                      # jump back to PREVIOUS directory (toggle)
cd ~                      # go to home directory
cd                        # also goes to home directory
```

### . and ..
```
. ← current directory  
.. ← parent directory

```

Always visible in ls -la output:
```
drwxrwxrwt 2 root root 40 . ← current directory  
drwxr-xr-x 17 root root 4000 .. ← parent directory

```

---
## Tab Autocomplete
```bash
cd /dev/s[TAB TAB]        # shows all options starting with s
cd /dev/sh[TAB]           # completes to shm/ (only one match)
```
> Use TAB constantly — saves typing, prevents typos, works for paths AND commands

---
## Combining Commands
```bash
cd shm && clear           # navigate to shm AND clear terminal
```
> && = only run second command if first succeeded (you know this from filtering section)

---
## Clearing the Terminal
```bash
clear                     # clear terminal output
Ctrl + L                  # keyboard shortcut — same result
```

---
## Command History
```bash
↑ / ↓                     # scroll through previous commands
Ctrl + R                  # search command history — type to filter
history                   # show full command history list
!42                       # re-run command number 42 from history
!!                        # re-run the last command
```

---
## Quick Reference — Navigation Essentials
| Command | Does |
|---------|------|
| pwd | show current location |
| ls | list directory contents |
| ls -la | list all including hidden, with details |
| cd /path | navigate to path |
| cd .. | go up one level |
| cd - | go back to previous directory |
| cd ~ | go to home directory |
| clear / Ctrl+L | clear terminal |
| Ctrl+R | search command history |
| TAB | autocomplete path or command |

---
## Security Relevance
```bash
# .bash_history can contain sensitive commands, credentials, previous activity
cat ~/.bash_history

# hidden files often contain credentials, tokens, config
ls -la ~
ls -la /var/www/html        # check web root for hidden files

# /dev/shm — a common attacker staging area (in-memory, not written to disk)
ls -la /dev/shm
```
> During pentesting — always check hidden files and history files
> /dev/shm is a popular location for storing malware/tools temporarily
> (in-memory filesystem, cleaned on reboot, no disk forensics trace)


Next:
[[Day -2 Working with files and Directories]]