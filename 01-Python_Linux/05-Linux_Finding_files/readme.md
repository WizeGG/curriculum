# 02 — Finding Files: Reconnaissance on Linux

---

## Learning Objectives

By the end of this module you will be able to:

- Use `locate`, `find`, `which`, and `whereis` to search the file system
- Understand the difference between database-driven search and real-time search
- Find sensitive files that should not be world-readable
- Identify SUID/SGID binaries — common privilege escalation vectors
- Search for files containing credentials or secrets
- Break down complex `find` commands and understand every component
- Understand how attackers use these same tools during post-exploitation

---

## Why This Matters

**For attackers (pentesters):** After gaining initial access to a system, the first step is reconnaissance. What files are here? Where are the credentials? Are there any misconfigured binaries? The tools you'll learn in this module are in every attacker's playbook.

**For defenders (SOC/Blue Team):** You need to audit your systems regularly. Are there files with overly permissive permissions? Are there unexpected SUID binaries? Are secrets stored in plain text somewhere? You use the same tools to hunt threats.

Both sides use exactly the same tools.

---

## Theory

### `locate` — Fast File Search Using a Database

`locate` searches a pre-built database of filenames on your system. This database is created and maintained by the system — it does not scan the filesystem in real time. This makes `locate` very fast, but it has a critical limitation: if you create a new file, it may not appear in `locate` results until the database is refreshed.

The database is typically updated once per day automatically, but you can refresh it manually.

**How it works:**

The system maintains a database (usually `/var/lib/mlocate/mlocate.db`) that maps every filename to its location. When you run `locate`, it searches this index rather than searching the disk directly.

```bash
locate passwd          # Find all files with "passwd" in the name
locate .ssh            # Find SSH-related files and directories
locate id_rsa          # Find RSA private keys
```

**Try it now:**

Create a test file and see if `locate` finds it immediately:

```bash
touch /tmp/test-secret-key.txt
locate test-secret-key
# You probably won't find it yet!
```

Then update the database and try again:

```bash
sudo updatedb
locate test-secret-key
# Now it should appear
```

**Security use case:** As a security analyst, `locate` is useful for quickly checking whether a known sensitive filename exists anywhere on the system — but only if the database is up to date.

---

### `find` — Precise Real-Time Search of the Live Filesystem

`find` is fundamentally different from `locate`. It does not use a pre-built database. Instead, it searches the **live filesystem** directly, walking through directories one by one. This makes `find` slower than `locate`, but infinitely more powerful because you can apply complex filters.

**The basic `find` command structure:**

```bash
find [starting-path] [test-conditions] [action]
```

Breaking this down:
- **starting-path**: Where to begin searching (e.g., `/`, `/home`, `/tmp`). If omitted, defaults to current directory.
- **test-conditions**: How to filter results (by name, type, permissions, owner, modification time, etc.)
- **action**: What to do with results (default is to print the path; `-exec` allows you to run a command)

**Example breakdown:**

```bash
find / -type f -name "*.log" -mtime -1 -exec ls -l {} \;
```

Let's dissect this:
- `find` — the command
- `/` — start from the root directory (search everywhere)
- `-type f` — test condition: only regular files (not directories or links)
- `-name "*.log"` — test condition: name must end with `.log`
- `-mtime -1` — test condition: modified time must be less than 1 day ago
- `-exec ls -l {} \;` — action: for each result, run `ls -l` on it

When a file matches ALL test conditions (file type AND name AND modification time), the action is executed.

---

#### Searching by Name

The simplest form of searching. The `-name` flag matches against the filename only (not the full path).

```bash
find / -name "*.log"           # All files ending in .log, anywhere on system
find / -name "password*"       # Files whose name starts with "password"
find /home -name ".bash_history"  # History files in all home directories
find /etc -name "*.conf"       # Configuration files in /etc
```

**Try it now:**

```bash
find /etc -name "*.conf" | head -5
# Shows first 5 .conf files found in /etc
```

**For sensitive file hunting:**

```bash
find / -name "id_rsa"          # SSH private keys (dangerous if world-readable!)
find / -name ".ssh"            # SSH directories
find / -name "*password*"      # Files with "password" in the name
```

---

#### Searching by File Type

The `-type` flag filters by what kind of file we're looking for. Common types:

- `-type f` — regular files (text, binary, etc.)
- `-type d` — directories
- `-type l` — symbolic links
- `-type s` — sockets
- `-type p` — named pipes (FIFOs)

```bash
find / -type f -name "*.conf"    # Regular files (not directories) ending in .conf
find / -type d -name ".ssh"      # Only directories named .ssh
find / -type l                   # All symbolic links on the system
```

**Why this matters in security:** Attackers sometimes hide files in plain sight by using symbolic links to point to sensitive data. Finding all symlinks helps you audit these.

**Try it now:**

```bash
find /etc -type f -name "*.conf"
# How many config files did you find? Compare to:
find /etc -name "*.conf"
# You might find more because this also includes directories
```

---

#### Searching by Permissions — The Most Critical Tool for Security

This is where `find` becomes a security weapon. The `-perm` flag allows you to hunt down files with dangerous permissions.

**Understanding permission bits:**

Every file has a permission value that can be expressed as:
- A string like `rwxr-xr-x`
- An octal number like `755` or `644`

Special permission bits:
- `4000` (or `u+s`) — SUID bit set
- `2000` (or `g+s`) — SGID bit set
- `1000` (or `o+t`) — Sticky bit set

**The `-perm` flag syntax:**

```bash
find / -perm -4000 2>/dev/null   # Find all SUID files
find / -perm -2000 2>/dev/null   # Find all SGID files
find / -perm -o+w 2>/dev/null    # Find files writable by "others"
find / -perm 777 2>/dev/null     # Find files with rwxrwxrwx (fully open)
```

The minus sign before the octal number means "has at least these permission bits set." So `-perm -4000` means "SUID bit is set" (regardless of other permissions).

**What is SUID (Set User ID)?**

SUID is a special permission. When a binary has the SUID bit set, it runs with the permissions of its **owner** (usually root) rather than the permissions of the user who launched it.

This is necessary for some tools. For example, the `passwd` command needs to modify `/etc/shadow` (which is owned by root). Regular users can run `passwd` because it has the SUID bit set on the root-owned binary.

However, SUID is also a classic privilege escalation attack vector. If a SUID binary has a vulnerability, an attacker can exploit it to execute commands as root.

You can recognize a SUID binary by the `s` in its permission string:

```bash
ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 59640 /usr/bin/passwd
#   ^ this "s" means SUID is set
```

In this case:
- Owner can read, write, **and execute as owner** (`rws`)
- Group can read and execute (`r-x`)
- Others can read and execute (`r-x`)

**Try it now:**

Find and list all SUID binaries on your system:

```bash
find / -perm -4000 2>/dev/null | head -10
# List the first 10 SUID files
```

Then examine one of them:

```bash
ls -l /usr/bin/sudo
# You should see an "s" in the permissions
```

**What about SGID?**

SGID (Set Group ID) works similarly to SUID, but the binary runs with the group's permissions instead of the owner's. Less common than SUID but just as dangerous.

**Finding world-writable files:**

Files that any user on the system can write to are a major security risk. An attacker who gains access to one user account could modify these files to escalate privileges or perform lateral movement.

```bash
find / -perm -o+w 2>/dev/null    # Files where "others" have write permission
find / -perm 777 2>/dev/null     # Files with rwxrwxrwx (fully open to everyone)
```

---

#### Searching by Ownership

The `-user` and `-group` flags let you find files based on who owns them.

```bash
find / -user root -perm -4000 2>/dev/null   # SUID binaries owned by root
find /home -not -user student               # Files NOT owned by student
find / -nouser 2>/dev/null                  # Files with no valid owner (orphaned)
```

**Why orphaned files matter:** When a user account is deleted, their files may remain behind with a UID that no longer exists. These orphaned files are a forensic indicator of past user accounts or lateral movement on a compromised system.

---

#### Searching by Modification Time — Critical for Forensics

When responding to a security incident, you often need to find files that were recently modified. An attacker might have uploaded a backdoor, modified a config file, or planted a rootkit.

The `-mtime` and `-mmin` flags search by modification time:

- `-mtime` — measured in days
- `-mmin` — measured in minutes

The number prefix determines the direction:
- `-mtime 1` — exactly 1 day ago
- `-mtime -1` — less than 1 day ago (in the last 24 hours)
- `-mtime +1` — more than 1 day ago (before 24 hours ago)

```bash
find /var/log -mtime -1          # Log files modified in the last 1 day
find /tmp -mmin -60              # Files modified in the last 60 minutes
find / -newer /etc/passwd        # Files newer than /etc/passwd (useful as a reference point)
```

**Try it now:**

Create a test file and find it by modification time:

```bash
touch /tmp/forensic-test.txt
find /tmp -mmin -5
# Should find your newly created file (modified within last 5 minutes)
```

**Security use case:** After detecting a breach, you might run:

```bash
find / -mtime -7 -type f 2>/dev/null | grep -v /proc
# Find all files modified in the last week (excluding /proc which is volatile)
```

---

#### Combining Tests (AND Logic)

When you specify multiple conditions, they are combined with AND logic. The file must match ALL conditions to be returned.

```bash
find /tmp -name "*.sh" -type f -perm -o+w -mtime -1
```

This command finds files that match ALL of:
- Name ends in `.sh`
- AND it is a regular file (not a directory)
- AND it is writable by others
- AND it was modified in the last day

---

#### The `-exec` Action — Running a Command on Each Result

The `-exec` flag lets you run a command on each file that matches your criteria.

**Syntax:**

```bash
find [path] [conditions] -exec command {} \;
```

Breaking this down:
- `command` — the command to run (e.g., `ls -l`, `chmod 600`, `cat`)
- `{}` — placeholder for the filename (gets replaced with each result)
- `\;` — end of the -exec clause (backslash escapes the semicolon for the shell)

**Examples:**

```bash
find / -perm 777 -exec ls -la {} \;
# Show detailed info for all world-writable files

find /tmp -name "*.sh" -exec chmod 600 {} \;
# Change permissions on all shell scripts in /tmp (fix the vulnerability!)

find / -name "*.key" -exec file {} \;
# Determine the file type of every .key file on the system
```

**Try it now:**

```bash
find /etc -name "*.conf" -exec wc -l {} \;
# Count lines in each .conf file
```

---

#### Suppressing Error Messages with `2>/dev/null`

When you search the entire filesystem with `find /`, you will encounter many directories where you don't have permission to read. By default, `find` prints "Permission denied" errors for every inaccessible directory. This floods the output with noise.

The `2>/dev/null` redirection suppresses these error messages. You'll learn exactly what this means in module 04, but for now: it redirects error output to `/dev/null` (the black hole), making it disappear.

**Compare:**

```bash
find / -name "id_rsa"
# Lots of "Permission denied" errors

find / -name "id_rsa" 2>/dev/null
# Only the actual results, much cleaner
```

---

### `which` — Find an Executable in Your PATH

`which` tells you the full path of a command that is available in your shell's PATH environment variable.

When you type a command like `nmap`, the shell doesn't know where it is. Instead, it searches a list of directories (stored in `$PATH`) and executes the first match it finds. The `which` command tells you which file gets executed.

```bash
which nmap           # Returns: /usr/bin/nmap
which python3        # Returns: /usr/bin/python3
which nc             # Returns: /usr/bin/nc (netcat)
```

**If the command is not found:**

```bash
which nonexistent-tool
# Returns nothing (exit code 1)
```

**Why this matters in security:** During post-exploitation on a compromised system, you run `which` to quickly discover what tools are already installed. This tells you:
- Do I need to upload tools, or are they already here?
- What version of tools are available?
- Has the system been hardened to remove dangerous tools like `nc` and `wget`?

**Try it now:**

```bash
which find
which locate
which nmap
# Check which tools are in your PATH
```

---

### `whereis` — Find Binary, Manual Page, and Source Code

`whereis` is similar to `which`, but it searches for binaries, manual pages, and source code in standardized system directories.

```bash
whereis nmap         # Returns binary path, manual page, and source
whereis bash
```

**The difference from `which`:**

- `which` — searches only in `$PATH` (where executables are)
- `whereis` — searches in standardized locations like `/usr/bin`, `/usr/share/man`, `/usr/src`

`whereis` is useful when `which` fails to find something because it searches a wider set of directories.

---

## Exercises

The exercises for this module are in the drill file.

> [Go to the drills](./drill.md)
