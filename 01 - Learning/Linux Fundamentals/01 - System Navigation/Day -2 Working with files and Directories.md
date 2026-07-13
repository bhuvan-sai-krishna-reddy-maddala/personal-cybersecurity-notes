# Working with Files and Directories

## Theory — Why This Matters
File management from the CLI is faster and more powerful than GUI.
You can create, move, edit, delete, and batch-process files without ever opening a file manager.
In pentesting — creating evidence folders, moving loot, organizing findings, staging payloads.

---
## Creating Files and Directories

```bash
touch filename.txt                    # create empty file
touch ./Storage/local/user/file.txt   # create file at specific path

mkdir foldername                      # create directory
mkdir -p Storage/local/user/documents # create nested directories all at once
```

> `touch` on existing file = updates its timestamp, doesn't overwrite it
> `-p` = parents — creates every directory in the path that doesn't exist yet

---
## Viewing Structure
```bash
tree .                    # visual tree of current directory
tree /path/to/folder      # tree of specific path
```

---
## Moving and Renaming
```bash
mv oldname.txt newname.txt            # rename a file
mv file.txt Storage/                  # move file into directory
mv file1.txt file2.txt Storage/       # move MULTIPLE files at once
mv Storage/ NewName/                  # rename a directory
```
> mv does both renaming AND moving — same command, different arguments

---
## Copying
```bash
cp file.txt Storage/local/            # copy file to directory
cp -r Storage/ Backup/                # copy directory recursively (-r required for dirs)
```

---
## Deleting — The Exercise They Left Out
```bash
rm file.txt                           # delete a file
rm -r foldername/                     # delete directory and everything inside
rm -rf foldername/                    # force delete — no confirmation (CAREFUL)
rmdir foldername/                     # delete EMPTY directory only
```
> rm -rf is dangerous — no recycle bin, no undo, permanent
> Always double-check what you're deleting before running rm -rf

---
## Path Shortcuts
```
.     ← current directory
..    ← parent directory
~     ← home directory
/     ← root of filesystem
```

```bash
touch ./Storage/file.txt        # ./ = start from current directory
cp file.txt ../                  # copy to parent directory
```

---
## Quick Reference
| Command | Does |
|---------|------|
| touch file | create empty file |
| mkdir folder | create directory |
| mkdir -p a/b/c | create nested directories |
| mv old new | rename or move |
| cp src dest | copy file |
| cp -r src dest | copy directory |
| rm file | delete file |
| rm -r folder | delete directory |
| rmdir folder | delete empty directory |
| tree . | show visual directory structure |

---
## Security Relevance
```bash
# create organized pentest evidence folder structure
mkdir -p engagement/recon engagement/scans engagement/exploitation engagement/loot

# move captured hashes to loot folder
mv hashes.txt engagement/loot/

# check what's in a directory before deleting
tree /tmp/suspicious_folder

# stage payloads
touch /tmp/.hidden_payload         # . at start = hidden file
```
> /tmp and /dev/shm are common attacker staging areas
> Hidden files (starting with .) are used to hide tools and payloads


# Editing Files

## Theory — Why Editors Matter
Sometimes you need to edit files interactively — config files, scripts, notes.
Two main editors you'll use constantly: Nano (beginner friendly) and Vim (powerful, industry standard).
Knowing at least basic Vim is essential — it's on every Unix system, nano sometimes isn't.

---
## Nano — The Beginner-Friendly Editor

```bash
nano filename.txt        # open/create file in nano
```
### Nano Keyboard Shortcuts
> `^` means Ctrl

| Shortcut     | Action                     |
| ------------ | -------------------------- |
| Ctrl+O       | save (Write Out)           |
| Ctrl+X       | exit                       |
| Ctrl+W       | search forward             |
| Ctrl+W again | next match                 |
| Ctrl+K       | cut line                   |
| Ctrl+U       | paste (uncut)              |
| Ctrl+G       | help                       |
| Ctrl+C       | show cursor position       |
| Ctrl+_       | go to specific line number |

### Basic Nano Workflow
```
1. nano filename.txt ← open file
2. type your content
3. Ctrl+O → Enter ← save
4. Ctrl+X ← exit

```

---
## Vim — The Powerful Editor

```bash
vim filename.txt           # open/create file in vim
vimtutor                   # built-in interactive tutorial (25-30 mins, highly recommended)
```

### The Core Concept — Vim is MODAL
Unlike every other editor — Vim has MODES.
What you type depends entirely on which mode you're in.

### Six Vim Modes
| Mode | What happens when you type |
|------|---------------------------|
| Normal | commands (not text) — DEFAULT mode on open |
| Insert | text gets typed into file |
| Visual | select/highlight text |
| Command | single-line commands at bottom (:) |
| Replace | overwrites existing characters |
| Ex | multiple sequential commands |

### Getting Into and Out of Modes
```
Normal → Insert: press i (insert before cursor) or a (after cursor) or o (new line below)  
Insert → Normal: press Esc  
Normal → Command: press :  
Normal → Visual: press v

```

### Essential Vim Commands (Normal Mode)
```

h j k l ← left, down, up, right (arrow keys also work)  
w ← jump forward one word  
b ← jump backward one word  
0 ← start of line  
$ ← end of line  
gg ← top of file  
G ← bottom of file  
dd ← delete (cut) current line  
yy ← yank (copy) current line  
p ← paste below  
u ← undo  
Ctrl+R ← redo  
/pattern ← search forward  
n ← next search result  
N ← previous search result

```

### Essential Vim Commands (Command Mode — press : first)
```
:w ← save  
:q ← quit  
:wq ← save and quit  
:q! ← quit WITHOUT saving (force)  
:wq! ← force save and quit  
:set nu ← show line numbers  
:42 ← jump to line 42  
:%s/old/new/g ← find and replace ALL occurrences

```

### Basic Vim Workflow
```
1. vim filename.txt ← open file
2. press i ← enter insert mode
3. type your content
4. press Esc ← back to normal mode
5. type :wq ← save and quit

```

### Why Learn Vim
```
Available on EVERY Unix/Linux system — no installation needed  
Nano isn't always available (especially minimal server installs)  
Faster for editing config files once you know the shortcuts  
Used by most experienced Linux admins and pentesters

```

---
## Important Files for Pentesters

```bash
cat /etc/passwd             # usernames, UIDs, GIDs, home directories, shells
cat /etc/shadow             # hashed passwords (root only)
```

### /etc/passwd Format
```
root:x:0:0:root:/root:/bin/bash  
│ │ │ │ │ │ │  
│ │ │ │ │ │ └── Default shell  
│ │ │ │ │ └── Home directory  
│ │ │ │ └── Comment/GECOS field  
│ │ │ └── GID  
│ │ └── UID  
│ └── x = password stored in /etc/shadow  
└── Username

```

> If /etc/passwd is world-writable = critical vulnerability
> Historically contained password hashes — now in /etc/shadow
> Misconfigured permissions on either file = privilege escalation opportunity

---

## Quick Reference — Editor Comparison
| Feature | Nano | Vim |
|---------|------|-----|
| Learning curve | easy | steep but worth it |
| Always installed | sometimes | almost always |
| Speed once learned | moderate | very fast |
| Best for | quick edits, beginners | everything |

---

## Security Relevance
```bash
# edit SSH config to harden it
sudo vim /etc/ssh/sshd_config

# edit crontab directly
vim /etc/crontab

# check for sensitive data in files
cat /etc/passwd
cat /etc/shadow         # need root

# quick note taking during pentest
nano /tmp/findings.txt

# edit a script in place
vim /opt/script.sh
```

Previous:
[[Day -1 Navigation]]

Next:
[[Day -3 File Descriptors and Redirections]]
