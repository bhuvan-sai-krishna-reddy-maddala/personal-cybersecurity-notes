
## The Big Picture

```txt
RDP → Windows world  
VNC → Linux world (cross-platform too)  
X11 → underlying display protocol Linux itself uses
```

## Core Theory — The One Distinction That Matters
```txt
X11 → renders LOCALLY, sends only data over network (efficient, old, insecure)  
VNC → renders REMOTELY, streams whole screen as an image (universal, heavier)
```

## The One Security Lesson That Matters
> NONE of these are encrypted by default.
> Always tunnel through SSH if security matters.
> X11 → use ssh -X
> VNC → use ssh -L tunnel

---
## X11 / XServer

### Network Transparency Concept
```txt
Application runs on Server A  
Display/rendering happens on Server B (your screen)
```
Opposite of VNC/RDP — X11 sends only DATA, not images.
### X11 Ports
```bash
TCP 6000 ← first display (:0)  
TCP 6001-6009 ← additional displays
````

### Security Issue — Unencrypted by Default
```bash
xwd          # capture screenshots of X windows (attacker tool)
xgrabsc      # similar screen capture tool
```
> Open X server (6000-6010) = potential to watch user's screen without traditional sniffing

### X11 Forwarding (Secure Method)
```bash
# server side
cat /etc/ssh/sshd_config | grep X11Forwarding
```

```bash
X11Forwarding yes
```

```bash
# client side
ssh -X htb-student@10.129.23.11 /usr/bin/firefox
```
-X = enables X11 forwarding, tunnels traffic through SSH (encrypted)
### CVE Reference

CVE-2017-2624, CVE-2017-2625, CVE-2017-2626
X.Org Server vulns from weak session keys — allowed arbitrary code execution

---
## XDMCP
- Manages remote X sessions over UDP port 177
- Insecure by design, vulnerable to MITM attacks
- Rarely set up yourself — recognize as a red flag if found exposed during recon

---
## VNC — Virtual Network Computing

### Ports
```bash
5900 ← display :0  
5901 ← display :1  
5902 ← display :2
```
Pattern: 590 + display number
### Common VNC Tools
```bash
TigerVNC, TightVNC, RealVNC, UltraVNC
````
> UltraVNC and RealVNC most used — better encryption/security

---
## TigerVNC Setup — Reference Only (look up when needed)

```bash
# Install
sudo apt install xfce4 xfce4-goodies tigervnc-standalone-server -y

# Set VNC password (separate from system login)
vncpasswd
```

```bash
# Create config files
touch ~/.vnc/xstartup ~/.vnc/config

cat <<EOT >> ~/.vnc/xstartup
#!/bin/bash
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
/usr/bin/startxfce4
EOT
```

```bash
chmod +x ~/.vnc/xstartup        # must be executable

cat <<EOT >> ~/.vnc/config
geometry=1920x1080
dpi=96
EOT

# Start server
vncserver

# List active sessions
vncserver -list
```

```txt
X DISPLAY # RFB PORT # PROCESS ID  
:1 5901 79746
```

---
## Securing VNC via SSH Tunnel

```bash
ssh -L 5901:127.0.0.1:5901 -N -f -l htb-student 10.129.14.130
```

```
-L 5901:127.0.0.1:5901 ← forward LOCAL port to REMOTE port  
-N ← don't execute remote command, just forward  
-f ← run in background  
-l htb-student ← login as this user
```

```bash
xtightvncviewer localhost:5901      # connects through the tunnel
```

---
## Encryption Comparison Table
| Protocol | Encrypted by Default? | Notes |
|----------|----------------------|-------|
| X11 | No | Use ssh -X to secure |
| XDMCP | No | Vulnerable to MITM, avoid |
| VNC | Partial (auth only) | Tunnel through SSH for full security |
| RDP | Mostly Yes | Windows-specific, has had own CVEs |

---
## Pentesting Angle
```bash
# recon for exposed ports
nmap -p 6000-6010,5900-5910 target-ip

# if X11 found open/unauthenticated
xwd -root -screen -display target-ip:0 -out screenshot.xwd

# if VNC found with weak/no auth
vncviewer target-ip:5900
```

> Open, unauthenticated VNC/X11 = instant visual access to that machine — major finding

Previous:
[[Day -16 Networking Configuration]]

Next:
[[Day -18 Linux Security]]