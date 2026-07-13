## Theory — Why Logs Matter

### Offensive (Pentester) Perspective
```txt
Logs reveal credentials, misconfigs, running services, user activity  
After engagement — check if YOUR activity triggered any alerts  
Find sensitive data accidentally logged (passwords, API keys, tokens)
```

### Defensive (Blue Team) Perspective
```txt
Logs are your eyes — only way to reconstruct what happened  
Incident response starts with log analysis  
Legal evidence for forensic investigation
````

---
## Five Types of System Logs

| Type           | Location                                            | Contains                                      |
| -------------- | --------------------------------------------------- | --------------------------------------------- |
| Kernel         | /var/log/kern.log                                   | hardware drivers, system calls, kernel events |
| System         | /var/log/syslog                                     | service starts/stops, reboots, cron jobs      |
| Authentication | /var/log/auth.log (Ubuntu) /var/log/secure (CentOS) | login attempts, sudo usage, user switching    |
| Application    | varies per app                                      | app-specific activity, access, errors         |
| Security       | varies per tool                                     | fail2ban bans, firewall blocks, audit events  |

---
## 1. Kernel Logs
```bash
/var/log/kern.log
```

Security relevance:
```txt
Outdated/vulnerable drivers → attack surface  
Suspicious system calls → possible malware  
Unexpected crashes → possible DoS or exploitation attempt

```

```bash
tail -f /var/log/kern.log
grep -i "error" /var/log/kern.log
```

---
## 2. System Logs
```bash
/var/log/syslog
```

### Reading a Syslog Entry
```bash
Feb 28 2023 15:00:01 server CRON[2715]: (root) CMD (/usr/local/bin/backup.sh)  
│ │ │ │ │  
│ │ │ │ └── What happened  
│ │ │ └── Process name and PID  
│ │ └── Hostname  
│ └── Timestamp  
└── Date

```

Security relevance:
```txt
Unusual cron jobs → persistence mechanism  
Failed service starts → misconfiguration or tampering  
Unexpected reboots → exploitation or instability

```

---
## 3. Authentication Logs — Most Important for Security
```bash
/var/log/auth.log          # Ubuntu/Debian
/var/log/secure             # CentOS/RHEL
```

Specifically tracks:
```txt
SSH login attempts (success AND failure)  
sudo usage — who ran what as root  
User switching (su)  
PAM authentication events
```

### Reading an Auth Log Entry
```bash
Feb 28 2023 15:04:22 server sshd[3010]: Failed password for htb-student from 10.14.15.2 port 50223 ssh2  
│ │ │  
│ │ └── Attacker IP  
│ └── Username tried  
└── Login attempt failed

```

### Key Searches
```bash
# Brute force — many failed logins from same IP
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Successful logins after failures — brute force that worked
grep "Accepted password" /var/log/auth.log

# Sudo abuse — who ran what as root
grep "sudo" /var/log/auth.log

# Public key auth — legitimate or stolen key
grep "Accepted publickey" /var/log/auth.log

# New users created
grep "useradd\|adduser" /var/log/auth.log
```

---
## 4. Application Logs

### Log Locations by Service
| Service | Log Location |
|---------|-------------|
| Apache | /var/log/apache2/access.log |
| Nginx | /var/log/nginx/access.log |
| OpenSSH | /var/log/auth.log (Ubuntu) |
| MySQL | /var/log/mysql/mysql.log |
| PostgreSQL | /var/log/postgresql/postgresql-version-main.log |
| Systemd | /var/log/journal/ |
### Reading an Apache Access Log Entry
```bash

127.0.0.1 - - [28/Feb/2023:15:06:43] "GET /index.html HTTP/1.1" 200 13484  
│ │ │ │ │ │  
│ │ │ │ │ └── Response size  
│ │ │ │ └── HTTP status code  
│ │ │ └── Request made  
│ │ └── Auth (- = none)  
│ └── Timestamp  
└── IP address

```

### HTTP Status Code Reference
```bash
200 = success  
403 = forbidden  
404 = not found  
500 = server error
```

### Security Patterns in Web Logs
```txt
403/404 storms → directory brute forcing (ffuf, gobuster)  
SQL keywords in URLs → SQL injection attempts  
../ patterns → path traversal attempts  
Unusual User-Agents → automated scanners/tools  
POST to odd endpoints → exploitation attempts

```

```bash
# Find directory brute forcing (404 storm)
grep " 404 " /var/log/apache2/access.log | awk '{print $1}' | sort | uniq -c | sort -rn

# Find SQL injection attempts
grep -i "select\|union\|insert\|drop" /var/log/apache2/access.log

# Find path traversal attempts
grep "\.\." /var/log/apache2/access.log
```

---
## 5. Security Logs

| Tool | Log Location |
|------|-------------|
| fail2ban | /var/log/fail2ban.log |
| UFW firewall | /var/log/ufw.log |
| auditd | /var/log/audit/audit.log |

```bash
grep "Ban" /var/log/fail2ban.log          # who got banned
grep "BLOCK" /var/log/ufw.log             # what UFW blocked
grep "api-keys" /var/log/audit/audit.log  # sensitive file access
```

### Access Log Entry Example (auditd)
```txt

2023-03-07T10:15:23+00:00 servername privileged.sh: htb-student accessed /root/hidden/api-keys.txt

```

Red flags:
```txt
/root/hidden/ ← regular user accessing root's hidden directory  
api-keys.txt ← sensitive credential file  
htb-student ← should NOT have this access

```

---
## Log Analysis Commands — Full Toolkit

```bash
tail -f /var/log/auth.log              # watch live in real time
tail -n 100 /var/log/syslog            # last 100 lines
grep "Failed" /var/log/auth.log        # filter specific pattern
grep -i "error" /var/log/kern.log      # case insensitive
grep -v "CRON" /var/log/syslog         # exclude CRON noise
```

### Power Combinations
```bash
# Top 10 IPs with failed SSH logins
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn | head -10

# All sudo commands run today
grep "sudo" /var/log/auth.log | grep "$(date +%b\ %d)"

# Timeline of specific user activity
grep "htb-student" /var/log/auth.log | tail -50

# Privilege escalation investigation
grep "sudo\|su" /var/log/auth.log | grep -v "session closed"
```

---
## Log Rotation
Logs would grow infinitely — logrotate handles this automatically.

```bash
cat /etc/logrotate.conf
ls /etc/logrotate.d/
```

Rotated logs get numbered/compressed:
```bash

/var/log/auth.log ← current  
/var/log/auth.log.1 ← yesterday  
/var/log/auth.log.2.gz ← older, compressed

```

```bash
zcat /var/log/auth.log.2.gz | grep "Failed"    # read compressed old logs
```

---
## Log Configuration Best Practices
```
Set appropriate log levels — too verbose = noise, too quiet = blind  
Configure log rotation — prevent logs filling disk  
Store logs securely — protected from unauthorized access  
Regularly review logs — not just when incidents happen  
Consider centralized logging — ship to SIEM (ELK, Splunk)

```

---
## Security Mindset — Logs in Pentesting

### During an Engagement
```bash
# Check if YOUR activity was detected
grep "your-ip" /var/log/auth.log
grep "your-ip" /var/log/apache2/access.log
grep "your-ip" /var/log/fail2ban.log
grep "your-ip" /var/log/ufw.log
```

### Covering Tracks (only in authorized engagements)
```bash
echo "" > /var/log/auth.log         # clear auth log (obvious, creates gaps)
```
> Clearing logs is detectable — creates obvious timestamp gaps
> Professional red teams use more subtle techniques

### Blue Team Incident Response Checklist
```bash
# Timeline reconstruction
grep "Feb 28" /var/log/auth.log | grep -E "Accepted|Failed|sudo"

# New users created during incident
grep "useradd\|adduser" /var/log/auth.log

# Privilege escalation events
grep "sudo\|su" /var/log/auth.log | grep -v "session closed"

# Cron persistence check
grep "CRON" /var/log/syslog | grep "root"
```

---

## The syslog Example — What Each Line Tells You

```bash

Feb 28 15:00:01 CRON[2715]: (root) CMD (/usr/local/bin/backup.sh) → cron running as root  
Feb 28 15:04:22 sshd[3010]: Failed password for htb-student → failed login attempt  
Feb 28 15:05:02 kernel: ata3.00: exception Emask → disk/hardware issue  
Feb 28 15:06:43 apache2[2904]: GET /index.html 200 → web request served  
Feb 28 15:07:19 sshd[3010]: Accepted password for htb-student → LOGIN SUCCEEDED after failure  
Feb 28 15:09:54 kernel: EXT4-fs (sda1): re-mounted → filesystem remounted  
Feb 28 15:12:07 systemd[1]: Started Clean PHP session files → scheduled cleanup

```

> Line 4 + Line 6 together = FAILED then SUCCEEDED = brute force that worked
> This is exactly what incident responders look for in log correlation

---
## The auth.log Example — Red Flags Analysis
```bash
18:15:01 Accepted publickey for admin from 10.14.15.2 → SSH key login  
18:15:03 sudo: admin → USER=root COMMAND=/bin/bash → SPAWNED ROOT SHELL  
18:15:05 sudo: admin → USER=root COMMAND=apt-get install netcat-traditional → INSTALLED NETCAT  
18:15:08 sshd: Disconnected from 10.14.15.2 → session ended  
18:15:12 kernel: firewall: unexpected traffic on port 22 → FIREWALL ALERT  
18:15:15 auditd: Audit daemon started → auditing started  
18:15:21 CRON: session opened for root → root cron running

```

Red flags:
```bash
Admin immediately spawned /bin/bash as root → privilege abuse  
Installed netcat right after root access → possible backdoor/C2 setup  
Unexpected traffic on port 22 → possibly scanning or tunneling
```

Previous:
[[Day -19 Firewall Setup]]

Next:
[[Home.base]]
