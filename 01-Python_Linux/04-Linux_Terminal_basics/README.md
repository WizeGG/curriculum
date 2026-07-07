# 01 — Terminal Basics for Security Professionals

---

## Learning Objectives

By the end of this module you will be able to:

- Explain what Linux is and why it dominates the server world
- Describe the role of the kernel, the shell, and the filesystem
- Navigate the Linux file system with confidence
- Perform file operations from the command line
- Understand users, groups, root, and `sudo`
- Manage file permissions — the single most important security concept on Linux
- Use a text editor and a package manager
- Read the manual for any command

---

## Why This Matters

The majority of servers on the internet run Linux. When you land on a machine during a pentest, or when you investigate an alert on a production server, there will be no graphical interface. Just a blinking cursor in a terminal. Penetration testers, SOC analysts, and incident responders all spend the majority of their time in a Linux shell. If you want to work in cybersecurity, mastering Linux is non-negotiable.

---

## Theory

### What is Linux?

Linux is an **operating system** — the software that sits between the hardware (CPU, RAM, disk) and the applications you run. When someone says "Linux", they usually mean the full operating system, but technically Linux is only the **kernel**.

**The kernel** is the core of the OS. It manages hardware resources: which process gets CPU time, how memory is allocated, how data is read from disk, how network packets are sent. You never interact with the kernel directly — you interact through programs that talk to the kernel on your behalf.

**The shell** is the program that reads what you type in the terminal and translates it into instructions for the kernel. The most common shell on Linux is **Bash** (Bourne Again Shell). When you open a terminal and type `ls`, Bash reads that input, finds the `/bin/ls` program, asks the kernel to execute it, and displays the output back to you.

**A distribution (distro)** is a complete package: the Linux kernel + a set of pre-installed tools + a package manager + a configuration system. Different distributions are built for different purposes:

| Distribution | Used for | You will encounter it |
|-------------|---------|----------------------|
| **Ubuntu / Debian** | Servers, desktops | Most common server distro you will audit or defend |
| **Kali Linux** | Penetration testing | Your attack machine — pre-installed with security tools |
| **CentOS / RHEL / Rocky** | Enterprise servers | Corporate infrastructure |
| **Arch Linux** | Advanced users | Less common in enterprise, but good to know |
| **Alpine** | Containers (Docker) | Minimal, fast — used in cloud environments |

All these distributions share the same kernel and the same core commands. Once you know one, you can work on any of them.

**Try it now:** Open a terminal on your VM and run:
```bash
uname -a
```
This prints information about the kernel. You should see the kernel version and architecture. Then run:
```bash
cat /etc/os-release
```
This tells you which distribution you are running.

---

### Users, Groups, Root, and sudo

Linux is a **multi-user system**. Several people (or services) can use the same machine at the same time, each with their own account, home directory, and permissions.

**root** is the superuser — the administrator account with unlimited power. Root can read any file, kill any process, and modify any configuration. Because of this, you should almost never log in as root directly. Instead, you use `sudo`.

**sudo** stands for "**s**uper **u**ser **do**". It lets a normal user run a single command with root privileges, if that user has been granted permission in the `/etc/sudoers` file.

```bash
whoami                    # Print your current username
id                        # Print your user ID, group ID, and all groups you belong to
sudo whoami               # Temporarily become root for this one command — prints "root"
sudo cat /etc/shadow      # Read a file only root can access
```

**Groups** are collections of users. Permissions can be set per group, which makes it easy to manage access for teams. For example, all web developers might belong to the `www-data` group and share access to `/var/www`.

```bash
groups                    # List groups your user belongs to
cat /etc/group            # List all groups on the system
cat /etc/passwd           # List all user accounts (one per line)
```

The `/etc/passwd` file contains one line per user. Despite its name, it does not contain passwords (those are in `/etc/shadow`, which only root can read). Each line has fields separated by colons:

```
alice:x:1001:1001:Alice Smith:/home/alice:/bin/bash
│     │  │    │    │           │           └── Default shell
│     │  │    │    │           └── Home directory
│     │  │    │    └── Full name (comment field)
│     │  │    └── Primary group ID
│     │  └── User ID (UID)
│     └── Password placeholder (actual hash is in /etc/shadow)
└── Username
```

**Try it now:** Run `id` in your terminal and read the output. Then run `cat /etc/passwd` and find your own user line. Can you identify all the fields?

---

### Understanding Commands and Flags

A **command** is a program you run in the terminal. For example, `ls` is a command that lists files.

A **flag** (also called an **option** or **switch**) modifies the behaviour of a command. Flags usually start with a dash (`-`) for short form, or double dash (`--`) for long form. For example:

```bash
ls           # Lists files in the current directory
ls -l        # The flag -l means "long format" — shows permissions, owner, size, date
ls -a        # The flag -a means "all" — includes hidden files (those starting with a dot)
ls -la       # You can combine short flags: this does both -l and -a at once
ls --all     # Same as -a, but the long form — more readable, same result
```

The general pattern is always: `command [flags] [arguments]`

```bash
cp file.txt backup/      # cp is the command, file.txt and backup/ are arguments
cp -r folder/ backup/    # -r is a flag meaning "recursive" (include everything inside)
cp -v file.txt backup/   # -v is a flag meaning "verbose" — shows what is being done
```

When you encounter an unfamiliar command, you can always check which flags it accepts by reading its manual page (covered later) or running it with `--help`:
```bash
ls --help
```

**Try it now:** Run `ls -l /etc` in your terminal. Read the output carefully. Each line starts with a permissions string, then a number, then the owner, the group, the size, the date, and the filename. You don't need to understand everything yet — we will break it down shortly.

---

### The Linux File System

Linux organises everything into a single tree of directories starting from `/` (the root directory — not to be confused with the root user). There are no drive letters like `C:\` on Windows. Everything is mounted under `/`.

```
/
├── bin/        → Essential command binaries (ls, cp, cat...)
├── sbin/       → System binaries (for root/admin use)
├── etc/        → Configuration files (all system settings live here)
├── home/       → Home directories for regular users
│   ├── alice/
│   └── bob/
├── root/       → Home directory for the root user
├── var/        → Variable data (logs, mail, temporary spool files)
│   └── log/    → System and application log files
├── tmp/        → Temporary files (writable by everyone, cleaned on reboot)
├── usr/        → User programs and libraries
│   ├── bin/    → Most user commands
│   └── lib/    → Shared libraries
├── proc/       → Virtual filesystem — info about running processes
├── dev/        → Device files (disks, terminals, USB devices)
├── opt/        → Optional/third-party software
└── mnt/        → Mount points for external filesystems
```

As a security professional, you need to know where things live:

| Directory | What it contains | Why you care |
|-----------|-----------------|--------------|
| `/etc/passwd` | User accounts | Enumerate users on the system |
| `/etc/shadow` | Password hashes | Target for cracking — only readable by root |
| `/etc/ssh/sshd_config` | SSH server configuration | Check if root login is allowed, which auth methods are enabled |
| `/var/log/auth.log` | Authentication logs | Who logged in, who failed, from where |
| `/var/log/syslog` | General system logs | System events, errors, service activity |
| `/home/user/.ssh/` | SSH keys for that user | Private keys = access to other machines |
| `/home/user/.bash_history` | Command history | What did this user do? Credentials leaked? |
| `/tmp` | Temporary files | Attackers often stage payloads here (world-writable) |
| `/proc` | Process information | Examine running processes, recover deleted files |

**Try it now:** Navigate to a few of these directories and look around:
```bash
ls /etc
ls /var/log
ls /home
ls /tmp
```

---

### Navigation

**`pwd`** — **P**rint **W**orking **D**irectory. Tells you where you currently are in the filesystem.

```bash
pwd
# Output: /home/alice
```

**`ls`** — **L**i**s**t files and directories.

```bash
ls              # Simple list of files and directories
ls -l           # Long format: permissions, owner, group, size, date, name
ls -a           # All files, including hidden ones (names starting with a dot)
ls -la          # Both: long + hidden
ls -lh          # Long + human-readable sizes (4.2K instead of 4096)
ls /var/log     # List a specific directory without navigating into it
```

**`cd`** — **C**hange **D**irectory. Moves you from one place to another in the filesystem.

```bash
cd /etc               # Go to /etc (absolute path — starts from /)
cd ssh                # Go into a subdirectory called "ssh" (relative to current location)
cd ..                 # Go up one level (.. always means "parent directory")
cd ../..              # Go up two levels
cd ~                  # Go to your home directory (~ is a shortcut for /home/yourname)
cd                    # Same as cd ~ (no arguments means "go home")
cd -                  # Go back to the previous directory you were in
```

**Absolute vs. relative paths:**
An **absolute path** always starts with `/` and describes the full location from the root of the filesystem: `/var/log/auth.log`. It works the same no matter where you currently are.
A **relative path** describes a location starting from your current directory. If you are in `/var`, then `log/auth.log` points to the same file, but only from that specific location.

**Try it now:** Practice moving around:
```bash
pwd                    # Note where you are
cd /var/log            # Go to the log directory
pwd                    # Confirm you moved
ls                     # See what's here
cd ~                   # Go back home
cd -                   # Jump back to /var/log
cd ..                  # Go up to /var
pwd                    # Confirm: /var
```

---

### File Operations

**`cat`** — Con**cat**enate and print. Displays the content of a file in the terminal.

```bash
cat /etc/hostname           # Print the hostname
cat /etc/passwd             # Print all user accounts
cat file1.txt file2.txt     # Concatenate and print both files
```

For long files, `cat` floods the screen. Use `less` instead:
```bash
less /var/log/syslog        # Navigate with arrow keys, q to quit
```

**`touch`** — Creates an empty file, or updates the timestamp of an existing file.

```bash
touch notes.txt             # Creates notes.txt if it doesn't exist
```

**`mkdir`** — **M**a**k**e **Dir**ectory.

```bash
mkdir reports               # Create a single directory
mkdir -p lab/loot/ssh       # The -p flag creates all parent directories as needed
```

**`cp`** — **C**o**p**y.

```bash
cp file.txt backup.txt             # Copy to a new name in the same directory
cp file.txt /tmp/                  # Copy to another directory
cp -r project/ /tmp/project_bak/  # Copy an entire directory (-r = recursive)
```

**`mv`** — **M**o**v**e. Also used to rename files.

```bash
mv file.txt /tmp/                  # Move to /tmp
mv old_name.txt new_name.txt      # Rename (same directory, different name)
```

**`rm`** — **R**e**m**ove.

```bash
rm file.txt                        # Delete a file
rm -r folder/                      # Delete a directory and its contents (-r = recursive)
```

> Be careful with `rm`. There is no recycle bin on Linux. Once deleted, a file is gone. The `-f` flag (force) suppresses confirmation prompts. Never run `rm -rf /` or `rm -rf *` without being absolutely sure of your current directory.

**`echo`** — Prints text. Combined with redirection (`>` or `>>`), it writes to files.

```bash
echo "hello"                        # Print text to the terminal
echo "admin:pass123" > creds.txt    # Write to file (overwrites)
echo "more data" >> creds.txt       # Append to file (does not overwrite)
```

**Try it now:** Create a small scenario:
```bash
mkdir ~/practice
cd ~/practice
touch note.txt
echo "This is my first note" > note.txt
cat note.txt
cp note.txt backup.txt
ls -l
mv backup.txt archive.txt
ls -l
rm archive.txt
ls -l
```

---

### Users and File Permissions — Deep Dive

This section is long because permissions are the foundation of Linux security. Take the time to really understand it.

Every file and directory on Linux has three attributes: an **owner**, a **group**, and a set of **permissions** that control who can do what.

When you run `ls -l`, each line looks like this:

```
-rw-r--r-- 1 alice developers 2048 Mar 10 09:00 report.txt
```

Let's break it apart:

```
-  rw-  r--  r--    1    alice    developers    2048    Mar 10 09:00    report.txt
│   │    │    │     │      │          │          │         │              │
│   │    │    │     │      │          │          │         │              └── Filename
│   │    │    │     │      │          │          │         └── Last modified
│   │    │    │     │      │          │          └── File size in bytes
│   │    │    │     │      │          └── Group
│   │    │    │     │      └── Owner
│   │    │    │     └── Number of hard links
│   │    │    └── Others permissions: r-- (read only)
│   │    └── Group permissions: r-- (read only)
│   └── Owner permissions: rw- (read + write)
└── File type: - = file, d = directory, l = symbolic link
```

**The three permission categories:**
1. **Owner (user)** — the person who created the file
2. **Group** — all members of the file's group
3. **Others** — everyone else on the system

**The three permission types:**

| Letter | Number | Meaning for files | Meaning for directories |
|--------|--------|-------------------|------------------------|
| `r` | 4 | Can read the file content | Can list the directory contents |
| `w` | 2 | Can modify the file | Can create/delete files in the directory |
| `x` | 1 | Can execute the file as a program | Can enter the directory with `cd` |
| `-` | 0 | Permission not granted | Permission not granted |

**Calculating numeric permissions:** Add the values together for each category.

```
rwx = 4+2+1 = 7  (full access)
rw- = 4+2+0 = 6  (read and write)
r-x = 4+0+1 = 5  (read and execute)
r-- = 4+0+0 = 4  (read only)
--- = 0+0+0 = 0  (no access)
```

So `chmod 755 script.sh` means: owner gets `rwx` (7), group gets `r-x` (5), others get `r-x` (5).

**Common permission patterns you will see:**

| Numeric | Symbolic | Used for |
|---------|----------|----------|
| `644` | `rw-r--r--` | Regular files (owner writes, everyone reads) |
| `600` | `rw-------` | SSH private keys, sensitive configs |
| `755` | `rwxr-xr-x` | Executable scripts, directories |
| `700` | `rwx------` | Private directories |
| `777` | `rwxrwxrwx` | World-writable — a security problem |

**`chmod`** — **Ch**ange **mod**e. Modifies permissions.

```bash
chmod 600 id_rsa            # rw------- : only owner can read/write
chmod 644 notes.txt         # rw-r--r-- : owner writes, everyone reads
chmod 755 script.sh         # rwxr-xr-x : owner has full, others can read/execute
chmod +x script.sh          # Add execute permission
chmod o-w shared_file.txt   # Remove write permission from others
chmod g+rw project/         # Add read+write for the group
```

**`chown`** — **Ch**ange **own**er.

```bash
chown alice file.txt               # Change owner to alice
chown alice:developers file.txt    # Change owner AND group
sudo chown root:root file.txt     # Change to root (needs sudo)
```

**Why permissions are critical for security:**

An attacker who finds a world-writable script that is executed by root can inject commands into it and get root access. An SSH private key (`id_rsa`) with permissions looser than `600` will be rejected by the SSH client because it is considered insecure. A web directory set to `777` allows any user to upload files into it.

**Try it now:** Experiment with permissions:
```bash
touch ~/test_perms.txt
ls -l ~/test_perms.txt               # See default permissions
chmod 000 ~/test_perms.txt           # Remove ALL permissions
cat ~/test_perms.txt                 # Try to read — what happens?
chmod 644 ~/test_perms.txt           # Restore read/write for owner, read for others
cat ~/test_perms.txt                 # Works now
```

**SUID and SGID — special permission bits:**

Normally, when you run a program, it runs with **your** permissions. But if a binary has the **SUID** (Set User ID) bit set, it runs with the permissions of the file's **owner** instead.

Example: `/usr/bin/passwd` is owned by root and has the SUID bit set. This allows any user to run it and change their own password, even though the password file (`/etc/shadow`) is only writable by root. The `s` in the permission string indicates SUID:

```bash
ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 59640 /usr/bin/passwd
#   ^ "s" instead of "x" = SUID is set
```

If an attacker finds a SUID binary with a vulnerability, they can exploit it to run commands as root. This is one of the most common privilege escalation techniques on Linux.

```bash
find / -perm -4000 2>/dev/null    # Find all SUID binaries on the system
```

**Try it now:** Run the command above. How many SUID binaries does your system have? Research what 2 or 3 of them do.

---

### Package Manager (apt)

`apt` is the package manager on Debian-based systems (Ubuntu, Kali). It downloads, installs, updates, and removes software from trusted repositories (online servers maintained by the distribution).

```bash
sudo apt update                  # Refresh the list of available packages (always do this first)
sudo apt install nmap            # Download and install nmap
sudo apt install wireshark curl  # Install multiple packages at once
sudo apt remove nmap             # Remove a package
sudo apt list --installed        # List everything currently installed
apt search wireshark             # Search for packages by name
```

> Always run `sudo apt update` before installing anything. Without it, apt uses its cached (possibly outdated) package list.

On Red Hat-based systems (CentOS, RHEL, Fedora), the equivalent is `yum` or `dnf`.

**Try it now:** Install the `tree` package and use it to visualise a directory:
```bash
sudo apt update
sudo apt install tree -y
tree /etc/ssh
```

---

### Text Editors

You will regularly need to edit configuration files in the terminal.

**nano** — simple and beginner-friendly. The keyboard shortcuts are shown at the bottom of the screen (the `^` symbol means Ctrl).

```bash
nano /etc/hosts
# Ctrl+O  → Save (write Out)
# Ctrl+X  → Exit
# Ctrl+W  → Search
# Ctrl+K  → Cut a line
# Ctrl+U  → Paste a line
```

**vim** — powerful but requires learning modes. Vim has two main modes:
- **Normal mode** (default when you open a file) — for navigating and issuing commands
- **Insert mode** — for typing text

```bash
vim myfile.txt
```

| Key | What it does |
|-----|-------------|
| `i` | Enter insert mode (you can now type) |
| `Esc` | Return to normal mode |
| `:w` | Save |
| `:q` | Quit |
| `:wq` | Save and quit |
| `:q!` | Quit without saving |
| `dd` | Delete the current line |
| `/pattern` | Search for "pattern" |
| `n` | Go to next search result |

**Try it now:** Create a small file with nano:
```bash
nano ~/hello.txt
```
Type a few lines, save with Ctrl+O, exit with Ctrl+X. Then view it with `cat ~/hello.txt`.

---

### Reading the Manual

Every command has a **manual page** (man page). This is your definitive reference for what a command does, what flags it supports, and examples of usage.

```bash
man ls          # Full manual for ls
man chmod       # Full manual for chmod
man find        # Full manual for find (very detailed and useful)
```

Inside `man`:
- Arrow keys or `j`/`k` to scroll
- `/word` to search for "word"
- `n` to go to the next match
- `q` to quit

Most commands also support a quick `--help` flag:

```bash
ls --help
nmap --help
curl --help
```

Getting comfortable with man pages is what separates someone who needs Google from someone who can work on a remote server with no internet. In a pentest or CTF, you often have no browser — just the terminal.

**Try it now:** Run `man ls` and find the flag for sorting files by modification time (most recent first). Then quit with `q`.

---

## Exercises

The exercises for this module are in the separate drill file. These are short checks to make sure you understood the theory above before moving on.

> [Go to the drills](./drill.md)
