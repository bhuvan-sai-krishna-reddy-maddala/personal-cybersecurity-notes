## File System Types
| File System | Best For |
|-------------|----------|
| ext2 | old, no journaling — USB drives, low overhead |
| ext3 | journaling added — crash recovery |
| ext4 | default for most modern Linux — balance of performance/reliability |
| Btrfs | snapshots, data integrity checks — complex storage setups |
| XFS | large files, high I/O performance |
| NTFS | Windows compatibility — dual-boot, external drives |

### Journaling
Before writing changes, filesystem logs the intended change first.
If system crashes mid-write — replay journal to recover instead of corrupting data.
```
ext2 = no journal = risky on crash  
ext3/ext4 = journal = recoverable on crash
```

---

## Inodes — Core Concept
Inode stores METADATA about a file — NOT the content or even the name.
```
Inode contains:

- permissions
- ownership
- size
- timestamps
- pointers to where actual DATA blocks are on disk
  
````
> Filename and inode are SEPARATE — directory just maps a name to an inode number

```bash
ls -il
```

```

10678872 -rw-r--r-- 1 cry0l1t3 htb 234123 Feb 14 19:30 myscript.py

```
First number = inode number

### Why It Matters
Can run out of INODES before running out of DISK SPACE — happens with millions of tiny files.
```bash
df -i              # check inode usage
df -h              # check disk space usage (df = disk free)
```
> df -i and df -h can show very different percentages

---

## File Types in Linux
| Type | Description |
|------|-------------|
| Regular file | text, binaries, images, executables |
| Directory | container for other files/directories |
| Symbolic link | shortcut/reference pointing to another file |

### Symbolic Links
```bash
ln -s /path/to/original /path/to/link
```

```bash
ls -l
```

```
lrwxrwxrwx 1 user user 20 Jan 1 10:00 shortcut -> /original/file/path
```


> l at start of permissions = symlink
> -> shows where it points

---

## Disk Management — fdisk
```bash
sudo fdisk -l           # list all disks and partitions
```

Disk /dev/vda: 160 GiB ← physical disk  
/dev/vda1 * ... 75.8G 83 Linux ← partition 1 (bootable, marked *)  
/dev/vda2 ... 4.2G 82 Linux swap ← partition 2 (swap space)

Device = partition name  
Boot = * means bootable  
Type = filesystem type code (83 = Linux, 82 = swap)

```bash
lsblk           # cleaner alternative view of disks/partitions
```

---

## Mounting
Drive/partition isn't usable until MOUNTED to a directory (mount point).
```bash
mount                                    # list currently mounted filesystems
sudo mount /dev/sdb1 /mnt/usb            # mount drive to directory
sudo umount /mnt/usb                     # unmount it
```
> Mounting = "plugging in" a drive to a specific folder

### USB Mounting — Full Walkthrough
```bash
# 1. Plug in USB (not auto-mounted on CLI systems)
sudo fdisk -l                  # or lsblk — find new device, e.g. /dev/sdb1

# 2. Create mount point
sudo mkdir /mnt/usb

# 3. Mount it
sudo mount /dev/sdb1 /mnt/usb

# 4. Use normally
ls /mnt/usb
cp /mnt/usb/file.txt ~/Desktop/

# 5. Unmount before removing physically
sudo umount /mnt/usb
```
> ALWAYS unmount before physically removing — risk of data corruption otherwise

---

## /etc/fstab — Auto-Mount on Boot
```bash
cat /etc/fstab
```

```
<file system> <mount point> <type> <options> <dump> <pass>  
/dev/sda1 / ext4 defaults 0 1  
/dev/sdb1 /mnt/usb ext4 rw,noauto,user 0 0

```

### Column Breakdown
| Column | Meaning |
|--------|---------|
| file system | device/partition (or UUID) |
| mount point | where it gets mounted |
| type | filesystem type |
| options | mount behavior settings |
| dump | legacy backup flag (old `dump` utility) — almost always 0 now |
| pass | fsck check ORDER at boot — 1=root(checked first), 2=after root, 0=never checked |

### Options Column — Common Values
| Option | Meaning |
|--------|---------|
| defaults | shortcut for: rw, suid, dev, exec, auto, nouser, async |
| rw | mount read-write |
| ro | mount read-only |
| auto | mount automatically at boot |
| noauto | don't mount automatically — manual only |
| user | allow normal users to mount it |
| nouser | only root can mount it |
| exec | allow executing binaries from this filesystem |
| noexec | block executing binaries — security hardening |
| suid | allow SUID/SGID bits to work |
| nosuid | block SUID/SGID — security hardening |
| nodev | block device files — security hardening |
| relatime | update access time less frequently — performance |

### Security Hardening Example (e.g. untrusted USB)
```

/dev/sdb1 /mnt/usb ext4 ro,noexec,nosuid,nodev 0 0

````
Even if USB contains malware — can't write, can't execute, no SUID exploits, no device files.
> Classic protection when inspecting a found/untrusted USB drive

---

## lsof — List Open Files
Check before unmounting if any process is using files on that filesystem.
```bash
lsof | grep username           # files opened by specific user
lsof /mnt/usb                  # processes using files on this mount
fuser -k /mnt/usb               # force kill processes using this mount point
```

---

## SWAP — Virtual Memory Extension
When RAM fills up, kernel moves inactive memory pages to SWAP on disk — frees RAM for active processes.

```bash
mkswap /dev/sdb2          # prepare partition/file as swap
swapon /dev/sdb2           # activate it
swapoff /dev/sdb2          # deactivate it
swapon -s                  # show current swap usage
free -h                    # RAM and swap usage together
```

### Why Encrypt Swap
Sensitive data (passwords, keys) in RAM can get written to swap.
Without encryption — that data sits in PLAINTEXT on disk.

### Encrypting Swap — LUKS / dm-crypt with Random Key Method
```bash
# 1. Disable current swap
sudo swapoff -a

# 2. Identify swap partition
sudo fdisk -l | grep -i swap

# 3. Edit /etc/crypttab
sudo vim /etc/crypttab
```
````

cryptswap /dev/vda2 /dev/urandom swap,cipher=aes-xts-plain64,size=256

cryptswap ← name for encrypted mapping  
/dev/vda2 ← actual swap partition  
/dev/urandom ← random key generated EVERY boot  
swap,cipher=...,size=256 ← encryption settings

````
> /dev/urandom = new random key every boot, no password needed
> Swap data is temporary anyway — losing key on reboot doesn't matter

```bash
# 4. Update /etc/fstab — point to encrypted mapper
# old: /dev/vda2   swap   swap   defaults   0   0
# new:
/dev/mapper/cryptswap   swap   swap   defaults   0   0
```
```bash
# 5. Apply
sudo systemctl daemon-reload
sudo swapon -a

# 6. Verify
swapon -s
lsblk           # should see cryptswap mapped over swap partition
```

### Swap for Hibernation
Hibernation saves system state (open apps, processes) to swap, powers off.
On boot — restores from swap, resumes exactly where left off.

---

## Security Angle
```bash
# check mounted filesystems for anything unusual
mount | grep -v "^proc\|^sys\|^cgroup"

# check fstab for unusual remote mounts
cat /etc/fstab

# inode exhaustion DoS check
df -i

# check for hidden/unusual swap
swapon -s
```
> Classic attack: fill disk with millions of tiny files to exhaust inodes —
> denial of service even with plenty of disk SPACE remaining

---

## Quick Command Reference
| Command | Purpose |
|---------|---------|
| fdisk -l | list disks and partitions |
| lsblk | cleaner view of disks/partitions |
| mount | list mounted filesystems / mount a drive |
| umount | unmount a drive |
| lsof | list open files (check before unmount) |
| fuser -k | force kill processes using a mount point |
| df -h | disk space usage |
| df -i | inode usage |
| mkswap | prepare swap space |
| swapon / swapoff | activate/deactivate swap |
| free -h | RAM and swap overview |

Previous:
[[Day -13 Backup and Restore]]

Next:
[[Day -15 Containerization]]