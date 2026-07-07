# 01 — Terminal Basics: Drills

- **Type**: Drills
- **Challenge type**: Solo

---

## Instructions

Work through each exercise in order. Write down the command you used and the output or answer.

---

## Part 1 — The System

1. What kernel version is running on your machine?
   > Your command & output:

2. What distribution are you running? What version?
   > Your command & output:

3. What is the architecture of your system (e.g. x86_64, arm64)?
   > Your command & output:

4. What is the hostname of your machine?
   > Your command:

5. What is the uptime of your system? How long has it been running?
   > Your command:

---

## Part 2 — Users

6. What is your current username?
   > Your command:

7. What groups does your user belong to?
   > Your command & output:

8. How many user accounts exist on this system? (Hint: count the lines in `/etc/passwd`)
   > Your command & count:

9. Which users have `/bin/bash` as their default shell? List their usernames.
   > Your command:

10. What is the UID and GID of the `root` account?
    > Your command & answer:

11. Run `id`. What is the difference between your UID and your GID?
    > Your answer:

12. Run `sudo whoami`. What does it return, and why?
    > Your answer:

---

## Part 3 — Navigation

13. Print your current directory. Navigate to `/var/log`, confirm with `pwd`, then go back home using `~`.
    > Your commands:

14. From your home directory, navigate to `/etc/ssh` using an **absolute path**. Then navigate back to your home using a **relative path** (using `../..`).
    > Your commands:

15. Use `cd -` to jump between two directories. Explain in your own words what `cd -` does.
    > Your commands & explanation:

16. What is the difference between `cd /tmp` and `cd tmp`? When would the second one fail?
    > Your answer:

17. List the contents of `/var/log` in long format, with human-readable file sizes, including hidden files.
    > Your command:

18. Without navigating there, list the contents of `/etc/ssh` from your home directory.
    > Your command:

---

## Part 4 — File Operations

19. Create a directory structure `~/lab/evidence/network` using a single `mkdir` command.
    > Your command:

20. Create three empty files in `~/lab/evidence/`: `note1.txt`, `note2.txt`, `note3.txt`.
    > Your command:

21. Write the text `Suspicious IP: 10.0.0.55` into `note1.txt` using `echo`.
    > Your command:

22. Append the text `Port: 4444` to `note1.txt` without overwriting the first line.
    > Your command:

23. Print the contents of `note1.txt` to verify both lines are present.
    > Your command & output:

24. Copy `note1.txt` to the `network/` subdirectory.
    > Your command:

25. Rename `note2.txt` to `discarded.txt`.
    > Your command:

26. Delete `discarded.txt`.
    > Your command:

27. Delete the entire `evidence/` directory and everything inside it.
    > Your command:

28. Recreate the `~/lab/evidence/` directory. What command do you use?
    > Your command:

---

## Part 5 — Permissions

29. Create a file `~/lab/secret.txt`. Run `ls -l` on it. What are the default permissions?
    > Your command & output:

30. Convert the permission string from the previous exercise into its numeric form (e.g., `rw-r--r--` = `644`).
    > Your answer:

31. Change the permissions on `secret.txt` so that only the owner can read and write it. Nobody else should have any access.
    > Your command:

32. Verify the change with `ls -l`. What does the permission string look like now?
    > Your output:

33. Create a file `~/lab/scan.sh`. Write `#!/bin/bash` into it with `echo`. Try to run it with `./scan.sh`. What happens?
    > Your commands & what happened:

34. Make `scan.sh` executable and run it again.
    > Your commands:

35. Change the owner of `secret.txt` to `root` using `sudo chown`. Then try to `cat` it as your normal user. What happens?
    > Your commands & answer:

36. Change it back to your own user.
    > Your command:

37. Explain the difference between `chmod 644` and `chmod 600` in your own words. When would you use each?
    > Your answer:

38. What does `chmod 777` mean? Why is it considered dangerous in a production environment?
    > Your answer:

39. Find all SUID binaries on your system. How many are there? List 3 and research what they do.
    > Your command, count, and explanation:

40. Find all world-writable files outside of `/tmp` and `/proc`. Are there any?
    > Your command & results:

---

## Part 6 — Package Manager

41. Update the package list on your machine.
    > Your command:

42. Install `nmap` and `net-tools`. Verify both are installed using `which`.
    > Your commands:

43. What version of nmap did you install?
    > Your command & answer:

44. Search for a package called `tcpdump`. Is it already installed? If not, install it.
    > Your commands:

45. Remove `net-tools`. Verify it is gone.
    > Your commands:

---

## Part 7 — Manual and Help

46. Use `man ls` to find the flag that sorts files by **modification time** (most recent first).
    > The flag:

47. Use `man ls` to find the flag that shows file sizes in **human-readable** format.
    > The flag:

48. Use `man chmod` to find the symbolic notation for adding execute permission to the group.
    > The command:

49. Use `man find` to find the flag that searches by **file type**. What letter represents a regular file?
    > Your answer:

50. Use `man grep` to find the flag for **case-insensitive** search. Test it on `/etc/passwd` searching for your username in uppercase.
    > The flag & your test command:

---

## Part 8 — Putting It Together

51. In a single line, create a directory `~/audit`, navigate into it, create a file `findings.txt`, write `Audit started` into it, and print the file. Use `&&` to chain the commands.
    > Your command:

52. List all files in `/etc` that start with the letter `s`. How many are there?
    > Your command & count:

53. Create a file `~/audit/users.txt` that contains only the usernames from `/etc/passwd` (one per line). Use `cut` to extract the first field (delimiter is `:`).
    > Your command:

54. Display the first 5 lines of `/var/log/syslog` using `head`, and the last 5 using `tail`.
    > Your commands:

55. You are investigating a machine and want a quick overview. Run all of the following and write down what each one tells you:
    - `uname -a`
    - `id`
    - `hostname`
    - `uptime`
    - `cat /etc/os-release`
    - `df -h`
    - `free -h`
    - `who`
    > For each command, write what it tells you and why it would be useful during an investigation.
