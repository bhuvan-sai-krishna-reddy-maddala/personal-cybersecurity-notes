---
tags:
  - otw
  - bandit
  - linux
  - wargames
  - cybersecurity
  - writeup
platform: OverTheWire - Bandit
levels: 0-15
status: in-progress
---

> **Note on passwords:** Per OverTheWire's rules (no posting spoilers), all passwords in this note are redacted as `*`. Actual passwords are stored **locally only** in a separate, non-public file — never pushed to GitHub.

## Connection basics

```bash
ssh -p 2220 banditN@bandit.labs.overthewire.org
```

- Port is **2220**, not the default 22 (keeps wargame SSH traffic separate from real SSH).
- First connection to any new host asks to confirm the key fingerprint — type `yes`.
- **Never run the `ssh` command for the *next* level while still logged into the *current* level's shell.** It resolves to `localhost` and gets blocked. Always `exit` back to your own machine first, then SSH in fresh.
---
## Level 0 → 1

**Goal:** Just connect via SSH and read a file.

**Approach:**

```bash
ls

cat readme

```

**Concept:** `cat` ("concatenate") dumps a file's contents to the terminal — simplest way to read small text files.

**Password:** `************************`

---
## Level 1 → 2

  **Goal:** Password stored in a file literally named `-`.

**Block hit:** `cat -` doesn't read the file — a leading `-` is interpreted as a command flag, and `cat -` specifically means "read from stdin," which just hangs waiting for input.

**Fix — two ways to force the shell to treat `-` as a filename, not a flag:**

```bash
cat ./-       # ./ prefix removes the ambiguity

cat -- -      # -- tells the command "stop parsing flags, rest are literal args"

```

**Gotcha discovered:** even `cat -- -` can still hang on some `cat` implementations, because a lone `-` after `--` is *still* conventionally read as "stdin" by some tools. `./-` proved to be the more reliable fix in practice.

**Password:** `************************`

---
## Level 2 → 3

**Goal:** Password stored in a file named `--spaces in this filename--`.

**Block hit:** Unquoted, the shell splits the filename on every space into separate arguments, and `--spaces` also gets misread as a flag.

**Fix:**

```bash
cat -- "--spaces in this filename--"

```

- Quotes → treat the whole string (spaces included) as **one** argument.
- `--` → stop treating anything after it as a flag, even the leading `--`.

>**Note:** the system's `cat` is a modern Rust-based rewrite (uutils coreutils) — stricter than GNU `cat`, it flatly refuses `-`-prefixed args even quoted, unless `--` is present. Confirmed by the tool's own error message and suggested fix.

**Rule of thumb locked in:** quoting handles spaces; `./` or `--` handles leading dashes. Two different shell parsing problems, two different fixes.

**Password:** `************************`

---
## Level 3 → 4

**Goal:** Password in a *hidden* file inside `inhere/`.

**Approach:**

```bash
cd inhere

ls -la

cat "...Hiding-From-You"

```

**Concept:** Files starting with `.` are hidden from plain `ls` by convention (not a security feature). `-a` shows all files including hidden ones; `-l` gives the detailed long listing (permissions, owner, group, size, date).

**Also covered:** reading `-rw-r-----` style permission strings and matching them against file ownership (`user:group`) to reason about who can read what — first real look at Linux permission bits, relevant later for privesc work.

**Password:** `************************`

---
## Level 4 → 5

**Goal:** Find the one human-readable file among many decoys in `inhere/`, identified only by content, not name.

**Tool introduced: `file`** — inspects actual file content/magic bytes to report true file type, regardless of filename or extension.

**Approach:**

```bash
file ./*

cat -- "-file07"

```

**Block hit:** `cat "-file07"` (quoted only) still failed — quoting alone doesn't stop a leading `-` from being parsed as a flag; quoting only protects spaces/special characters. Needed `./-file07` or `cat -- "-file07"` specifically.


**Password:** `************************`

---
## Level 5 → 6

**Goal:** Find a file under `inhere/` (20+ nested directories) matching exact size (1033 bytes) and not executable.

**Tool introduced: `find`** — recursive search filtered by file properties (type, size, permissions, ownership, etc.), unlike `ls` which just lists.

**Approach — built incrementally, testing conditions one at a time:**

```bash
find . -type f -size 1033c

find . -type f -size 1033c ! -executable

```

- `-type f` → regular files only
- `-size 1033c` → exact byte size (`c` = bytes, not 512-byte blocks)
- `! -executable` → negation; excludes executable files

**Good habit demonstrated:** building the find command up piece by piece and sanity-checking each addition rather than guessing the whole thing at once.

**Password:** `************************`

---
## Level 6 → 7

**Goal:** File somewhere on the *entire server* (not just home dir), owned by a specific user+group, exact size.

**Approach:**

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null

```

- Searching from `/` (filesystem root) instead of home dir, since the level goal says "somewhere on the server."
- `2>/dev/null` → redirects **stderr** (permission-denied spam from searching directories you can't access) to a null device, keeping output clean.

>**Note:** found the match even before adding the stderr redirect, buried in the noise — good practice reading past clutter to find the actual signal.

**Password:** `************************`

---
## Level 7 → 8

**Goal:** Password on a specific line in `data.txt`, next to a known keyword.

**Approach:**

```bash
grep "millionth" data.txt

```

**Correction made:** originally ran `cat data.txt | grep "..."` — works, but unnecessary use of `cat` ("useless use of cat"). `grep` takes a filename directly.

**Password:** `************************`

---
## Level 8 → 9

**Goal:** Find the one line in `data.txt` that occurs exactly once, among many duplicated lines.

**Tools: `sort` + `uniq`**

```bash
sort data.txt | uniq -u

```

- `sort` groups identical lines adjacently (`uniq` only catches *adjacent* duplicates, so sorting first is mandatory).
- `uniq -u` prints only lines with **no** duplicates.

**Side detour — `man -k` / `apropos` limitations:**

> 	Tried `man -k "occur only once"` and `man -k "only once"` — both failed. `apropos`/`man -k` does a **literal string search over the one-line NAME summary only**, not full-text or synonym-aware search. Even the "correct" jargon guesses (`unique`, `duplicate`) failed to surface `uniq`, because its actual summary uses the word **"repeated"** instead. Lesson: `apropos` is brittle; for "what tool does X" questions, a web search or asking someone/an LLM directly is far more reliable than guessing exact synonyms against a strict-match local tool.


**Password:** `************************`

---
## Level 9 → 10

**Goal:** Extract a password from a binary file, preceded by several `=` characters.

**Tool introduced: `strings`** — extracts printable/human-readable character sequences from any file, binary or not. Core tool in malware analysis, forensics, reverse engineering.

**Approach:**

```bash
strings data.txt | grep "="

```

Output revealed a fragmented sentence across several lines (Bandit hides "the / password / is / [password]" split up across separate matches) — had to read across multiple grep hits to reconstruct the actual sentence and find the right one.

**Password:** `*`

---
## Level 10 → 11

**Goal:** Decode Base64 data in `data.txt`.

**Concept:** Base64 is an **encoding**, not encryption — represents binary data using 64 printable ASCII characters so it can travel safely through text-only channels. Anyone can decode it instantly; provides zero confidentiality. Extremely common in real-world work (JWTs, embedded images, API payloads).

**Approach:**

```bash
base64 -d data.txt

```

**Side note:** running bare `base64` with no file argument hangs, same reason as `cat -` earlier — defaults to reading from stdin when no file is given.

**Password:** `************************`

---
## Level 11 → 12

**Goal:** Decode ROT13-rotated text in `data.txt`.

**Concept:** ROT13 shifts each letter 13 places through the alphabet; applying it twice returns the original (self-inverse, since 26 letters / 2 = 13). No real security value — historically used to obscure spoilers, not for secrecy.

**Tool: `tr`** — one-to-one character substitution based on two character-set arguments.

**Approach:**

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'

```

Source set `A-Za-z` mapped position-by-position onto the destination set, which is the alphabet rotated 13 places (uppercase and lowercase handled separately within the same command).

**Password:** `************************`

---
## Level 12 → 13

**Goal:** Reverse a hexdump of a file that's been compressed multiple times, in nested layers, and extract the final password.

**Concept — hexdump:** a text representation of raw binary data, showing each byte as a 2-character hex value, alongside an offset counter and an ASCII sidebar (printable bytes shown as characters, others as `.`). Exists because raw binary can't be safely displayed/transmitted as plain text.

**Workspace setup (as instructed by the level, to avoid clobbering shared space):**

```bash
mktemp -d

cd <printed-path>

cp ~/data.txt .

```

`~` is a shell shortcut for the current user's home directory (`/home/bandit12` in this case).

**Reversing the hexdump back to binary:**

```bash
xxd -r data.txt > data

```

**Then repeated this loop multiple times, peeling one compression/archive layer at a time:**

```bash

file data              # identify current layer's format

mv data data.<ext>     # rename with correct extension

gunzip / bunzip2 / tar xf   # decompress or extract, matching the format

ls                      # see what new file appeared

# repeat file → identify → decompress on the new file

```

**Actual chain encountered (8 layers deep):**

gzip → bzip2 → gzip → tar → tar → bzip2 → tar → gzip → **plain ASCII text**


**Mistakes made and corrected along the way:**

- Used `mv data data.bz2 | bunzip2 data.bz2` (pipe) — this "worked" by accident, not by design: `mv` produces no output, so the pipe carried nothing; the two commands just happened to run in sequence fast enough. Corrected to the semantically right operator:

  ```bash
  mv data data.gz && gunzip data.gz

  ```

 >	`&&` = "run the second command only if the first succeeded" — the actual intended logic, vs `|` which is specifically for passing data between programs.

- Typo'd `bunzip` instead of `bunzip2` — shell's command-not-found suggestions caught it immediately, fixed on retry.

- `mv` failed once on a re-run because the file had already been renamed in a prior successful step — caught via `ls`, adjusted command target.

**Confirmation trick used:** recognized the gzip magic bytes (`1f 8b`) directly in the very first hexdump output before ever running `file` — a nice sanity check that decoding was working correctly from the start.

**Password:** `************************`

---
## Level 13 → 14

**Goal:** No password given — instead, given a private SSH key (`sshkey.private`) to log in as `bandit14`. Read `/etc/bandit_pass/bandit14` (a **file**, not a directory).

**Concept — SSH key-based authentication:** instead of a password, a cryptographic key pair proves identity. This is the professional-standard method for SSH in real environments, not just a wargame quirk.

**Important constraint (per level HINT):** cannot chain-login from bandit13's session directly to bandit14 on localhost — blocked deliberately. Had to move the key out to the actual client machine and SSH in fresh from there.

**Approach:**

1. Copied the private key contents out (`cat sshkey.private`), saved locally as `bandit13_14.key` on the client machine.

2. Fixed permissions — SSH refuses to use a key that's too open:

```bash
   chmod 600 bandit13_14.key

```

3. Logged in using the key instead of a password:

```bash
   ssh -i bandit13_14.key -p 2220 bandit14@bandit.labs.overthewire.org

```

4. Read the password file directly:

```bash
   cat /etc/bandit_pass/bandit14

```

**Password:** `************************`

---
## Level 14 → 15

**Goal:** Submit the current password to a service listening on **port 30000** on localhost (plain, unencrypted).

**Concept:** first hands-on manual interaction with a raw TCP network service — same underlying mechanic as writing a port scanner or basic client/server script later (`socket.connect()` + send + receive, just done by hand here).

**Tool: `nc` (netcat)** — general-purpose tool for opening raw TCP/UDP connections, sending/receiving data, port scanning, banner grabbing.

**Approach (run from inside the bandit14 SSH session, since "localhost" = the Bandit server itself):**

```bash
nc localhost 30000

<paste current password, hit Enter>

```

Service replied `Correct!` followed directly by the next password.

**Password:** `************************`

---
## Level 15 → 16

**Goal:** Same idea as Level 14, but the service on **port 30001** requires SSL/TLS encryption.

**Concept:** plain `nc` sends unencrypted data and won't complete a TLS handshake — same practical distinction as HTTP vs HTTPS. Needed a TLS-aware client instead.

**Tool: `openssl s_client`** — lets you manually open and interact with a TLS-secured connection; useful generally for testing/debugging TLS services and inspecting certificates.

  **Approach:**

```bash
openssl s_client -connect localhost:30001

<paste current password, hit Enter>

```

Output showed certificate/handshake details first (normal), then `Correct!` and the next password, followed by `closed` (server closing the TLS session after responding — expected, not an error).

**Mistake caught mid-level:** an incorrect/fabricated password was tried first and failed — caught immediately by re-checking against the actual previously-logged Level 15 password rather than trusting it blindly, then retried successfully. Good general practice: verify credentials against your own source of truth rather than assuming any single instruction is correct.

**Password:** `************************`

  

---

  

## Running list of core skills unlocked (Levels 0–15)

  

| Skill | Levels |

|---|---|

| SSH basics, reading files (`ls`, `cat`) | 0–1 |

| Shell argument parsing gotchas (`-`, spaces, `--`, `./`) | 1–2, 4 |

| Hidden files, long listing, permission bits | 3 |

| File-type identification by content (`file`) | 4, 12 |

| Recursive filtered search (`find`) | 5–6 |

| Redirecting stderr (`2>/dev/null`) | 6 |

| Pattern matching (`grep`) | 7 |

| Sorting + uniqueness (`sort`, `uniq -u`) | 8 |

| Extracting text from binaries (`strings`) | 9 |

| Base64 decoding | 10 |

| Character substitution / ROT13 (`tr`) | 11 |

| Hexdump reversal, multi-layer decompression (`xxd`, `gunzip`, `bunzip2`, `tar`) | 12 |

| SSH key-based auth, file permissions on keys (`chmod 600`) | 13 |

| Raw TCP interaction (`nc`) | 14 |

| TLS/SSL interaction (`openssl s_client`) | 15 |

| Command chaining semantics: `\|` (pipe) vs `&&` (conditional chain) | 12 |

| `apropos`/`man -k` limitations vs web search / LLM lookup | 8 |

  

---

  

## Open follow-ups for later chats (not roadmap-tracking)

  

- Deeper dive on Linux permission bits (rwx, owner/group/other) — came up in Levels 3 and 13, worth a dedicated concept session.

- `tar` flags beyond `xf` (creating archives, `-c`, `-v`, `-z` combined flags) — only extraction was needed so far.

- `openssl s_client` deeper usage (certificate inspection, CONNECTED COMMANDS like renegotiate) — only the minimal interaction was needed for Level 15.


[[Day -20 System Logs and Monitoring]]

