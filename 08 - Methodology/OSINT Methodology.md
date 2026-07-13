## What is OSINT
Open Source Intelligence — gathering information about a target 
using only publicly available sources.
No hacking. No unauthorized access. Just finding what's already 
exposed.

## Why it matters
Every professional engagement starts with OSINT.
You cannot attack or defend what you don't understand.
Recon → everything else.

## The methodology (in order)

### 1. Start with what you have
- A name, username, email, image, domain, IP — anything
- Don't jump to tools yet. Think first.
- What do you know? What do you want to find?

### 2. Extract everything from the artifact
- Image → run ExifTool immediately
- Document → check metadata
- Email → check breach databases
- Domain → WHOIS, DNS records

### 3. Expand from what you find
- Username found → search same username on every platform
  (Twitter, GitHub, Reddit, Instagram, LinkedIn)
- Location found → cross-reference with Wigle, Google Maps, 
  Street View
- Real name found → LinkedIn, Facebook, public records

### 4. Cross-reference everything
- One source can lie or be outdated
- Two independent sources confirming = high confidence
- Build a picture from multiple angles

### 5. Document as you go
- Screenshot everything
- Note which tool found what
- You will forget details — write them down immediately

## Tools used in OSINT
| Tool                 | Purpose                          |
| -------------------- | -------------------------------- |
| ExifTool             | Extract file metadata            |
| Wigle                | WiFi network geolocation         |
| Reverse image search | Find image source and copies     |
| WhatsMyName          | Username search across platforms |
| WHOIS                | Domain registration info         |
| Shodan               | Internet-connected device search |
| theHarvester         | Email and subdomain enumeration  |

## Platforms to check for usernames
- GitHub
- Twitter/X
- Reddit
- Instagram
- LinkedIn
- HackerOne (for security researchers)

## Key principle
**Absence of information is also information.**
If someone has scrubbed their digital footprint — that tells 
you something too.

## Real example — OhSINT room
1. Had: one image file
2. ExifTool → found username, GPS coordinates, device info
3. Username → found social media profile
4. Social media → found more personal details
5. GPS + Wigle → confirmed physical location
6. Built complete profile from one image

## Connected topics
- [[Exif Tool]]
- [[Wigle (Wireless Geographic Logging Engine)]]
- [[OhSINT — TryHackMe Writeup]]