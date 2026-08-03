# Permission Management

## Permission Basics
Every file and directory has:
- An owner (user)
- A group
- Permissions for owner, group, and others

## Three Permission Types
| Symbol | Name | On a File | On a Directory |
|--------|------|-----------|----------------|
| `r` | read | view contents | list files inside |
| `w` | write | modify file | create/delete/rename inside |
| `x` | execute | run as program | enter the directory |

> Execute on a directory = the key to walk in
> Without it — Permission Denied even if you can see inside

---

## Reading ls -l Output

- rwx rw- r-- 1 root root 1641 May 4 23:42 /etc/passwd  
  | | | | | | |  
  | | | | | | └── group  
  | | | | | └── owner  
  | | | | └── hard links  
  | | | └── others permissions  
  | | └── group permissions  
  | └── owner permissions  
  └── file type (- file, d directory, l symlink)


---

## Octal Permission System
Each permission has a value:

```
r = 4  
w = 2  
x = 1

- = 0
```

Add them up for each group:
```
rwx = 4+2+1 = 7  
rw- = 4+2+0 = 6  
r-x = 4+0+1 = 5  
r-- = 4+0+0 = 4  
--- = 0+0+0 = 0
```

### Binary Breakdown
```
rwx = 1 1 1 = 7  
rw- = 1 1 0 = 6  
r-x = 1 0 1 = 5  
r-- = 1 0 0 = 4  
--- = 0 0 0 = 0
````

### Common Permission Values
| Octal | Symbolic | Meaning |
|-------|----------|---------|
| 777 | rwxrwxrwx | everyone full access — dangerous |
| 755 | rwxr-xr-x | standard executables |
| 644 | rw-r--r-- | standard files |
| 600 | rw------- | private files |
| 400 | r-------- | read only restricted |

---

## chmod — Change Permissions

### Octal Way
```bash
chmod 755 file
chmod 644 file
chmod 777 file      # dangerous
```

### Symbolic Way
| Reference | Means |
|-----------|-------|
| `u` | owner |
| `g` | group |
| `o` | others |
| `a` | all three |

| Operator | Means |
|----------|-------|
| `+` | add permission |
| `-` | remove permission |
| `=` | set exactly |

```bash
chmod u+x file              # add execute for owner
chmod g-w file              # remove write from group
chmod o+r file              # add read for others
chmod a+r file              # add read for everyone
chmod u+x,g-w file          # multiple changes at once
chmod u=rwx,g=rw,o=r file   # set all three exactly
```

### Octal vs Symbolic
| Situation | Use |
|-----------|-----|
| Setting all permissions at once | Octal |
| Adding/removing one permission | Symbolic |
| Scripting | Octal |
| Quick adjustments | Symbolic |

---

## chown — Change Owner
```bash
chown user file                 # change owner only
chown user:group file           # change owner and group
chown :group file               # change group only
chown -R user:group directory   # recursive — everything inside too
```

> chown only takes owner and group — no third slot
> Others is not an entity you assign — it's everyone else by default
> To control others — use chmod

## chgrp — Change Group Only
```bash
chgrp developers file           # same as chown :group file
```

---

## Special Permissions

### SUID — Set User ID
File runs with OWNER's permissions instead of the user running it.
```bash
-rwsr-xr-x 1 root root /usr/bin/passwd
```
`s` in owner execute position = SUID set.

### SGID — Set Group ID
File runs with GROUP's permissions instead of the user running it.
```bash
-rwxr-sr-x root shadow /usr/bin/write
```
`s` in group execute position = SGID set.
On directories — files created inside inherit the directory's group.

### Sticky Bit
Set on directories — users can only delete their OWN files.
```bash
drwxrwxrwt /tmp
```
`t` at end = sticky set with execute.
`T` at end = sticky set WITHOUT execute.

---

## s/S/t/T — Case Meaning
| Symbol | Where | Meaning |
|--------|-------|---------|
| `s` | owner execute | SUID set + execute ON |
| `S` | owner execute | SUID set + execute OFF |
| `s` | group execute | SGID set + execute ON |
| `S` | group execute | SGID set + execute OFF |
| `t` | others execute | Sticky set + execute ON |
| `T` | others execute | Sticky set + execute OFF |

> lowercase = special bit ON + execute ON
> UPPERCASE = special bit ON + execute OFF
> Uppercase is unusual — worth investigating in security

---
## Special Permission Octal Values
```bash
chmod 4755 file     # SUID + 755
chmod 2755 file     # SGID + 755
chmod 1755 file     # Sticky + 755
chmod 6755 file     # SUID + SGID (4+2=6) + 755
chmod 7755 file     # SUID + SGID + Sticky (4+2+1=7) + 755
```

Leading digit:
````

4 = SUID  
2 = SGID  
1 = Sticky Bit

````

---

## Finding Special Permission Files (Security)
```bash
# find all SUID files
find / -perm -4000 2>/dev/null

# find all SGID files
find / -perm -2000 2>/dev/null

# find both
find / -perm -4000 -o -perm -2000 2>/dev/null
```

> `-` before number = has this bit set (don't care about rest)
> Without `-` = exact match only
> `-o` = OR condition in find

### Common Exploitable SUID Binaries
```bash
/usr/bin/find  
/usr/bin/vim  
/usr/bin/python  
/usr/bin/less  
/usr/bin/awk  
/bin/bash
```
Check each on GTFOBins — https://gtfobins.github.io

### Privilege Escalation Workflow
- step 1 — find SUID binaries
	find / -perm -4000 2>/dev/null
- step 2 — check each on GTFOBins
- step 3 — exploit → root

Previous:
[[Day -5 RegEX]]

Next:
[[Day -7 User Management]]

Index:
[[00 - Index Linux Fundamentals]]
