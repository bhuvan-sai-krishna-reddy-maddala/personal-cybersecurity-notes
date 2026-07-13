# Find Tool
``` find / -type f -name *.conf -user root -size +20k -newermt 2020-03-03 -exec ls -al {} \; 2>/dev/null ```

| **Option**            | **Description**                                                                                                                                                                                                                                                                    |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-type f`             | Hereby, we define the type of the searched object. In this case, '`f`' stands for '`file`'.                                                                                                                                                                                        |
| `-name *.conf`        | With '`-name`', we indicate the name of the file we are looking for. The asterisk (`*`) stands for 'all' files with the '`.conf`' extension.                                                                                                                                       |
| `-user root`          | This option filters all files whose owner is the root user.                                                                                                                                                                                                                        |
| `-size +20k`          | We can then filter all the located files and specify that we only want to see the files that are larger than 20 KiB.                                                                                                                                                               |
| `-newermt 2020-03-03` | With this option, we set the date. Only files newer than the specified date will be presented.                                                                                                                                                                                     |
| `-exec ls -al {} \;`  | This option executes the specified command, using the curly brackets as placeholders for each result. <br>The backslash escapes the next character from being interpreted by the shell because otherwise, the semicolon would terminate the command and not reach the redirection. |
| `2>/dev/null`         | This is a `STDERR` redirection to the '`null device`', which we will come back to in the next section. This redirection ensures that no errors are displayed in the terminal. This redirection must `not` be an option of the 'find' command.                                      |

### `-type`

|Flag|Matches|
|---|---|
|`-type f`|regular files|
|`-type d`|directories|
|`-type l`|symbolic links|
|`-type s`|sockets|
|`-type p`|pipes|
### `-newermt`

`newer` = more recent than `m` = modification time `t` = a specific timestamp/date string

So `-newermt 2020-03-03` means **"modified after March 3, 2020"**

Other variations:

| Flag       | Meaning                     |
| ---------- | --------------------------- |
| `-newermt` | modified after a date       |
| `-newerat` | accessed after a date       |
| `-newerct` | status changed after a date |

### `-exec ls -al {} \;`

The pattern is always:

```bash
-exec <command> {} \;
```

`{}` = placeholder for each matched file `\;` = "end of command, run once per file"

You can swap `ls -al` for **anything**:

```bash
-exec cat {} \;        # print contents of each file
-exec cp {} /tmp/ \;   # copy each file to /tmp
-exec grep -l "password" {} \;   # search inside each file
```

 **Bonus** — use `+` instead of `\;` to pass all results at once instead of one at a time. Faster.

 ```-exec ls -al {} + ```

# Locate Tool

### Basic Syntax

```bash
locate <pattern>
```

---

### Basic Usage

bash

```bash
locate passwd           # find everything with "passwd" in the path
locate "*.conf"         # find all .conf files
locate nginx            # find anything with nginx in the name or path
```

---
### Useful Flags

#### `-i` — case insensitive

```bash
locate -i "*.CONF"      # matches .conf .CONF .Conf
```

---

#### `-n` — limit results

```bash
locate -n 10 "*.conf"   # show only first 10 results
```

---
#### `-c` — count only

```bash
locate -c "*.conf"      # just shows the number — not the list
```

---
#### `-e` — verify files exist

```bash
locate -e "*.conf"      # skips files that were deleted since last updatedb
```

> This one matters. The database can be stale — a file might be in the database but already deleted. `-e` checks if it actually exists right now.

---
#### `-r` — use regex instead of pattern

```bash
locate -r "\.conf$"     # files ending in .conf using regex
```

---

### Combine Flags

```bash
locate -i -n 20 "*.conf"        # case insensitive, first 20 results
locate -e -n 5 "*.log"          # verified existing, first 5 results
```

---

### Pipe It

Since `locate` just outputs a list of paths, you can pipe it into anything:

```bash
locate "*.conf" | grep nginx          # only nginx related configs
locate "*.conf" | wc -l              # count all conf files
locate "*.log" | xargs ls -al        # list details of all log files
locate "*.conf" | xargs grep -l "password" 2>/dev/null   # search inside them
```

---

### `locate` vs `find` — When to Use Which

|Situation|Use|
|---|---|
|Need speed|`locate`|
|Need real-time accuracy|`find`|
|Just created a file|`find`|
|Hunting across whole system fast|`locate`|
|Need size/owner/date filters|`find`|
|Simple name search|`locate`|

---

### The Workflow Together

```bash
sudo updatedb                          # refresh the database
locate "*.conf" | grep -i "ssh"        # find ssh related configs fast
```

---

### One Thing to Remember

`locate` only knows about files that existed **at the time of the last `updatedb`**. Always keep that in mind during enumeration on a live system.