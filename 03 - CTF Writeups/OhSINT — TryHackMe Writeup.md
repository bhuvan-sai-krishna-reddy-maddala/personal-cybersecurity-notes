## Room info
- Platform: TryHackMe
- Type: OSINT challenge
- Difficulty: Easy
- Date completed: June 2025
- Badge: Earned ✓

## What the room teaches
How to extract intelligence from a single image using only 
open-source tools and public information.

## Tools used
- ExifTool (EXIF metadata extraction)
- Reverse image search
- Username OSINT across platforms
- Wigle (WiFi geolocation)

## Methodology used
1. Downloaded the challenge image
2. Ran ExifTool → extracted metadata
3. Found username embedded in metadata
4. Searched username across platforms
5. Found social media profile with more details
6. Used GPS data + Wigle for location flag

## Flags
- [x] Flag 1 — found via EXIF metadata
- [x] Flag 2 — found via social media profile
- [x] Flag 3 — found via reverse image search
- [x] Flag 4 — found via profile details
- [x] Flag 5 — found via Wigle (SSID lookup)
- [x] Remaining flags — completed

## Key lesson
One image file contains enough information to build a 
significant profile of a person. Most people have no idea 
how much metadata their photos expose.

## What to learn next
- More OSINT tools: theHarvester, Maltego, Shodan
- Sock puppet accounts for covert OSINT
- OSINT framework: osintframework.com

## Connected topics
- [[OSINT Methodology]]
- [[Exif Tool]]
- [[Wigle (Wireless Geographic Logging Engine)]]
