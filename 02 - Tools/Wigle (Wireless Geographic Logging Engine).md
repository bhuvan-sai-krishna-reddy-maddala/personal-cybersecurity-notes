## What it is
>A crowd sourced database of wireless networks (WiFi, Bluetooth, 
cell towers) mapped to physical locations worldwide.

>Wardriving enthusiasts drive around scanning and submitting 
network data — Wigle aggregates it all.

## Why it matters in security

>If you find an SSID (WiFi network name) or BSSID (router MAC 
address) during OSINT, Wigle can tell you the physical location 
of that network.

>Works in reverse too — give it coordinates, get nearby SSIDs.

## Access

- Website: wigle.net
- Requires free registration (note: some IP ranges blocked 
  from registering — use mobile hotspot if blocked)
- Has a mobile app for wardriving

## How to use for OSINT

1. Get an SSID or BSSID from your target (via EXIF, photos, 
   social media check-ins, etc.)
2. Go to wigle.net → Search → Basic Search
3. Enter the SSID or BSSID
4. Map shows physical location of that network
5. Cross-reference with other intel

## Alternatives if Wigle is blocked

- radiocells.org — open database, no registration
- mylnikov.org — geolocation API, no registration
- openwifi.su — alternative open WiFi database
- Google dorking: "(SSID name)" + location keywords

## Real example — OhSINT room

- Found GPS coordinates in EXIF data
- Used those to narrow the area
- Wigle SSID search confirmed exact location
- Completed the location-based flag

## Key terms

- SSID: the name of a WiFi network (what you see when 
  connecting)
- BSSID: the MAC address of the router — unique identifier
- Wardriving: driving around scanning and mapping wireless 
  networks

## Gotchas
- Coverage varies by country — major cities well covered, 
  rural areas sparse
- Data can be outdated — networks move or get renamed
- Registration blocked on VPNs and some ISP ranges

## Connected topics
- [[Exif Tool]]
- [[OSINT Methodology]]
- [[OhSINT — TryHackMe Writeup]]