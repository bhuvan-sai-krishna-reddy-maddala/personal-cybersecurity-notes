## What it is 
A tool that reads, writes, and edits metadata (EXIF data) embedded inside files — images, PDFs, audio, video. Metadata is hidden information baked into a file that most people don't know exists. 
## Why it matters in security 
Photos taken on phones embed GPS coordinates, device model, timestamp, and software used — all invisible to the naked eye. In OSINT this is often your first clue about a target's location, device, or identity. 
## Install 
``` bash
sudo apt install exiftool 
``` 
## Core commands 
```bash 
# Read all metadata from a file 
exiftool filename.jpg 

# Read only GPS data 
exiftool -GPS* filename.jpg 

# Read specific tag 
exiftool -Artist filename.jpg 

# Remove all metadata from a file (useful for your own privacy) 
exiftool -all= filename.jpg 

# Output to text file 
exiftool filename.jpg > output.txt 
``` 
## Real example — OhSINT room 
- Downloaded the challenge image 
- Ran exiftool on it 
- Found: GPS coordinates, device info, and a username embedded in the metadata 
- That one command unlocked every other flag in the room 
- 
## Key metadata fields to look for 
- GPSLatitude / GPSLongitude → physical location 
- Artist / Author → username or real name 
- CreateDate → when the file was made 
- Make / Model → device used 
- Software → editing software (can reveal workflow)
## Online alternative (no install needed)
- [exifdata.com][https://www.exifdata.com/]
- Just upload the image, same output 
## Gotchas 
- Some platforms (Twitter, Instagram, WhatsApp) strip EXIF automatically on upload 
- Absence of EXIF data is itself useful intel — means it was deliberately removed or uploaded via a stripping platform 
- Always run exiftool before any other analysis on an image 
## Connected topics 
- [[OhSINT — TryHackMe Writeup]] 
- [[Wigle (Wireless Geographic Logging Engine)]]
- [[OSINT Methodology]]
