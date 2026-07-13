>A file descriptor (`FD`) in Unix/Linux operating systems is a reference, maintained by the kernel, that allows the system to manage Input/Output (`I/O`) operations. It acts as a unique identifier for an open file, socket, or any other I/O resource. In Windows-based operating systems, this is known as a file handle. Essentially, the file descriptor is the system's way of keeping track of active `I/O` connections, such as reading from or writing to a file.
>
>Think of it as a ticket number you get when checking in your coat at a coatroom. The ticket (file descriptor) represents your connection to your coat (file or resource), and whenever you need to retrieve your coat (perform I/O), you present the ticket to the attendant (operating system) who knows exactly where your coat is stored (which resource the file descriptor refers to). Without the ticket, you'd have no way of efficiently accessing your coat among the many others stored, just as without a file descriptor, the operating system wouldn't know which resource to interact with. You will soon see why file descriptors are so important and why understanding them is crucial as we dive into the upcoming examples.

By default, the first three file descriptors in Linux are:

| **1. Data Stream for Input**                                      | - `STDIN – 0`  | Default destination |
| ----------------------------------------------------------------- | -------------- | ------------------- |
| **2. Data Stream for Output**                                     | - `STDOUT – 1` | keyboard            |
| **3. Data Stream for Output that relates to an error occurring.** | - `STDERR – 2` | terminal screen     |

- `stdin`, `stdout`, `stderr` are just **human-friendly names** for FD 0, 1, 2
- When you open a file, kernel assigns the **lowest free number** (starts at 3)
- Each process has its **own FD table** — isolated from other processes

## Three Kernel Layers

FD Table (per process) 
	└── Open File Table (kernel-wide) 
		- tracks read/write position 
		- tracks access mode (r/w) 
	└── Inode Table (actual file on disk)

## Redirection Symbols

| Symbol    | Meaning                              |
|-----------|--------------------------------------|
| `>`       | redirect stdout to file (overwrites) |
| `>>`      | append stdout to file                |
| `2>`      | redirect stderr to file              |
| `2>/dev/null` | discard errors                   |
| `<`       | feed file as stdin                   |
| `<<`      | here document (multiline input)      |
| `\|`      | pipe stdout to next command          |
| `2>&1`    | merge stderr into stdout             |
| `1>&2`    | merge stdout into stderr             |

```bash

# Discard errors
find / -name "*.conf" 2>/dev/null

# Save stdout, discard errors
find / -name "*.conf" 2>/dev/null > results.txt

# Save stdout and stderr to separate files
find /etc/ -name shadow 2> stderr.txt 1> stdout.txt

# Save both to same file
find / -name "*.conf" > all.txt 2>&1

# Append to existing file
find /etc/ -name passwd >> stdout.txt 2>/dev/null

# Feed file as stdin
cat < stdout.txt

# Pipe chain
find /etc/ -name "*.conf" 2>/dev/null | grep systemd | wc -l

# Here document
cat << EOF > stream.txt
Hack The Box
EOF
```

## How Redirection Actually Works
Shell does this BEFORE forking the child process:
1. Opens destination file → gets FD 3
2. Copies FD 3 onto FD 1 (stdout)
3. Closes FD 3
4. Forks child — child inherits modified FD table
5. Child writes to FD 1 normally — has no idea redirection happened

## Why Order Matters in `2>&1`
## CORRECT
> results.txt 2>&1

FD1 → results.txt, then FD2 follows FD1 → results.txt ✅

## WRONG
2>&1 > results.txt

FD2 copies FD1 (still terminal), then FD1 moves → results.txt ❌
stderr ends up on screen

## How Pipes Work Under the Hood
- Pipe = kernel memory buffer (~64KB)
- Shell creates two FDs: pipe_read and pipe_write
- command1 stdout → pipe_write
- command2 stdin  → pipe_read
- Both run simultaneously
- If buffer fills up → command1 pauses until command2 drains it

command1 → pipe buffer → command2

## /dev/null
Special file that discards everything written to it.
The black hole. Use it to silence any output you don't care about.

```bash

## Useful Commands

## Count results
command | wc -l

# Send stdout to /dev/null with stderr
>/dev/null 2>&1

# List installed packages (Debian/Ubuntu)
dpkg -l | grep "^ii" | wc -l
```


Previous:
[[Day -2 Working with files and Directories]]

Next:
[[Day -4 Filtering Contents]]