---

## tags: [web-security, access-control, owasp-a01, apprentice, portswigger] owasp: A01 - Broken Access Control cwe: CWE-22 status: labs-complete date: 2026-10-06
---
---
owasp: A01 - Broken Access Control
cwe: CWE-22
status: labs-complete
date: 2026-10-06

---
## What it is

A path traversal (directory traversal) bug lets an attacker read — and sometimes write — arbitrary files on the server's filesystem, by manipulating a file path parameter the application trusts.

**Core pattern:**

```
server builds path = base_folder + user_input
no check that the result stays inside base_folder
```

Example:

```
base:    /var/www/images/
input:   ../../../etc/passwd
result:  /var/www/images/../../../etc/passwd  →  /etc/passwd
```

## Root cause

The application takes user-supplied input (a filename, usually) and concatenates it directly into a filesystem path, with no validation or canonicalization. The operating system — not the app — resolves `../` sequences, climbing one directory per `../`. Extra `../` beyond the root are harmless, since `/` can't go higher, which is why overshooting (`../../../../../../etc/passwd`) is a common attacker shortcut when the exact depth is unknown.

`../` is the same concept as `cd ..` in a shell — "go to parent directory" — just written inline in a path string instead of as a terminal command.

## Why it matters even as "just reading files"

- App source code → reveals other bugs, hidden endpoints, logic flaws
- Config files / `.env` → DB credentials, API keys, cloud secrets
- SSH keys / tokens → pivot to other systems
- Logs → other users' data/sessions

If write access is also possible, this escalates to modifying app behavior or full server takeover.

## Where to look for it in real apps

Any parameter that smells like a filename or path:

```
?filename=   ?file=   ?img=   ?doc=   ?path=   ?template=   ?page=
```

Common in: image/avatar loaders, download/export endpoints, template or language file loaders.

**Key gotcha:** the vulnerable parameter is often NOT on the page you're viewing. A product page's `?productId=` is a dead end — the real bug lives in the _resource_ that page loads (e.g. the `<img>` tag's own request). Open the image/resource directly (new tab, or find it in Burp history) to find the real endpoint and parameter.

## OS fingerprinting (which payload to use)

|Clue|Suggests|
|---|---|
|`Server: Apache` / `nginx` header|Linux|
|`Server: Microsoft-IIS`|Windows|
|`.aspx` extensions|Windows|
|Error text with `C:\...` or `/var/...`|Direct answer|

```
Linux:    ../../../../etc/passwd
Windows:  ..\..\..\..\windows\win.ini   (Windows accepts ../ too)
```

When unsure, just try both — costs nothing.

## Defence / fix (strongest → weakest)

1. Don't accept filenames from the user at all — use an ID, look up the real filename server-side
2. Whitelist allowed filenames
3. Canonicalize the resolved path, verify it still starts with the intended base folder, reject if not
4. Least privilege — run the app as a user that can't read sensitive files anyway

Weak/wrong fix: stripping `../` from input (bypassable — covered in later Academy content on filter evasion).

## Classification

- **OWASP Top 10 (2021):** A01 Broken Access Control
- **CWE:** CWE-22 — Improper Limitation of a Pathname to a Restricted Directory

---

## Lab: File path traversal, simple case

**Status:** ✅ Solved **Goal:** Retrieve contents of `/etc/passwd`

**How I found the vulnerable endpoint:** The product page's `?productId=` parameter was a dead end. The real bug was in the image-serving request, which only shows up when the `<img>` tag's own request is made. Found it by opening the image in a new tab to reveal its real URL (`?filename=`).

**Payload:**

```
../../../etc/passwd
```

Resolved: `/var/www/images/` + `../../../etc/passwd` → `/etc/passwd`

**Tooling note:** Didn't see the request in Burp HTTP History at first — Burp's "Hide CSS, images and other binary content" filter was ON by default and was hiding exactly the image requests needed. Turn this filter off before every session.

**Gotcha to remember:** Vulnerable parameters are often on resource requests triggered BY a page, not the page's own query string. Check what a page loads, not just its own URL.

---

## Recon habits reinforced this section

- Open resource requests (images, downloads) directly to find their real URL/params
- Turn off Burp's "hide binary content" filter at the start of every session
- Practice editing requests in Repeater, not just the address bar — needed for POST bodies/cookies in upcoming sections

## Links

- [PortSwigger: File path traversal](https://portswigger.net/web-security/file-path-traversal)
- [OWASP: Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [CWE-22](https://cwe.mitre.org/data/definitions/22.html)