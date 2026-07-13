## Tools Overview

| Tool | Does |
|------|------|
| `more` | page through file, output stays on terminal |
| `less` | page through file, output disappears on quit |
| `head` | first N lines (default 10) |
| `tail` | last N lines (default 10), `-f` for live follow |
| `sort` | sort lines alphabetically or numerically |
| `grep` | filter lines by pattern |
| `cut` | extract specific columns by delimiter |
| `tr` | replace or delete characters |
| `column` | format output as neat table |
| `awk` | extract fields, text processing |
| `sed` | find and replace text |
| `wc` | count lines, words, characters |

---
## more
cat /etc/passwd | more

### Navigation
| Key | Action |
|-----|--------|
| Space | next page |
| Enter | next line |
| q | quit (output stays on terminal) |
| /pattern | search forward |

---
## less
less /etc/passwd

### Navigation
| Key | Action |
|-----|--------|
| Space | next page |
| b | previous page |
| g | go to top |
| G | go to bottom |
| q | quit (output disappears) |
| /pattern | search forward |
| ?pattern | search backward |
| n | next match |
| N | previous match |

> less is more — less does MORE than more

---
## head

```bash
head /etc/passwd              # first 10 lines (default)
head -n 5 /etc/passwd         # first 5 lines
head -n 20 /etc/passwd        # first 20 lines
```

---
## tail
```bash
tail /etc/passwd              # last 10 lines (default)
tail -n 5 /etc/passwd         # last 5 lines
tail -f /var/log/auth.log     # live follow — prints new lines as they appear
```

---
## sort
```bash
cat /etc/passwd | sort        # alphabetical
cat /etc/passwd | sort -r     # reverse
cat /etc/passwd | sort -n     # numerical
cat /etc/passwd | sort -u     # unique (removes duplicates)
```

---
## grep
```bash
cat /etc/passwd | grep "root"               # lines containing root
cat /etc/passwd | grep -v "nologin"         # exclude nologin lines
cat /etc/passwd | grep -v "false\|nologin"  # exclude false OR nologin
cat /etc/passwd | grep -i "root"            # case insensitive
cat /etc/passwd | grep -n "root"            # show line numbers
cat /etc/passwd | grep -E "false|nologin"   # extended regex (no backslash needed)
```

### Flags
| Flag | Meaning |
|------|---------|
| `-v` | invert — exclude matching lines |
| `-i` | case insensitive |
| `-n` | show line numbers |
| `-c` | count matching lines |
| `-l` | show filenames that match |
| `-r` | recursive search in directories |
| `-E` | extended regex |

### OR in grep
```bash
grep "a\|b"          # lines containing a OR b
grep -E "a|b"        # same — cleaner with -E
```

---

## cut
```bash
cut -d":" -f1 /etc/passwd         # field 1 (username)
cut -d":" -f1,3 /etc/passwd       # fields 1 and 3
cut -d":" -f1,3,7 /etc/passwd     # fields 1, 3, and 7
```

### Flags
| Flag | Meaning |
|------|---------|
| `-d` | delimiter character |
| `-f` | field number(s) to extract |

> Default delimiter is TAB — always specify -d for non-tab files
> Original file is NEVER modified — output only

```
/etc/passwd field map
root : x : 0 : 0 : root : /root : /bin/bash
 1     2   3   4    5      6        7
f1=username, f3=UID, f7=shell
```

---

## tr
```bash
cat /etc/passwd | tr ":" " "      # replace : with space
cat /etc/passwd | tr ":" ","      # replace : with comma
cat /etc/passwd | tr -d ":"       # delete colons entirely
cat /etc/passwd | tr "a-z" "A-Z"  # lowercase to uppercase
```

> Original file is NEVER modified — output only

---

## column

```bash
cat /etc/passwd | tr ":" " " | column -t    # format as aligned table
```

---

## awk
```bash
cat /etc/passwd | tr ":" " " | awk '{print $1}'         # first field
cat /etc/passwd | tr ":" " " | awk '{print $1, $NF}'    # first and last field
cat /etc/passwd | tr ":" " " | awk '{print $1, $3}'     # first and third field

# $1 = first field
# $NF = last field (NF = Number of Fields)
# fields split by whitespace by default
```

### Find field numbers when unsure

```bash
grep "username" /etc/passwd | awk -F":" '{for(i=1;i<=NF;i++) print i, $i}'
```

---

## sed
```bash
cat /etc/passwd | sed 's/bin/HTB/g'     # replace every "bin" with "HTB"
cat /etc/passwd | sed 's/:/,/g'         # replace every : with ,

# s = substitute
# g = global (all occurrences, not just first)
# syntax: sed 's/old/new/g'
```

---

## wc
```bash
cat /etc/passwd | wc -l        # count lines
cat /etc/passwd | wc -w        # count words
cat /etc/passwd | wc -c        # count characters
```

---

``` bash
Pipeline Thinking
Always build left to right:
read → filter → extract → transform → format → count

cat /etc/passwd | grep -v "nologin\|false" | cut -d":" -f1,3,7 | tr ":" "," | sort | wc -l
```

---

## The 8 Exercises (reference answers)
```bash
# 1. Line with cry0l1t3
cat /etc/passwd | grep "cry0l1t3"

# 2. All usernames
cat /etc/passwd | cut -d":" -f1

# 3. cry0l1t3 username and UID
cat /etc/passwd | grep "cry0l1t3" | cut -d":" -f1,3

# 4. cry0l1t3 username and UID separated by comma
cat /etc/passwd | grep "cry0l1t3" | cut -d":" -f1,3 | tr ":" ","

# 5. cry0l1t3 username, UID, shell separated by comma
cat /etc/passwd | grep "cry0l1t3" | cut -d":" -f1,3,7 | tr ":" ","

# 6. All usernames, UID, shell separated by comma
cat /etc/passwd | cut -d":" -f1,3,7 | tr ":" ","

# 7. All usernames, UID, shell separated by comma — exclude nologin and false
cat /etc/passwd | grep -v "nologin\|false" | cut -d":" -f1,3,7 | tr ":" ","

# 8. Same as 7 — exclude nologin only, count lines
cat /etc/passwd | grep -v "nologin" | cut -d":" -f1,3,7 | tr ":" "," | wc -l
```

Previous:
[[Day -3 File Descriptors and Redirections]]

Next:
[[Day -5 RegEX]]