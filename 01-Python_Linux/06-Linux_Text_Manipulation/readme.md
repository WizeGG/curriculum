# 03 — Text Manipulation: Log Analysis & IOC Extraction

---

## Learning Objectives

By the end of this module you will be able to:

- Read and filter large log files using `cat`, `head`, `tail`, `grep`
- Understand and use regular expressions with `grep` for pattern matching
- Extract specific columns or fields from structured text using `cut` and `awk`
- Count, sort, and deduplicate data using `wc`, `sort`, and `uniq`
- Replace patterns in files using `sed` and understand the `s/pattern/replacement/flags` syntax
- Identify Indicators of Compromise (IOCs) in log files
- Produce actionable reports from raw log data

---

## Why This Matters

Log analysis is **the core skill of a SOC analyst**. When an alert fires, you will be handed log files — sometimes gigabytes of them. Your ability to quickly filter, parse, and extract meaningful data determines how fast you can confirm or dismiss a threat.

Every tool in this module is a weapon in a SOC analyst's arsenal. These are the tools you will use to hunt attackers, extract Indicators of Compromise (IOCs), and build threat reports.

---

## Theory

### Reading Files

Before you analyze data, you need to view it. The following commands help you read and navigate large files without opening them in an editor.

```bash
cat /var/log/auth.log             # Print entire file (careful with large files!)
less /var/log/auth.log            # Page through file interactively (q to quit)
head -n 20 /var/log/auth.log      # First 20 lines
tail -n 50 /var/log/auth.log      # Last 50 lines
tail -f /var/log/auth.log         # Live view — updates as new lines are added
```

**Important flags:**
- `-n` specifies the number of lines to display

**Security use case:** `tail -f` is invaluable when monitoring a live system for attacks in real time. Open one terminal and run `tail -f /var/log/auth.log`, then simulate an attack (e.g., failed SSH login) in another terminal. You'll see the attack logged in real time.

---

### `grep` — Filtering Lines by Pattern

`grep` is the most important text tool in a security analyst's toolkit. It searches for lines matching a pattern and prints them to the screen.

**Basic syntax:**

```bash
grep [flags] "pattern" filename
```

The pattern can be a literal string or a regular expression (regex).

**Common flags:**

- `-i` — ignore case (search case-insensitively)
- `-v` — invert match (show lines that DO NOT match)
- `-n` — show line numbers
- `-c` — count matching lines (instead of printing them)
- `-r` — recursive (search in directories and subdirectories)
- `-A N` — show N lines AFTER each match
- `-B N` — show N lines BEFORE each match
- `-C N` — show N lines of context (before AND after)
- `-E` — use Extended Regular Expressions (more powerful patterns)

**Simple literal string searches:**

```bash
grep "Failed password" /var/log/auth.log         # Find failed SSH logins
grep "Accepted password" /var/log/auth.log       # Find successful SSH logins
grep "CRON" /var/log/syslog                      # Find cron job activity
grep -i "error" /var/log/syslog                  # Case-insensitive search (matches "Error", "ERROR", etc.)
grep -v "^#" /etc/ssh/sshd_config                # Exclude comment lines (lines starting with #)
grep -r "password" /etc/                         # Recursive search through all files in /etc
grep -n "root" /etc/passwd                       # Show line numbers for matching lines
grep -c "Failed" /var/log/auth.log               # Count matching lines (output: a single number)
```

**Try it now:**

```bash
grep "Failed" /var/log/auth.log | head -5
# See failed login attempts in your auth log
```

**Using grep with context:**

```bash
grep -A 3 "Failed password" /var/log/auth.log    # Show 3 lines AFTER each failed login
grep -B 2 "Failed password" /var/log/auth.log    # Show 2 lines BEFORE each failed login
```

This is useful in security analysis because the context often contains useful information (like the attacking IP address).

---

#### Understanding Regular Expressions

Regular expressions (regex) are patterns that match text. They're more powerful than simple string matching.

**Key regex syntax:**

- `^` — start of line (e.g., `^root` matches lines starting with "root")
- `$` — end of line (e.g., `bash$` matches lines ending with "bash")
- `.` — matches any single character (e.g., `a.c` matches "abc", "a1c", "aXc")
- `*` — matches zero or more of the previous character (e.g., `ab*c` matches "ac", "abc", "abbc", "abbbc")
- `+` — matches one or more of the previous character (e.g., `ab+c` matches "abc", "abbc" but NOT "ac")
- `[a-z]` — character class: matches any single character in the range (e.g., `[a-z]` matches any lowercase letter)
- `[0-9]` — matches any digit
- `\d` — shorthand for [0-9] (in Perl regex)
- `{N}` — matches exactly N occurrences (e.g., `a{3}` matches "aaa")
- `{N,M}` — matches between N and M occurrences

**Examples with actual log data:**

```bash
grep "^root" /etc/passwd                                      # Lines starting with "root"
grep "bash$" /etc/passwd                                      # Lines ending with "bash"
grep "[0-9]\{1,3\}\.[0-9]\{1,3\}" /var/log/auth.log          # Basic IP pattern
```

**Try it now:**

```bash
grep "^[a-z]" /etc/passwd | head -3
# Show first 3 lines starting with a lowercase letter
```

**Extended Regular Expressions with -E flag:**

The `-E` flag enables Extended Regular Expressions (ERE), which support more powerful syntax.

```bash
grep -E "^[a-z]+.*[0-9]$" /etc/passwd              # Lines starting with lowercase letters, ending with digits
grep -E "([0-9]{1,3}\.){3}[0-9]{1,3}" auth.log    # More flexible IP address matching
grep -E "failed|error|warning" /var/log/syslog    # Lines containing any of three words
```

**Try it now:**

```bash
grep -E "[0-9]{1,3}\.[0-9]{1,3}" /var/log/auth.log | head -3
# Find lines with partial IP addresses
```

---

### `cut` — Extract Columns from Structured Text

Many important files on Linux are structured as columns separated by a delimiter (a character that separates fields). The most common delimiter is `:` (colon).

`cut` extracts specific columns (fields) from these structured files.

**Syntax:**

```bash
cut -d "delimiter" -f field_numbers filename
```

Breaking this down:
- `-d` — the delimiter character (which character separates fields)
- `-f` — which fields to extract (field numbers, comma-separated)

**Important:** Fields are numbered starting from 1 (not 0).

**Understanding /etc/passwd:**

The `/etc/passwd` file contains user account information. Each line represents one user, with fields separated by colons:

```
username:x:UID:GID:full_name:home_directory:login_shell
student:x:1000:1000:Student User:/home/student:/bin/bash
```

**Extracting fields from /etc/passwd:**

```bash
cut -d ":" -f 1 /etc/passwd              # Extract field 1 (usernames only)
cut -d ":" -f 1,3 /etc/passwd            # Extract fields 1 and 3 (username and UID)
cut -d ":" -f 1,6 /etc/passwd            # Extract username and home directory
cut -d ":" -f 1,7 /etc/passwd            # Extract username and login shell
```

**Try it now:**

```bash
cut -d ":" -f 1 /etc/passwd | head -5
# Show first 5 usernames
```

**Extracting IP addresses from web logs:**

```bash
cut -d " " -f 1 /var/log/nginx/access.log    # Extract the first field (IP address) from web logs
```

**Why this matters:** In the `/etc/passwd` file, you can see which users have a real login shell (like `/bin/bash`) versus service accounts (which have `/bin/false` or `/usr/sbin/nologin`).

---

### `sort` and `uniq` — Counting and Deduplicating Data

**`sort` — arranging data in order:**

```bash
sort file.txt                         # Sort alphabetically (ASCII order)
sort -n file.txt                      # Sort numerically (useful for numbers)
sort -r file.txt                      # Reverse sort (Z to A, or 999 to 1)
sort -u file.txt                      # Sort and remove duplicates in one pass
```

**`uniq` — removing or counting duplicates:**

```bash
uniq file.txt                         # Remove consecutive duplicate lines
uniq -c file.txt                      # Count occurrences of each line
uniq -d file.txt                      # Show only lines that appear more than once
```

**Critical difference:** `uniq` only works on consecutive identical lines. You must `sort` first!

**Combining sort and uniq to find the most common items:**

```bash
sort file.txt | uniq -c | sort -rn
```

Breaking this down:
1. `sort file.txt` — arrange lines in order
2. `uniq -c` — count consecutive identical lines
3. `sort -rn` — sort by count in reverse numeric order (highest first)

**Security use case — Find top attacking IPs:**

```bash
grep "Failed password" /var/log/auth.log \
  | grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" \
  | sort | uniq -c | sort -rn | head -10
```

Breaking this down:
1. `grep "Failed password"` — find all failed login attempts
2. `grep -oE "[0-9]{1,3}..."` — extract only IP addresses (one per line)
3. `sort` — organize them in order
4. `uniq -c` — count occurrences
5. `sort -rn` — sort by count, highest first
6. `head -10` — show top 10

**Try it now:**

Create a test file and analyze it:

```bash
cat > test.txt << EOF
apple
banana
apple
apple
cherry
banana
EOF

sort test.txt | uniq -c | sort -rn
# Output should show: apple appears 3 times, banana 2 times, cherry 1 time
```

---

### `wc` — Word / Line / Character Count

`wc` counts various aspects of text files.

**Flags:**

- `-l` — count lines
- `-w` — count words
- `-c` — count characters (bytes)
- `-m` — count characters (Unicode-aware)

**Security examples:**

```bash
wc -l /var/log/auth.log               # How many log entries are there?
grep "Failed" /var/log/auth.log | wc -l  # How many failed logins?
grep "Accepted" /var/log/auth.log | wc -l  # How many successful logins?
```

---

### `sed` — Stream Editor (Find and Replace)

`sed` is a powerful tool for replacing text patterns in files. It's especially useful for redacting sensitive information from logs.

**Basic syntax:**

```bash
sed 's/pattern/replacement/flags' filename
```

Breaking this down:
- `s` — substitute (find and replace)
- `pattern` — what to find (can be a literal string or regex)
- `replacement` — what to replace it with
- `flags` — modifiers:
  - `g` — global (replace all occurrences on each line, not just the first)
  - `i` — case-insensitive
  - `p` — print the modified line (usually used with `-n`)

**Important:** By default, `sed` prints all lines. Modified lines are printed with your changes. Use `-n` to suppress output and only print lines you've modified.

**Examples:**

```bash
sed 's/admin/REDACTED/g' creds.txt
# Replace "admin" with "REDACTED" (global flag = all occurrences)

sed 's/password=[^ ]*/password=****/g' logs.txt
# Redact passwords in logs (replace "password=ANYTHING" with "password=****")

sed -n '10,20p' /var/log/auth.log
# Print only lines 10 to 20 (the 'p' flag, combined with -n to suppress normal output)

sed '/^#/d' /etc/ssh/sshd_config
# Delete comment lines (lines starting with #)

sed -i 's/PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
# Edit in place (-i flag): modify the file directly without creating a backup
```

**Try it now:**

Create a test file with passwords and redact them:

```bash
cat > test_creds.txt << EOF
user1:password123:admin
user2:supersecret:guest
EOF

sed 's/:password[^:]*/:REDACTED/g' test_creds.txt
# Now all passwords are hidden
```

---

### `awk` — Powerful Field-Based Processing

`awk` is a full programming language for processing structured text. It's incredibly powerful for log analysis.

**Basic syntax:**

```bash
awk '[condition] {action}' filename
```

Key concepts:
- `awk` processes input line by line
- Each line is automatically split into fields
- `$1` — first field
- `$2` — second field
- `$0` — entire line
- `-F` — field separator (default is space/whitespace)
- `BEGIN` — runs before processing any lines
- `END` — runs after processing all lines

**Simple field extraction:**

```bash
awk '{print $1}' /var/log/apache2/access.log       # Print first field (IP address)
awk '{print $1, $7}' /var/log/apache2/access.log   # Print IP (field 1) and URL (field 7)
awk -F ":" '{print $1}' /etc/passwd                # Extract usernames (with colon delimiter)
```

**Try it now:**

```bash
awk '{print $1}' /etc/passwd
# This won't work well because the default field separator is whitespace
# But /etc/passwd uses colons, so:
awk -F ":" '{print $1}' /etc/passwd
# Now you get usernames!
```

**Pattern matching in awk:**

```bash
awk '/Failed/{print $11}' /var/log/auth.log        # For lines containing "Failed", print field 11 (IP)
awk '$9 == "404" {print $1}' access.log            # Print IPs where field 9 (status code) is 404
awk '/error/ {count++} END {print count}' syslog   # Count lines containing "error"
```

**Aggregating data with awk:**

```bash
# Sum numbers in first column
awk '{sum += $1} END {print "Total:", sum}' numbers.txt

# Count occurrences of each IP (similar to sort | uniq -c)
awk '{count[$1]++} END {for (ip in count) print count[ip], ip}' access.log \
  | sort -rn | head -10
```

**Why awk is powerful:** With `awk`, you can do in one command what would normally take multiple commands piped together. This is especially valuable in security analysis.

**Try it now:**

Create a test CSV file and analyze it:

```bash
cat > logins.csv << EOF
alice,10.0.0.1,success,2024-01-15
bob,10.0.0.2,failed,2024-01-15
admin,192.168.1.100,failed,2024-01-16
EOF

awk -F "," '{print $1, $2, $3}' logins.csv
# Extract username, IP, and status
```

---

### `diff` — Compare Files

`diff` shows the differences between two files. This is crucial for detecting unauthorized changes to configuration files.

```bash
diff config.original config.modified    # See what changed
diff -u /etc/passwd.bak /etc/passwd     # Unified diff format (easier to read)
```

**Security use case:** After a suspected breach, compare a known-good backup of critical files against the current version to identify what was modified.

---

## Exercises

The exercises for this module are in the drill file.

> [Go to the drills](./drill.md)
