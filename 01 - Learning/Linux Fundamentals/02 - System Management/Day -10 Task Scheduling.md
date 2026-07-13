# Task Scheduling

## Why It Matters in Security
- Legitimate automation tool AND a persistence/privesc vector
- Attackers hide backdoors in cron jobs and systemd timers
- Writable scripts called by root's cron = privilege escalation

---

## Cron

### Cron Time Format
.   .  .  .  .   /path/to/script.sh  
│ │ │ │ │  
│ │ │ │ └── day of week (0-7, 0 and 7 = Sunday)  
│ │ │ └──── month (1-12)  
│ │ └────── day of month (1-31)  
│ └──────── hour (0-23)  
└────────── minute (0-59)

### Special Characters
| Symbol | Meaning |
|--------|---------|
| `*` | every — any value |
| `*/5` | every 5 units |
| `,` | list — e.g. 1,15,30 |
| `-` | range — e.g. 1-5 |

### Examples
```bash
0 */6 * * *      # every 6 hours
0 0 1 * *        # midnight, 1st of every month
0 0 * * 0        # midnight, every Sunday
*/15 * * * *     # every 15 minutes
30 8 * * 1-5     # 8:30 AM, Monday to Friday
```

### Special Shortcuts
```bash
@reboot     # run once at startup
@daily      # = 0 0 * * *
@hourly     # = 0 * * * *
@weekly     # = 0 0 * * 0
@monthly    # = 0 0 1 * *
@yearly     # = 0 0 1 1 *
```

### Editing Crontab
```bash
crontab -e               # edit YOUR crontab
crontab -l               # list YOUR scheduled jobs
crontab -r               # remove all your cron jobs
sudo crontab -e -u user  # edit another user's crontab
```

### System-Wide Cron Locations
```bash
/etc/crontab               # system-wide cron jobs
/etc/cron.d/                # additional system cron jobs
/etc/cron.daily/            # scripts run once daily
/etc/cron.hourly/           # scripts run once hourly
/etc/cron.weekly/           # scripts run once weekly
/etc/cron.monthly/          # scripts run once monthly
```

### Redirecting Output (IMPORTANT)
```bash
*/5 * * * * /opt/script.sh > /dev/null 2>&1            # silence everything
*/5 * * * * /opt/script.sh >> /var/log/script.log 2>&1 # log everything
```
> Always redirect — silent failures are hard to debug

### Environment Gotcha
Cron runs with a LIMITED environment — not your normal shell's PATH.
```bash
# might fail in cron
python3 script.py

# always use full paths in cron
/usr/bin/python3 /opt/script.py
```
> Script works manually but fails when scheduled = almost always a PATH issue

---

## at — One-Time Scheduled Tasks
Different from cron — runs ONCE, not repeatedly.
```bash
sudo apt install at                       # install if missing
echo "/opt/script.sh" | at 14:30          # run once today at 2:30 PM
at now + 5 minutes                        # run once, 5 mins from now
atq                                       # list pending jobs
atrm 3                                    # remove job number 3
```

---

## Systemd Timers

### Shebang Line
```bash
#!/bin/bash         # first line of every script
```
- Tells Linux which interpreter to use
- Must be the very first line, no blank line before it
- Path must be exact interpreter location
```bash
which bash          # find exact path
```

### Example Script
```bash
#!/bin/bash
echo "$(date): $(uptime)" >> /var/log/uptime_log.txt
```
```bash
chmod +x script.sh   # make executable
```
> Script location and filename are NOT fixed — put it anywhere, name it anything
> Just reference the exact path later in timer/cron config

### Service File — what to run
Location: /etc/systemd/system/uptime-logger.service
```ini
[Unit]
Description=My Service

[Service]
ExecStart=/full/path/to/script.sh
```

### Timer File — when to run it
Location: /etc/systemd/system/uptime-logger.timer
```ini
[Unit]
Description=My Timer

[Timer]
OnBootSec=3min
OnUnitActiveSec=1hour

[Install]
WantedBy=timers.target
```

> CRITICAL: .service and .timer filenames must MATCH (same base name)
> backup.service + backup.timer = linked automatically
> backup.service + cleanup.timer = NOT linked

### Key Timer Settings
| Setting | Meaning |
|---------|---------|
| OnBootSec | run once, X time after boot |
| OnUnitActiveSec | run repeatedly, every X interval |
| OnCalendar | run at specific calendar dates/times |

### Systemd Targets (memorize these two)
```ini
WantedBy=timers.target       # for timer files
WantedBy=multi-user.target   # for service files
```
> targets = checkpoints during boot that group what starts together

### Activating
```bash
sudo systemctl daemon-reload          # MUST run after adding/editing files
sudo systemctl start mytimer.timer    # start now
sudo systemctl enable mytimer.timer   # auto-start on boot
```
> Start/enable the TIMER, not the service directly

### Verifying
```bash
systemctl list-timers                       # see all active timers
systemctl status uptime-logger.timer        # check timer status
journalctl -u uptime-logger.service         # check service logs
```

### File Locations
| Path | Purpose |
|------|---------|
| /etc/systemd/system/ | YOUR custom services/timers go here |
| /lib/systemd/system/ | system/distro defaults — don't touch |

---

## Cron vs Systemd Comparison
| | Cron | Systemd Timers |
|---|------|----------------|
| Files needed | 1 script + crontab line | script + .service + .timer |
| Setup | One line | Two config files |
| Reload needed | No | Yes — daemon-reload |
| Logging | Manual/syslog | Built-in via journalctl |
| Complexity | Simple | More flexible, more verbose |

---

## Security — Checking for Malicious Persistence
```bash
crontab -l                              # current user's cron
sudo crontab -l -u root                 # root's cron (if accessible)
cat /etc/crontab
ls -la /etc/cron.d/ /etc/cron.daily/ /etc/cron.hourly/

systemctl list-timers --all             # all timers including disabled

# recently added services/timers — possible backdoors
find / -name "*.service" -newer /etc/passwd 2>/dev/null
find / -name "*.timer" -newer /etc/passwd 2>/dev/null
```

### Common Hiding Spots for Attackers
- Cron job calling a script in /tmp/
- Disguised systemd service with legit-sounding name
- @reboot cron entry

### Privilege Escalation Pattern
If root's cron runs a script that YOUR user can write to:

```bash
ls -la /opt/backup.sh           # check permissions

# if writable — privesc opportunity
echo "chmod +s /bin/bash" >> /opt/backup.sh
# wait for cron to run as root → bash now has SUID
```

Previous:
[[Day -9  Service and Process Management]]

Next:
[[Day -11 Network Services]]