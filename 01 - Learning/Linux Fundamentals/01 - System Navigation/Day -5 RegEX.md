## What is Regex
A pattern you define to match text — instead of searching for an exact word,
you describe the SHAPE of what you're looking for.

## Two Modes in grep
| Mode | Flag | Supports |
|------|------|---------|
| Basic Regex (BRE) | default | `.` `^` `$` `*` `[]` `\` |
| Extended Regex (ERE) | `-E` | everything above + `+` `?` `()` `{}` `\|` |

> Rule — whenever you use + ? () {} | always add -E

> Easier: just always use -E and never think about it

---

## Metacharacters

| Metacharacter | Meaning | Needs -E |
|---------------|---------|----------|
| `.` | any single character | No |
| `^` | start of line | No |
| `$` | end of line | No |
| `*` | zero or more of previous | No |
| `+` | one or more of previous | Yes |
| `?` | zero or one of previous | Yes |
| `\` | escape next character | No |
| `[]` | character class | No |
| `[^]` | negated character class | No |
| `()` | grouping | Yes |
| `{}` | quantifier | Yes |
| `\|` | OR | Yes |
| `.*` | anything in between | No |

---

## Metacharacter Examples

### `.` — any single character
```bash
grep "r..t" file        # matches root, rest, rant, r00t
```

### `^` — start of line
```bash
grep "^root" file       # lines starting with root
grep "^#" file          # lines starting with #
grep "^[^#]" file       # lines NOT starting with #
```

### `$` — end of line
```bash
grep "bash$" file       # lines ending with bash
grep "yes$" file        # lines ending with yes
grep "^$" file          # empty lines
```

### `*` — zero or more
```bash
grep "ro*t" file        # matches rt, rot, root, rooot
```

### `+` — one or more (needs -E)
```bash
grep -E "ro+t" file     # matches rot, root, rooot (NOT rt)
grep -E "[0-9]+" file   # lines with at least one digit
```

### `?` — zero or one (needs -E)
```bash
grep -E "ro?t" file     # matches rt or rot only
grep -E "https?" file   # matches http or https
```

### `\` — escape
```bash
grep "192\.168\.1\.1" file      # literal dots not any character
grep "1\+1" file                # literal 1+1
```

### `[]` — character class
```bash
grep "[aeiou]" file             # any vowel
grep "[0-9]" file               # any digit
grep "[a-z]" file               # any lowercase letter
grep "[a-zA-Z0-9]" file         # any letter or digit
```

### `[^]` — negated character class
```bash
grep "[^0-9]" file              # any character NOT a digit
grep -v "#" file                # lines NOT containing # (simpler with -v)
```

> IMPORTANT: [] is ALWAYS a character class — never a word
> [Key] matches K, e, or y — NOT the word Key
> [yes] matches y, e, or s — NOT the word yes
> To match a word just type it: grep "Key" file

### `()` — grouping (needs -E)
```bash
grep -E "(my|false)" file               # my OR false as a group
grep -E "(nologin|false)$" file         # ending with nologin or false
```

### `{}` — quantifiers (needs -E)
```bash
grep -E "[0-9]{3}" file         # exactly 3 digits
grep -E "[0-9]{1,3}" file       # 1 to 3 digits
grep -E "[0-9]{3,}" file        # 3 or more digits
```

### `|` — OR (needs -E)
```bash
grep -E "root|bash" file                # root OR bash
grep -E "nologin|false" file            # nologin OR false
```

### `.*` — anything in between
```bash
grep -E "Permit.*yes" file              # Permit then anything then yes
grep -E "^root.*bash$" file             # starts with root ends with bash
```

---

```bash
## Common Patterns

# IP address
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" file

# Email (basic)
grep -E "[a-zA-Z0-9]+@[a-zA-Z0-9]+\.[a-zA-Z]+" file

# http or https URLs
grep -E "https?://[a-zA-Z0-9]+" file

# Lines starting with a letter
grep -E "^[a-zA-Z]" file

# Lines with at least one digit
grep -E "[0-9]+" file
```

---

## The 6 Exercises (sshd_config)

```bash
# 1. Lines not containing #
grep -v "#" sshd_config

# 2. Lines containing a word starting with Permit
grep "Permit" sshd_config

# 3. Lines containing a word ending with Authentication
grep "Authentication$" sshd_config

# 4. Lines containing the word Key
grep "Key" sshd_config

# 5. Lines beginning with Password and containing yes
grep -E "^Password.*yes" sshd_config

# 6. Lines ending with yes
grep "yes$" sshd_config
```

> [[Day -5 RegEX]] and [[Day -4 Filtering Contents]] are often used together to get the best results


Previous:
[[Day -4 Filtering Contents]]

Next:
[[Day -6 Permission Management]]