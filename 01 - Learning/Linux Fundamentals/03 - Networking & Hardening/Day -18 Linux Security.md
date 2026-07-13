## First Line of Defense — Patching
```bash
sudo apt update && sudo apt dist-upgrade
```
> dist-upgrade handles dependency changes — can install/remove packages as needed
> Out-of-date systems = #1 reason privilege escalation works in CTFs and real engagements

---
## SSH Hardening
```bash
sudo vim /etc/ssh/sshd_config
```

```
PermitRootLogin no  
PasswordAuthentication no

```

| Setting | Why |
|---------|-----|
| PermitRootLogin no | even if root password leaks, can't SSH directly as root |
| PasswordAuthentication no | forces key-based auth — much harder to brute force |

```bash
sudo systemctl restart sshd
```

---
## Principle of Least Privilege
Give users ONLY what they need, nothing more.

```bash
# BAD — full sudo access
usermod -aG sudo flash

# GOOD — specific command only via sudoers
sudo visudo
```

```
flash ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart apache2
```
> Only lets flash restart Apache as root — nothing else

---
## fail2ban — Brute Force Protection
```bash
sudo apt install fail2ban -y
sudo systemctl status fail2ban
sudo fail2ban-client status sshd          # check banned IPs for SSH
```
> Counts failed login attempts, bans IP after X failures via iptables
> Why brute-forcing SSH often fails in real pentests — fail2ban locks you out quickly

---
## Auditing for Privilege Escalation Risks
```bash
uname -r                              # kernel version — outdated?
find / -perm -4000 2>/dev/null        # SUID binaries
find / -writable -type d 2>/dev/null  # world-writable directories
crontab -l                            # misconfigured cron jobs
sudo -l                               # current user's sudo permissions
```

---
## Security Tools
| Tool | Purpose |
|------|---------|
| Snort | network intrusion detection system (IDS) |
| chkrootkit | scans for known rootkits |
| rkhunter | rootkit hunter — different detection methods |
| Lynis | comprehensive security auditing tool |

```bash
sudo apt install rkhunter -y
sudo rkhunter --check

sudo apt install lynis -y
sudo lynis audit system          # full system security report
```
> lynis audit system is genuinely useful — flags hardening suggestions for any Linux box

---

## Hardening Checklist
```
✓ Remove unnecessary services/software  
✓ Remove services using unencrypted auth (telnet, ftp without TLS)  
✓ Enable NTP (time sync) and Syslog (logging)  
✓ One account per user — no shared accounts  
✓ Enforce strong passwords  
✓ Password aging + restrict password reuse  
✓ Lock accounts after failed login attempts  
✓ Disable unnecessary SUID/SGID binaries

```

### Password Aging
```bash
sudo chage -M 90 flash          # password expires every 90 days
sudo chage -l flash              # view current aging settings
```

### Account Locking After Failures
```bash
sudo apt install libpam-pwquality -y
# configured via /etc/pam.d/common-auth or /etc/security/faillock.conf
```

---

## SELinux
Every process, file, directory, and system object is given a LABEL.
Policy rules control access between labeled processes and objects.
Enforced by the kernel — even specifies who can append to or move a file.

```bash
getenforce              # check current mode
sestatus                # detailed status
sudo setenforce 0       # permissive (log only)
sudo setenforce 1       # enforcing (actively blocks)
```

---

## TCP Wrappers — Deeper Reference

### Two Config Files
```bash
/etc/hosts.allow      # explicitly ALLOWED — checked FIRST
/etc/hosts.deny        # explicitly DENIED — checked SECOND
```

### Syntax
```
service : host_or_network
```
### Examples
```bash
# /etc/hosts.allow
sshd : 10.129.14.0/24                  # allow SSH from subnet
ftpd : 10.129.14.10                     # allow FTP from specific IP
telnetd : .inlanefreight.local          # allow telnet from domain

# /etc/hosts.deny
ALL : .inlanefreight.com                # deny EVERYTHING from this domain
sshd : 10.129.22.22                     # deny SSH from specific IP
ftpd : 10.129.22.0/24                   # deny FTP from subnet
```

### Critical Rules
```
1. hosts.allow is checked FIRST
2. First matching rule wins
3. If host matches hosts.allow — allowed regardless of hosts.deny
4. TCP Wrappers control SERVICES not PORTS
5. NOT a replacement for a firewall — use alongside iptables/ufw

```

---

## Complete Hardened Box Checklist
```bash
# 1. Patch everything
sudo apt update && sudo apt dist-upgrade -y

# 2. Harden SSH
sudo vim /etc/ssh/sshd_config
# PermitRootLogin no
# PasswordAuthentication no
sudo systemctl restart sshd

# 3. Install fail2ban
sudo apt install fail2ban -y

# 4. Audit for privesc risks
find / -perm -4000 2>/dev/null
sudo -l
crontab -l

# 5. Run full security audit
sudo apt install lynis -y
sudo lynis audit system

# 6. Configure TCP wrappers if needed
sudo vim /etc/hosts.allow
sudo vim /etc/hosts.deny
```

---

## Security Is a Process, Not a Product
> "specific steps must always be taken to protect systems better"
> The better administrators know their system, the better their security measures will be
> There is no final state of "secure" — it requires continuous effort

Previous:
[[Day -17 Remote Desktop Protocols in Linux]]

Next:
[[Day -19 Firewall Setup]]