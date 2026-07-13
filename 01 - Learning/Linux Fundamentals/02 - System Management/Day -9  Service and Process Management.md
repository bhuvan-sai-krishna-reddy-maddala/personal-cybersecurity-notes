
## What is a Service/Daemon
Runs in background without user interaction. Named with d at end usually:
sshd = SSH daemon
systemd = system daemon

## Two Types of Services
| Type | Description |
|------|-------------|
| System Services | required at startup, hardware-related, essential |
| User-Installed Services | added by users, optional features |

---

## systemctl — Control Services
```bash
systemctl start ssh                    # start service
systemctl stop ssh                     # stop service
systemctl restart ssh                  # restart service
systemctl status ssh                   # check status
systemctl enable ssh                   # auto-start on boot
systemctl disable ssh                  # don't auto-start
systemctl list-units --type=service    # list all services
```
> systemctl = remote control for services

---

## journalctl — Service Logs
```bash
journalctl -u ssh.service              # logs for ssh service
journalctl -u ssh.service --no-pager   # print all at once, no pager
journalctl -u ssh.service -f           # follow live logs
journalctl -u ssh.service --since today
```
> journalctl = logbook kept by systemd for every service

---

## Process Basics
| Term | Meaning |
|------|---------|
| PID | Process ID — unique number for running process |
| PPID | Parent Process ID — who started this process |

## ps — List Processes
```bash
ps                  # only your processes in current terminal
ps -e               # every process on system
ps -ef              # every process, full format
ps -aux             # every process, detailed (BSD style)
ps -aux | grep ssh  # filter for ssh process
```

### Common Flags
| Flag | Meaning |
|------|---------|
| a | all processes with a terminal |
| u | user-friendly format (user, CPU%, MEM%) |
| x | processes without a terminal too |
| e | every process |
| f | full format listing |

### Getting Help
```bash
man ps                  # full manual
ps --help               # quick summary categories
ps --help all           # everything in one go
ps --help simple        # simple selection options
ps --help list          # list format options
ps --help output        # output format options
```

---

## top / htop — Live Monitor
```bash
top          # live process view
htop         # nicer interface (may need install)
```
Inside top:
q → quit  
k → kill process by PID  
M → sort by memory  
P → sort by CPU

---

## /proc/ Directory
```bash
ls /proc/1234              # info about process PID 1234
cat /proc/1234/status      # detailed status
cat /proc/1234/cmdline     # exact command that started it
```
> ps actually reads from /proc/ to get its information

---

## Process States

Running — actively executing  
Waiting — waiting on resource/event  
Stopped — paused, not running  
Zombie — finished but entry not cleaned up

---

## Signals
```bash
kill -l       # list all signals
```

| Number | Signal | Meaning |
|--------|--------|---------|
| 1 | SIGHUP | terminal closed |
| 2 | SIGINT | Ctrl+C — interrupt |
| 3 | SIGQUIT | Ctrl+D — quit |
| 9 | SIGKILL | force kill, no cleanup |
| 15 | SIGTERM | normal termination request |
| 19 | SIGSTOP | pause, can't be ignored |
| 20 | SIGTSTP | Ctrl+Z — suspend, resumable |

```bash
kill 9 <PID>            # force kill
kill -15 <PID>          # polite termination
killall firefox         # kill all processes by name
pkill -9 firefox        # kill by name pattern
pgrep firefox           # find PID by name
```
> kill -9 = no cleanup, last resort
> SIGTERM (15) = polite way first

---

## nohup — Survive Terminal Closing
```bash
nohup long_running_script.sh &
```
Ignores SIGHUP — process survives terminal disconnect.
Useful for long scans during pentesting.

---

## Background & Foreground

### Ctrl+Z — Suspend (sends SIGTSTP)
```bash
ping -c 10 google.com
[Ctrl+Z]
jobs
```

### bg — Resume in Background
```bash
bg
```

### & — Start in Background Directly
```bash
ping -c 10 google.com &
```

### fg — Bring to Foreground
```bash
fg 1
```

### jobs — List Background Jobs
```bash
jobs
```

---

## Combining Commands
```bash
# ; — run regardless of success/failure
echo '1'; ls MISSING_FILE; echo '3'

# && — only continue if previous succeeded
echo '1' && ls MISSING_FILE && echo '3'

# | — pipe output to next command
command1 | command2
```

| Symbol | Use When |
|--------|----------|
| `;` | run all commands no matter what |
| `&&` | only continue if previous succeeded |
| `\|` | feed output of one into another |
| `&` | run in background |

---

## Security Angle
```bash
ps -aux                   # spot unusual processes
ps -aux | grep -v grep    # exclude grep's own process
ps -ef --forest           # parent-child process tree — spot weird spawns
```
> A webserver spawning bash = red flag during incident response

Previous:
[[Day -9  Service and Process Management]]

Next:
[[Day -10 Task Scheduling]]