# 04 — Piping & Redirection: Chaining Commands for Security

---

## Learning Objectives

By the end of this module you will be able to:

- Understand the three standard streams: `stdin`, `stdout`, and `stderr`
- Chain multiple commands together using pipes (`|`)
- Redirect standard output and standard error to files
- Suppress noisy error output during enumeration using `2>/dev/null`
- Use file descriptors (0, 1, 2) to control where data flows
- Build multi-step command pipelines for log analysis and reconnaissance
- Understand why piping and redirection are essential for security work

---

## Why This Matters

Security professionals rarely use single commands. Almost every real-world task involves **chaining tools together**: capturing output from one tool and feeding it as input to another. When you enumerate a target system during penetration testing, you'll use pipelines constantly to:

- Filter noise and extract relevant data
- Suppress errors so you can focus on results
- Chain multiple analysis steps into a single powerful command
- Automate tedious manual analysis tasks

Learning to compose powerful one-liners is what separates beginners from professionals.

```bash
# Real-world example: Find top attacking IPs from SSH logs
grep "Failed password" /var/log/auth.log \
  | awk '{print $11}' \
  | sort | uniq -c | sort -rn \
  | head -10
```

---

## Theory

### Understanding the Three Standard Streams

Every Linux process has three built-in data streams that allow it to communicate with the operating system and the user:

| Stream | File Descriptor | Default Source/Destination | Purpose |
|--------|----------------|-----------------------------|---------|
| `stdin` | 0 | Keyboard | Data going INTO a program |
| `stdout` | 1 | Terminal screen | Normal output FROM a program |
| `stderr` | 2 | Terminal screen | Error messages FROM a program |

**Visual representation:**

```
User's keyboard  →  [stdin - File Descriptor 0]  →  Your command
                                                        ↓
                                    Your command processes input
                                                        ↓
                              ┌─────────────────────────┴─────────────────────────┐
                              ↓                                                   ↓
                    [stdout - FD 1]                                    [stderr - FD 2]
                    Normal results                                     Error messages
                              ↓                                                   ↓
                    Terminal screen                                    Terminal screen
                    (or redirected)                                    (or redirected)
```

**Why does this matter?** When you use redirection operators, you're telling the shell: "Take the output from file descriptor X and send it to location Y instead of the default."

**Try it now:**

```bash
ls /home /nonexistent
# You'll see:
# - Output from /home on stdout
# - "Permission denied" error on stderr
# Both appear on your terminal

ls /home /nonexistent 2>/dev/null
# Now only the successful /home output appears
# The error goes to /dev/null (the black hole)
```

---

### What is `/dev/null`?

`/dev/null` is a special file that acts as a "black hole" for data. Anything you send to it is discarded immediately. It's useful when you want to suppress output but don't care about saving it.

Think of it as a trash can. Once you throw something in, it's gone.

```bash
command 2>/dev/null
# Errors are sent to /dev/null and disappear
```

---

### Piping — Chaining Commands Together with `|`

A pipe (`|`) connects the **stdout** of one command directly to the **stdin** of the next command. The data flows through the pipe without being written to disk or displayed on screen.

**Syntax:**

```bash
command1 | command2 | command3
```

Breaking this down:
- `command1` runs and produces output (stdout)
- That output becomes input (stdin) for `command2`
- `command2` processes it and produces new output
- That output becomes input for `command3`
- And so on...

**Important:** Only `stdout` is piped. If a command produces error messages (stderr), those go directly to your terminal, not through the pipe.

**Security examples:**

```bash
ps aux | grep sshd
# Find running sshd processes

grep "Failed" /var/log/auth.log | wc -l
# Count failed login attempts

cat /etc/passwd | cut -d: -f1 | sort
# Extract and sort all usernames

grep "192.168.1" /var/log/access.log | grep "404" | wc -l
# Find how many 404 errors came from 192.168.1.x subnet
```

**Try it now:**

```bash
echo "banana apple cherry apple banana" | tr ' ' '\n' | sort | uniq -c
# Count occurrences of each word:
# - echo produces the words
# - tr converts spaces to newlines (one word per line)
# - sort arranges them
# - uniq -c counts occurrences
```

---

### Output Redirection — Saving Results to Files

By default, `stdout` goes to your terminal screen. With `>` and `>>`, you can redirect it to a file instead.

**Syntax:**

```bash
command > filename       # Overwrite the file with the output
command >> filename      # Append the output to the end of the file
```

**Important distinction:**

- `>` — overwrites the file (deletes previous contents)
- `>>` — appends to the file (keeps previous contents)

**Examples:**

```bash
ls > file_list.txt
# Create a new file with list of files in current directory

date >> file_list.txt
# Add the current date to the end of file_list.txt

echo "192.168.1.100" >> allowed_ips.txt
# Add an IP address to a whitelist

nmap -sV 192.168.1.0/24 > network_scan.txt
# Save nmap scan results to a file
```

**Try it now:**

```bash
echo "test1" > output.txt
echo "test2" >> output.txt
cat output.txt
# You should see both lines, because >> appended the second
```

---

### Error Redirection — Controlling stderr

By default, errors (stderr) go directly to your terminal, where they can make output confusing. You can redirect errors using the file descriptor number.

Remember: **stderr is file descriptor 2**

**Syntax:**

```bash
command 2> error_file       # Redirect errors to a file
command 2>/dev/null         # Discard errors
command > output.txt 2>&1   # Send both stdout and stderr to the same file
command &> filename         # Shorthand for above
command 2>&1 | grep pattern # Redirect errors and pipe them to grep
```

**Breaking down the syntax:**

- `2>` — redirect file descriptor 2 (stderr) to...
- `/dev/null` — the black hole (discard it)
- `2>&1` — redirect FD 2 (stderr) to FD 1 (stdout)
- `&>` — shorthand for redirecting both stdout and stderr

**Why `2>&1` is useful:** You can pipe both normal output and errors through a pipeline.

```bash
command 2>&1 | grep "error"
# Both stdout and stderr go through the pipe
# Then grep filters for lines containing "error"
```

**Security examples:**

```bash
find / -name "id_rsa" 2>/dev/null
# Find SSH keys, suppress "Permission denied" errors from inaccessible directories

find /tmp -mmin -60 2>/dev/null > recent_files.txt
# Find recently modified files in /tmp, ignore permission errors, save results

nmap -sV 192.168.1.0/24 > scan.txt 2> scan_errors.txt
# Save successful scan output and error messages to separate files
```

**Try it now:**

```bash
# Create a scenario that produces both output and errors:
find /home /root /nonexistent -type f -name "*.txt" 2>/dev/null
# You'll only see results from /home and /root
# Errors from /nonexistent and permission denials are suppressed

ls /tmp /nonexistent 2>&1 | head -5
# Show first 5 lines of both output and errors
```

---

### Reviewing Module 02: Understanding `2>/dev/null` in find

In Module 02, you learned that `find` commands often used `2>/dev/null` to suppress permission errors:

```bash
find / -name "id_rsa" 2>/dev/null
```

Now you understand exactly what this does:
- `find /` — searches the entire filesystem
- `-name "id_rsa"` — looks for files named id_rsa
- `2>/dev/null` — **sends all error messages (stderr) to /dev/null**, so permission denied errors don't clutter your output

Without this, you'd see hundreds of "Permission denied" messages as `find` tries to access directories you don't have permission to read. With `2>/dev/null`, only the actual results appear.

---

### Input Redirection — `<`

Input redirection is less common, but allows you to feed a file as input to a command that normally reads from the keyboard.

**Syntax:**

```bash
command < filename       # Use file as input instead of keyboard
```

**Example:**

```bash
sort < unsorted.txt
# Instead of sorting keyboard input, sort the contents of unsorted.txt

wc -l < file.txt
# Count lines in a file
```

This is rarely used because pipes are usually more practical.

---

### Named Pipes — Creating Channels Between Processes

A named pipe (FIFO — First In, First Out) is a special file that acts as a channel between two processes. It persists on the filesystem (unlike regular pipes, which only exist in memory).

**Syntax:**

```bash
mkfifo pipe_name           # Create a named pipe
```

**How it works:**

```bash
# Terminal 1: Create the pipe
mkfifo MyPipe

# Terminal 1: Write to the pipe in background
echo "secret message" > MyPipe &

# Terminal 2: Read from the pipe
cat < MyPipe
# Output: secret message
```

**Security use case:** Security analysts monitor for unexpected named pipes on compromised systems. Attackers sometimes use them for covert communication between processes.

---

### `tee` — Splitting Output (Both Save AND Display)

By default, when you redirect output to a file with `>`, you lose the terminal display. The `tee` command lets you save output to a file **AND** display it on the terminal simultaneously.

**Syntax:**

```bash
command | tee filename
```

**Examples:**

```bash
nmap -sV 192.168.1.1 | tee scan.txt
# Display scan results on screen AND save to scan.txt

grep "Failed" /var/log/auth.log | tee failed_logins.txt
# Show failed logins on screen AND save to file

ls -la | tee -a logfile.txt
# Append (-a flag) to an existing file while displaying
```

**Why use `tee`?** During a long-running analysis, you want to see results immediately (for feedback) while also saving them for your report.

---

### Building Security One-Liners — Putting It All Together

Now that you understand piping and redirection, you can build powerful single-command analyses that would otherwise require multiple steps.

**Example 1 — Find top attacking IPs:**

```bash
grep "Failed password" /var/log/auth.log \
  | grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" \
  | sort | uniq -c | sort -rn | head -10
```

Breaking this down:
1. `grep "Failed password"` — find failed login attempts
2. `grep -oE "[0-9]..."` — extract only IP addresses (one per line)
3. `sort` — arrange IPs in alphabetical order
4. `uniq -c` — count consecutive identical IPs
5. `sort -rn` — sort by count, highest first
6. `head -10` — show top 10 results

**Example 2 — Extract and sort users who have bash shells:**

```bash
grep "/bin/bash" /etc/passwd \
  | cut -d: -f1 \
  | sort
```

Breaking this down:
1. `grep "/bin/bash"` — find lines where shell is bash
2. `cut -d: -f1` — extract username (field 1, delimiter :)
3. `sort` — arrange alphabetically

**Example 3 — Count log entries by hour:**

```bash
grep "$(date +%Y-%m-%d)" /var/log/auth.log \
  | awk '{print $1, $2}' \
  | sort | uniq -c | sort -rn
```

Breaking this down:
1. `grep "2024-01-15"` — find today's log entries
2. `awk '{print $1, $2}'` — extract date and time
3. `sort | uniq -c` — count each unique timestamp
4. `sort -rn` — sort by count, highest first

**Example 4 — Monitor for new root user additions:**

```bash
diff /etc/passwd.bak /etc/passwd \
  | grep "^>" \
  | tee root_changes.txt
```

Breaking this down:
1. `diff` — compare old and new passwd file
2. `grep "^>"` — show only newly added lines
3. `tee` — save to file AND display on screen

---

## Exercises

The exercises for this module are in the drill file.

> [Go to the drills](./drill.md)
