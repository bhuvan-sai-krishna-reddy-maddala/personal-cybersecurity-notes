
> [!warning] Scope reminder Only scan hosts you own or are explicitly authorized to test — your own lab VMs, `scanme.nmap.org` (Nmap's official, permission-granted test target), or a platform like HTB/TryHackMe. Scanning anything else without written authorization is illegal, full stop.

## Table of contents

- [[#1. Theory]]
- [[#2. Examples]]
- [[#3. Exercises]]
- [[#4. Resources]]

---

## 1. Theory

### 1.1 What nmap actually does

Nmap sends crafted packets to a target and reads how it responds. That's the whole trick — every "scan type" is really just a different combination of TCP flags (or a different protocol entirely) chosen to provoke a response that reveals whether a port is open, closed, or filtered.

To make sense of scan types, you need the TCP handshake underneath them:

```
Normal 3-way handshake (full connection):
Client → SYN        → Server
Client ← SYN/ACK     ← Server
Client → ACK        → Server
```

- **Open port:** replies SYN/ACK
- **Closed port:** replies RST
- **Filtered port:** no reply at all (or an ICMP "unreachable" — firewall silently dropping it)

Everything nmap does builds on that one exchange.

### 1.2 SYN scan (`-sS`) — the default, and why it's called "half-open"

```
Client → SYN         → Server
Client ← SYN/ACK      ← Server
Client → RST (no ACK)  → Server   ← nmap tears it down here
```

Nmap sends the SYN, reads the SYN/ACK to confirm the port is open, then sends a RST instead of completing the handshake. The full connection is **never established**, which is why it's called a half-open scan. Two consequences worth remembering:

- It's faster and stealthier than a full connect, because the application layer (e.g. the web server process) often never even logs the "connection."
- It requires raw socket access, which is why `-sS` needs root/sudo.

### 1.3 Connect scan (`-sT`) — the no-privilege fallback

Completes the full 3-way handshake using the OS's normal socket API. Slower, noisier (application-level logs will show it), but works without root — the only option available to an unprivileged user.

### 1.4 Why UDP scanning (`-sU`) is fundamentally different

UDP has no handshake, so nmap can't infer state from a SYN/ACK. Instead:

- **Open port:** ideally an application-layer response (rare, protocol-dependent)
- **Closed port:** an ICMP "port unreachable" message
- **Open|filtered:** no response at all — which is the _most common_ result, because most services don't respond to unsolicited empty UDP packets. This ambiguity is exactly why UDP scans are slow and results are less certain than TCP.

### 1.5 Why the odd scans (Null, FIN, Xmas) exist

```
Null: no flags set
FIN:  FIN flag only
Xmas: FIN + PSH + URG (all "lit up" like a Christmas tree)
```

RFC 793 says a closed port should respond with RST to any of these when no SYN was sent first, while an open port silently ignores them. In theory this lets you distinguish open vs. closed without ever sending a SYN, evading firewalls that only look for SYN-flagged packets. In practice: **Windows doesn't follow this part of the RFC and responds to everything with RST**, so these scans only reliably work against Unix-like TCP/IP stacks. This is a good example of "textbook technique, but always verify against the real target" — a theme that recurs constantly in security work.

### 1.6 Timing — why it's a dial, not a switch

Nmap's timing templates (`-T0` through `-T5`) aren't really about "speed" in the abstract — they control the delay between probes and how aggressively nmap retries. Slower timing means:

- Fewer packets per unit time → less likely to trip an IDS/IPS threshold-based alert
- More patience for round-trip time, which matters on slow/unstable links (Tor, satellite, congested networks)

Faster timing is fine on a local lab network where nothing is watching. It becomes a liability the moment you care about detection — real engagements, red team exercises, or bug bounty recon against a WAF that rate-limits aggressively.

### 1.7 The scripting engine (NSE) — nmap as a mini-framework

NSE scripts are Lua programs that hook into nmap's scan lifecycle. Categories matter because they set expectations:

- `safe` — won't crash or disrupt the target
- `intrusive` — might trigger alarms or cause instability
- `vuln` — specifically checks for known vulnerabilities
- `brute` — credential brute-forcing (be careful — this is loud and can lock accounts)

This is the mechanism that turns nmap from "a port scanner" into "a lightweight vulnerability scanner and service enumerator."

### 1.8 Evasion — what it actually buys you

Every evasion technique (decoys, fragmentation, spoofing, idle scans) works by exploiting a gap between what a naive filter checks and what a full TCP/IP stack actually does. None of it makes traffic invisible — a properly tuned IDS with packet reassembly and anomaly detection will still see it. What evasion really does is:

1. Increase the _cost_ of attribution (decoys, idle scans)
2. Slip past _simple, signature-based_ filters (fragmentation, odd flag combos)
3. Reduce your _statistical footprint_ (timing, rate limiting)

Treat evasion as "raising the bar," not "becoming invisible."

---

## 2. Examples

### 2.1 A sane default first pass

```bash
nmap -sV -sC -oA initial_scan target
```

`-sV` grabs service/version banners, `-sC` runs the default safe NSE script set, `-oA` writes normal + XML + grepable output in one go — good habit for keeping evidence.

### 2.2 Full port sweep before deep enumeration

```bash
nmap -p- -T4 -oN allports.txt target
```

Run this first on an unfamiliar host — the default top-1000 ports miss plenty of real services running on high ports.

### 2.3 Feed discovered ports into a focused scan

```bash
# after allports.txt shows 22,80,8080,3306 open:
nmap -p 22,80,8080,3306 -sV -sC -oA focused_scan target
```

This two-pass pattern (fast full-port sweep → targeted deep scan) is the standard workflow and saves a lot of time versus running `-A -p-` on everything up front.

### 2.4 Quiet internal scan

```bash
nmap -sS -T2 -Pn --data-length 24 -oA quiet target
```

`-Pn` skips the ping check (assume host is up), `--data-length` pads packets so they don't match a bare "nmap default" signature some IDS rules look for.

### 2.5 Vulnerability-hunting pass

```bash
nmap -sV --script=vuln -oA vuln_scan target
```

Good second step once you know what's running — this cross-references detected versions against known-vulnerable script checks.

---

## 3. Exercises

> [!tip] How to use this section Work through these on `scanme.nmap.org` first (always allowed) or your own lab VM. Try to answer _before_ checking the example command — the goal is recalling the flag, not copy-pasting.

### Exercise 1 — Basic discovery

**Task:** Find out which of the top 100 ports are open on `scanme.nmap.org`.

> [!question]- Example solution
> 
> ```bash
> nmap -F scanme.nmap.org
> ```
> 
> `-F` = fast mode, scans nmap's list of the 100 most common ports instead of the default 1000.

### Exercise 2 — Full port range

**Task:** Scan _all_ 65535 TCP ports on the same target and save the output to a file you could hand to someone else.

> [!question]- Example solution
> 
> ```bash
> nmap -p- -oN full_ports.txt scanme.nmap.org
> ```

### Exercise 3 — Service fingerprinting

**Task:** For whatever ports came back open in Exercise 2, identify the exact service and version running on each.

> [!question]- Example solution
> 
> ```bash
> nmap -p <ports_from_ex2> -sV scanme.nmap.org
> ```
> 
> Compare the reported version against a CVE search (cvedetails.com or Exploit-DB) — this is the mental habit that turns "open port" into "possible finding."

### Exercise 4 — NSE in practice

**Task:** Run the default safe script set against the host and read through what each script actually reports.

> [!question]- Example solution
> 
> ```bash
> nmap -sC -sV scanme.nmap.org
> ```
> 
> Pick one script from the output (e.g. `http-title` or `ssh-hostkey`) and look it up under `/usr/share/nmap/scripts/` — read the actual Lua source. Understanding one script deeply teaches you how to read all of them.

### Exercise 5 — Timing comparison (do this on your own lab VM, not scanme)

**Task:** Time a full port scan at `-T4` and again at `-T1`. Note the real difference.

> [!question]- Example solution
> 
> ```bash
> time nmap -p- -T4 target
> time nmap -p- -T1 target
> ```
> 
> Expect an order-of-magnitude difference. This is the trade-off you're making every time you pick a timing template on a real engagement.

### Exercise 6 — UDP vs TCP behavior

**Task:** On your Metasploitable2 lab VM, scan the top 20 UDP ports and compare the output style to a TCP scan of the same range.

> [!question]- Example solution
> 
> ```bash
> nmap -sU --top-ports 20 -sV target
> nmap --top-ports 20 -sV target
> ```
> 
> Notice how many UDP results come back `open|filtered` versus a clean `open`/`closed` split on TCP — that ambiguity is the theory point from [[#1.4 Why UDP scanning is fundamentally different]] showing up in real output.

### Exercise 7 — Evasion, conceptually

**Task:** Without running it against anything (decoys need caution even in labs), write out in your own words what this command is doing and why each flag is there:

```bash
nmap -D RND:10 -T1 -f --data-length 20 -Pn target
```

> [!question]- Example solution
> 
> - `-D RND:10` → generates 10 random decoy source addresses so the target's logs show 11 possible scanners, not one
> - `-T1` → slows probes down to avoid threshold-based IDS alerts
> - `-f` → fragments packets to dodge simple signature-matching filters
> - `--data-length 20` → pads packet size so it doesn't match a bare default-nmap signature
> - `-Pn` → skips the initial ping so no extra "is it alive" probe is sent before the real scan

---
