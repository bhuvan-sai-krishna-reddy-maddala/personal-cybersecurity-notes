## ifconfig vs ip
```txt
ifconfig    → older, deprecated but still common
ip          → newer, modern standard
```

```bash
ifconfig              # old
ip addr / ip a         # new
```

---
## Interface Naming
### Old (sequential)

eth0, eth1, lo
### New (predictable naming — based on hardware location)

ens33, enp0s3, eno1

en = ethernet  
s = slot  
p = PCI bus  
o = onboard

> Old naming could randomly swap on reboot. New naming is fixed to hardware location.

```bash
ip a        # check what YOUR system actually calls its interfaces
```

---
## Reading ifconfig/ip Output

### Flags
| Flag | Meaning |
|------|---------|
| UP | interface active/enabled |
| BROADCAST | can send broadcast packets |
| RUNNING | operational, link detected |
| MULTICAST | can send multicast packets |
| LOOPBACK | this is the loopback interface (lo only) |
### Other Fields
```txt
mtu 1500 → Maximum Transmission Unit, largest packet size in bytes  
inet → IPv4 address  
inet6 → IPv6 address  
ether → MAC address (hardware address of network card)
```

### RX / TX Stats
```
RX = data received  
TX = data transmitted  
RX/TX errors, dropped, overruns = non-zero values indicate network problems
```

---
## Configuring Interfaces
```bash
sudo ifconfig eth0 up                      # bring interface up (old)
sudo ip link set eth0 up                   # bring interface up (new)

sudo ifconfig eth0 192.168.1.2              # assign IP
sudo ifconfig eth0 netmask 255.255.255.0    # assign netmask

sudo route add default gw 192.168.1.1 eth0  # set default gateway
```
> Default gateway = router that sends traffic outside local network

---
## DNS Configuration
```bash
sudo vim /etc/resolv.conf
```

```txt
nameserver 8.8.8.8  
nameserver 8.8.4.4
```
> WARNING: gets overwritten by NetworkManager/systemd-resolved
> For permanent DNS, use /etc/network/interfaces instead

---
## Persistent Network Config — /etc/network/interfaces

```bash
sudo vim /etc/network/interfaces
```


auto eth0  
iface eth0 inet static  
address 192.168.1.2  
netmask 255.255.255.0  
gateway 192.168.1.1  
dns-nameservers 8.8.8.8 8.8.4.4
### Line by Line Meaning
```
auto eth0 → bring this interface up automatically at boot  
iface eth0 inet static → configure eth0, IPv4, STATIC IP (not DHCP)  
address 192.168.1.2 → this machine's IP address  
netmask 255.255.255.0 → defines size of local network  
gateway 192.168.1.1 → router for traffic outside local network  
dns-nameservers 8.8.8.8 8.8.4.4 → servers to resolve domain names

````

### How to Know What Values to Use
```bash
ip a                  # see current IP/netmask
ip route               # see current gateway
cat /etc/resolv.conf   # see current DNS servers
```
> Copy your CURRENT working DHCP values into static config to make them permanent

### Static vs DHCP
```
static = manually set every value, never changes  
dhcp = router automatically assigns values

iface eth0 inet dhcp # automatic  
iface eth0 inet static # manual, fixed

```
> Most home setups use dhcp. Static is for servers/labs where IP must never change.

### Apply Changes
```bash
sudo systemctl restart networking
```

---
## Network Access Control (NAC) Models

| Model | Who Decides Access |
|-------|---------------------|
| DAC (Discretionary) | resource OWNER decides |
| MAC (Mandatory) | OS enforces fixed security levels — owner has no say |
| RBAC (Role-Based) | access based on job role/group |

### Simplified
DAC → resource owner sets permissions on their own resource (chmod/chown)  
MAC → user/process security clearance must be >= resource's security level  
RBAC → role assigned to user, role has permissions, user inherits them via role  
(exactly like Linux groups — sudo group, developers group, etc.)
### IMPORTANT — MAC vs MAC Naming Collision

MAC (Mandatory Access Control) → security model  
MAC (Media Access Control) → hardware address (network card)

> Same acronym, completely unrelated concepts — context tells you which is meant
```bash
ip a            # ether field = MAC address (hardware)
getenforce      # relates to MAC (access control / SELinux)
```

---
## Network Troubleshooting Tools

### ping — Connectivity Test
```bash
ping 8.8.8.8
ping -c 4 8.8.8.8        # send only 4 packets
```
No response = connectivity issue, blocked ICMP, or host down
### traceroute — Map the Path
```bash
traceroute www.inlanefreight.com
```
Shows every hop (router) packet passes through
```shell
1 * * * ← no response (blocked/filtered)  
2 10.80.71.5 2.7ms 2.7ms 2.7ms ← responded
```
### netstat / ss — Active Connections
```bash
netstat -a              # all connections
netstat -tlnp           # tcp, listening, numeric, with process
```
> ss is the MODERN replacement for netstat — faster, more features

---
## Common Network Issues & Causes
| Issue | Common Causes |
|-------|---------------|
| Connectivity issues | bad cables, firewall misconfig, hardware failure |
| DNS resolution issues | wrong DNS settings, DNS server failure |
| Packet loss | network congestion, faulty hardware |
| Performance issues | outdated hardware, misconfigured settings |

---
## Security Hardening Tools

### SELinux — Mandatory Access Control, Kernel-Level
Very strict, granular, complex to configure.
```bash
sudo apt install selinux-utils selinux-basics -y
sudo selinux-activate

getenforce              # check current mode
sestatus                # detailed status

sudo setenforce 0       # permissive (log only)
sudo setenforce 1       # enforcing (actively blocks)
```
Modes: Enforcing (blocks) / Permissive (logs only) / Disabled

### Practice — Block user from a file
```bash
ls -Z /path/to/file                          # see SELinux context/label
sudo chcon -t admin_home_t /path/to/file     # change security context
```
### Practice — Network service control
```bash
sudo semanage port -l | grep http
sudo setsebool -P httpd_can_network_connect off
```

---
### AppArmor — Simpler MAC Alternative
Profile-based, easier than SELinux.
```bash
sudo apt install apparmor apparmor-utils -y
sudo aa-status                    # check status
```
### Practice — Restrict file access
```bash
sudo aa-genprof /path/to/application     # generate profile interactively
```
### Practice — Network service restriction
```bash
sudo aa-complain /etc/apparmor.d/usr.sbin.sshd    # observe mode first
sudo aa-enforce /etc/apparmor.d/usr.sbin.sshd     # enforce after testing
```

---
### TCP Wrappers — Network-Level Access Control
Controls service access by IP address.
```bash
sudo apt install tcpd -y
```

```
/etc/hosts.allow # explicitly allowed IPs  
/etc/hosts.deny # explicitly denied IPs
```
Format: `service: IP_or_range`
### Practice Examples
```bash

echo "sshd: 192.168.1.100" | sudo tee -a /etc/hosts.allow      # allow specific IP
echo "sshd: 192.168.1.200" | sudo tee -a /etc/hosts.deny       # deny specific IP
echo "sshd: 192.168.1.0/24" | sudo tee -a /etc/hosts.allow     # allow a range
```

---
## SELinux vs AppArmor vs TCP Wrappers
|             | SELinux           | AppArmor           | TCP Wrappers       |
| ----------- | ----------------- | ------------------ | ------------------ |
| Type        | MAC, kernel-level | MAC, profile-based | network-level only |
| Complexity  | high              | medium             | low                |
| Granularity | very fine         | moderate           | IP-based only      |
| Ease of use | hardest           | easier             | simplest           |

---
## Security Use in Pentesting
```bash
ss -tlnp                # recon - what's listening
ping target-ip           # check connectivity
traceroute target-ip     # map network path
getenforce               # check if SELinux blocks your payload
sudo aa-status           # check AppArmor status
```
> If target has SELinux in Enforcing mode — many privesc techniques get blocked
> Worth checking early in an engagement

---
## Quick Reference
| Tool | Purpose |
|------|---------|
| ip a / ifconfig | view/configure interfaces |
| ping | test connectivity |
| traceroute | map network path |
| netstat / ss | view active connections |
| nslookup / dig | DNS troubleshooting |
| getenforce | check SELinux status |
| aa-status | check AppArmor status |
| /etc/hosts.allow /etc/hosts.deny | TCP wrappers config |
## Additions

### The Office Building Analogy
Network interfaces = wiring/infrastructure
NAC = building security (some rooms open to all - DAC, others restricted - MAC/RBAC)
Monitoring = surveillance cameras/alarms
Troubleshooting = toolkit:
	ping = checking a connection
	nslookup = checking a lock (DNS)
	nmap = checking an entrance (port scanning)

### nslookup — DNS Lookup Tool
```bash
nslookup google.com          # domain → IP
nslookup 8.8.8.8              # IP → domain (reverse lookup)
```

### Monitoring Tools (Full List)
| Tool | Purpose |
|------|---------|
| syslog / rsyslog | system logging |
| ss | socket statistics |
| lsof | list open files |
| ELK stack | Elasticsearch + Logstash + Kibana — log analysis platform |

### Full Troubleshooting Tool List
| Tool | Purpose |
|------|---------|
| ping | basic connectivity test |
| traceroute | map network path |
| netstat | active connections |
| tcpdump | CLI packet capture |
| wireshark | GUI packet capture/analysis |
| nmap | port scanning |

> Deep coverage of tcpdump/wireshark/nmap comes in the dedicated 
> Network Traffic Analysis module — this section just introduces them

### Monitoring — Real Pentest Example
Capturing credentials when a user connects via UNENCRYPTED FTP
— same lesson as Section 19, reinforced in a monitoring context

> Always check if traffic is encrypted before trusting it

Previous:
[[Day -15 Containerization]]

Next:
[[Day -17 Remote Desktop Protocols in Linux]]