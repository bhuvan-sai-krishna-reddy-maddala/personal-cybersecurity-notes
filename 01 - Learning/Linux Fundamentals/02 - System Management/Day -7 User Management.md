## Key Files
| File | Contains | Readable by |
|------|----------|-------------|
| `/etc/passwd` | usernames, UID, shell, home dir | everyone |
| `/etc/shadow` | hashed passwords | root only |
| `/etc/group` | groups and members | everyone |

---

## Commands Overview
| Command | Does |
|---------|------|
| `sudo` | run command as another user (usually root) |
| `su` | switch entire session to another user |
| `useradd` | create new user |
| `userdel` | delete user |
| `usermod` | modify existing user |
| `addgroup` | create a group |
| `delgroup` | delete a group |
| `passwd` | change password |

---
## sudo
```bash
sudo command                # run as root
sudo -u flash command       # run as specific user
sudo -l                     # list what you can run with sudo
sudo su                     # switch to root shell
```

---
## su — Switch User
```bash
su root                     # switch to root (needs root password)
su flash                    # switch to user flash
su -                        # switch to root with full root environment
su - flash                  # switch to flash with their full environment
```

> su = switches entire session
> sudo = runs one command as another user

---
## useradd — Create User
```bash
useradd flash                           # basic create
useradd -m flash                        # create with home directory
useradd -m -s /bin/bash flash           # with home dir and bash shell
useradd -m -g developers flash          # with specific group
useradd -m -u 1337 flash                # with specific UID
```

---
## userdel — Delete User
```bash
userdel flash               # delete user, keep home directory
userdel -r flash            # delete user AND home directory and files
```

---
## usermod — Modify User
```bash
usermod -aG sudo flash          # add to sudo group
usermod -aG developers flash    # add to developers group
usermod -s /bin/bash flash      # change shell
usermod -l newname flash        # rename user
usermod -L flash                # lock account
usermod -U flash                # unlock account
```

> CRITICAL: always use -aG together
> -a = append to group
> without -a you REPLACE all groups instead of adding

---
## addgroup / delgroup
```bash
addgroup developers         # create group
delgroup developers         # delete group
```

---
## passwd — Change Password
```bash
passwd                      # change your own password
passwd flash                # change flash's password (needs sudo)
passwd -l flash             # lock account
passwd -u flash             # unlock account
```

---
## Enumeration Commands (Security)
```bash
# what can you run with sudo
sudo -l

# all users on system
cat /etc/passwd | cut -d":" -f1

# all groups
cat /etc/group

# groups a user belongs to
groups flash
id flash
```

> These are the first commands you run when you land on a box
> sudo -l especially — misconfigured sudo = instant privilege escalation

Previous:
[[Day -6 Permission Management]]

Next:
[[Day -8  Package Management]]
