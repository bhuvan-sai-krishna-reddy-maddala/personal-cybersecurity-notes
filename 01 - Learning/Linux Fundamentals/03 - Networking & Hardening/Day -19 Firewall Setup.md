## What Firewalls Do
Filter incoming and outgoing traffic based on pre-defined rules,
protocols, ports, and criteria to prevent unauthorized access.

## Linux Firewall Evolution
```txt
ipfwadm → ipchains → iptables (2000, kernel 2.4) → nftables (modern)
```

All built on the **Netfilter** framework — integral part of the kernel.
Netfilter provides hooks to intercept and modify traffic as it passes through.

## Firewall Alternatives
| Tool | Notes |
|------|-------|
| iptables | default, flexible, widely used |
| nftables | modern replacement, better performance, incompatible syntax |
| ufw | "Uncomplicated Firewall" — friendly wrapper around iptables |
| firewalld | dynamic, zone-based, used heavily on Red Hat/CentOS |

---
## iptables — Core Mental Model
```txt
Tables → categories of what you're doing  
└── Chains → stages of traffic flow  
	└── Rules → specific conditions to check (top to bottom)  
		└── Targets → what to DO if rule matches

```
> Think of it as a flowchart — packet arrives, checked against rules in order,
> FIRST match decides its fate, stops checking after first match

---
## Tables — Four Types

| Table | Job | Built-in Chains |
|-------|-----|-----------------|
| filter | allow/block traffic (DEFAULT) | INPUT, OUTPUT, FORWARD |
| nat | change source/destination IPs | PREROUTING, POSTROUTING |
| mangle | modify raw packet headers | PREROUTING, OUTPUT, INPUT, FORWARD, POSTROUTING |
| raw | bypass connection tracking | PREROUTING, OUTPUT |

> 99% of basic firewall work = filter table
> Don't need -t flag for filter (it's the default)
> For other tables: sudo iptables -t nat / -t mangle / -t raw

---
## Chains — Direction of Traffic

### filter table chains
```txt
INPUT → traffic coming INTO this machine  
OUTPUT → traffic going OUT from this machine  
FORWARD → traffic passing THROUGH (this machine acting as a router)
```
> INPUT = mail arriving at your house
> OUTPUT = mail you're sending out
> FORWARD = mail passing through your house to someone else

### nat table chains
```txt
PREROUTING → modify destination IP BEFORE routing decision  
POSTROUTING → modify source IP AFTER routing decision

```

### User-Defined Chains
```bash
sudo iptables -N MYCHAIN          # create new chain
sudo iptables -A INPUT -p tcp --dport 80 -j MYCHAIN   # send traffic into it
sudo iptables -A MYCHAIN -j ACCEPT                     # rules inside it
```
> Useful for organizing complex rulesets — group related rules together

---

## Rules — Syntax Breakdown

```bash
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

```
-A INPUT ← Append rule to INPUT chain  
-p tcp ← Protocol must be TCP  
--dport 22 ← Destination port must be 22  
-j ACCEPT ← Jump to ACCEPT target (allow it through)

```

### Key Flags
| Flag | Meaning |
|------|---------|
| -A | Append rule to chain |
| -I | Insert rule at specific position |
| -D | Delete rule |
| -N | New user-defined chain |
| -F | Flush (clear) all rules |
| -L | List rules |
| -t | Specify table (default: filter) |
| -p | Protocol (tcp, udp, icmp) |
| --dport | Destination port |
| --sport | Source port |
| -s | Source IP address |
| -d | Destination IP address |
| -j | Jump to target |
| -v | Verbose (with -L) |
| --line-numbers | Show rule numbers (with -L) |

---
## Targets — What to Do When Rule Matches

| Target | What Happens |
|--------|-------------|
| ACCEPT | let it through |
| DROP | silently block — no response sent back |
| REJECT | block AND notify sender with error message |
| LOG | just log it, don't block or allow |
| SNAT | change source IP (for NAT) |
| DNAT | change destination IP (port forwarding) |
| MASQUERADE | like SNAT but for dynamic IPs (home routers) |
| REDIRECT | redirect to another port or IP |
| MARK | add internal kernel mark for routing decisions |

### DROP vs REJECT — Critical Distinction
```txt
DROP → attacker gets NOTHING back, port looks non-existent  
REJECT → attacker gets error message, knows it's actively blocked

```
> DROP preferred — gives attackers less information

---
## -j (Jump) — Explained

```bash
-j = jump to this TARGET or CHAIN

```

Not just for built-in targets — can jump to user-defined chains too:

```bash
-j ACCEPT           # jump to built-in target
-j MYCHAIN          # jump into a custom chain for further processing
```

---
## Order Matters — CRITICAL

Rules checked TOP TO BOTTOM, first match wins, stops there.

```bash
# WRONG — everyone dropped before specific allow is checked
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
sudo iptables -A INPUT -p tcp -s 192.168.1.100 --dport 80 -j ACCEPT

# CORRECT — allow specific IP first, then block everyone else
sudo iptables -A INPUT -p tcp -s 192.168.1.100 --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
```

---
## -m (Match Extensions) — Advanced Matching

### 1. -m state — Connection State
```bash
-m state --state NEW           # brand new connection
-m state --state ESTABLISHED   # part of already accepted connection
-m state --state RELATED       # related to established (e.g. FTP data)
-m state --state INVALID       # doesn't match any known state
```

### Stateful Firewall Pattern (Real World)
```bash
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -m state --state NEW -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -j DROP
```
> Allow traffic from connections YOU already started
> Allow new SSH connections
> Drop everything else

### 2. -m multiport — Multiple Ports at Once
```bash
sudo iptables -A INPUT -p tcp -m multiport --dports 80,443,8080 -j ACCEPT
```

### 3. -m limit — Rate Limiting (DoS Protection)
```bash
sudo iptables -A INPUT -p icmp -m limit --limit 1/second -j ACCEPT
sudo iptables -A INPUT -p icmp -j DROP
```
> Allows 1 ping per second — excess dropped (ping flood protection)

### 4. -m string — Packet Content Matching
```bash
sudo iptables -A INPUT -m string --string "malware" --algo bm -j DROP
```

### 5. -m mac — MAC Address Matching
```bash
sudo iptables -A INPUT -m mac --mac-source 8a:d9:fa:cf:79:7a -j ACCEPT
```
> Only works on same network segment — MAC doesn't survive routing

### 6. -m iprange — IP Range Matching
```bash
sudo iptables -A INPUT -m iprange --src-range 192.168.1.1-192.168.1.50 -j DROP
```

### 7. -m conntrack — Modern Connection Tracking
```bash
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```
> Modern version of -m state — more detailed, same concept

---
## The Other Three Tables

### nat Table — Translating IPs
```bash
# MASQUERADE — all outgoing looks like it comes from THIS machine (home router)
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# DNAT — port forwarding, redirect incoming to internal server
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:80
```
> This is how home routers work — your 192.168.x.x devices all share one public IP

### mangle Table — Modifying Packet Headers
Modify raw fields in packet headers — not drop/allow, not change IPs.

What you can change:

```txt
TTL (Time To Live) → how many hops before packet dies  
TOS (Type of Service) → priority/QoS marking  
MARK → internal kernel mark for routing decisions  
DSCP → QoS traffic classification

```


```bash
# TTL manipulation — prevent ISP tethering detection
sudo iptables -t mangle -A PREROUTING -i eth0 -j TTL --ttl-set 64

# Mark packets for advanced routing
sudo iptables -t mangle -A PREROUTING -p tcp --dport 80 -j MARK --set-mark 1

# QoS — prioritize SSH traffic
sudo iptables -t mangle -A PREROUTING -p tcp --dport 22 -j DSCP --set-dscp 46
```

### raw Table — Bypass Connection Tracking
```bash
# NOTRACK — skip connection tracking for this traffic
sudo iptables -t raw -A PREROUTING -p tcp --dport 80 -j NOTRACK
```

> Performance optimization for very high-traffic servers

---

## Packet Processing Order (Full Picture)
```bash
Incoming packet:

1. raw PREROUTING ← bypass connection tracking?
2. mangle PREROUTING ← modify headers?
3. nat PREROUTING ← change destination IP?
4. [routing decision]
5. mangle INPUT ← modify headers again?
6. filter INPUT ← allow or block?

````

---
## Practical Examples

```bash
# Block traffic on a port
sudo iptables -A INPUT -p tcp --dport 8080 -j DROP

# Allow traffic on a port
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT

# Block specific IP
sudo iptables -A INPUT -s 192.168.1.50 -j DROP

# Allow specific IP
sudo iptables -A INPUT -s 192.168.1.50 -j ACCEPT

# Block by protocol (block ping)
sudo iptables -A INPUT -p icmp -j DROP

# Allow by protocol
sudo iptables -A INPUT -p icmp -j ACCEPT

# Multiple conditions (AND logic — all must match)
sudo iptables -A INPUT -p tcp -s 192.168.1.100 --dport 22 -j ACCEPT
```

---
## Listing and Managing Rules

```bash
sudo iptables -L                    # list all rules
sudo iptables -L -v                  # verbose — show packet/byte counts
sudo iptables -L --line-numbers      # show rule numbers
sudo iptables -D INPUT 3             # delete rule number 3 in INPUT
sudo iptables -D INPUT -p tcp --dport 8080 -j DROP   # delete by matching rule
sudo iptables -F                     # flush ALL rules (careful on remote servers!)
```

---
## Saving Rules (Persistence)
iptables rules vanish on reboot unless saved.
```bash
sudo apt install iptables-persistent -y
sudo netfilter-persistent save
```

---
## UFW — Simple Alternative
```bash
sudo ufw enable
sudo ufw allow 22/tcp
sudo ufw deny 8080/tcp
sudo ufw status
```

> Same underlying iptables engine, much friendlier syntax
> Use iptables for deep control, ufw for quick simple setups

---
## 10 Practice Exercises

```bash
# 1. Start web server and block port 8080
python3 -m http.server 8080 &
sudo iptables -A INPUT -p tcp --dport 8080 -j DROP

# 2. Allow incoming traffic on 8080
sudo iptables -D INPUT -p tcp --dport 8080 -j DROP
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT

# 3. Block traffic from specific IP
sudo iptables -A INPUT -s <ip> -j DROP

# 4. Allow traffic from specific IP
sudo iptables -A INPUT -s <ip> -j ACCEPT

# 5. Block traffic based on protocol
sudo iptables -A INPUT -p icmp -j DROP

# 6. Allow traffic based on protocol
sudo iptables -A INPUT -p icmp -j ACCEPT

# 7. Create a new chain
sudo iptables -N MYCHAIN

# 8. Forward traffic to specific chain
sudo iptables -A INPUT -p tcp --dport 80 -j MYCHAIN

# 9. Delete a specific rule
sudo iptables -L --line-numbers
sudo iptables -D INPUT <rule_number>

# 10. List all existing rules
sudo iptables -L -v --line-numbers
```

Previous:
[[Day -18 Linux Security]]

Next:
[[Day -20 System Logs and Monitoring]]