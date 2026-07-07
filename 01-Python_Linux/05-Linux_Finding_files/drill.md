# 02 — Finding Files: Drills

- **Type**: Drills
- **Challenge type**: Solo

---

## Part 1 — Understanding locate vs find

1. Create a new file called `secret-audit-2024.txt` in your home directory using `touch`. Then immediately run `locate secret-audit-2024.txt`.
   > Your command and result:

2. Run `sudo updatedb` to refresh the database. Then run `locate secret-audit-2024.txt` again. Did it appear this time?
   > Your command and result:

3. Now use `find` to search for the same file starting from your home directory. Does `find` find it immediately (before running updatedb)?
   > Your command and result:

4. Explain in your own words: Why does `locate` not find newly created files immediately, but `find` does?
   > Your answer:

---

## Part 2 — Basic find by Name

5. Search for all files ending in `.conf` anywhere on the system. How many do you find?
   > Your command and count:

6. Search for all files in `/etc` (not subdirectories, just top-level) whose name contains the word `ssh`.
   > Your command and results:

7. Find all `.log` files in `/var/log` that start with the letter `a`.
   > Your command and results:

8. Search for all files named `README` or `README.md` anywhere on the system.
   > Your command and results:

9. Find all files whose name contains `2024` anywhere on the system.
   > Your command and results:

---

## Part 3 — Searching by File Type

10. List all directories (not files) named `.ssh` on the system.
    > Your command and results:

11. Find all symbolic links in `/usr/bin`. How many are there?
    > Your command and count:

12. Find all regular files in `/etc` (not subdirectories) that end in `.d`.
    > Your command and count:

13. Search for all named pipes (FIFOs) on your system. (Hint: `-type p`)
    > Your command and results:

14. Find all sockets on the system. (Hint: `-type s`)
    > Your command and results:

---

## Part 4 — Permission Auditing (Critical for Security)

15. Find all SUID binaries on your system. List the first 10.
    > Your command and results:

16. Using `ls -l`, examine one of the SUID binaries you found. Can you identify the `s` in the permission string?
    > Your command and result:

17. Find all SGID binaries on your system. How many are there?
    > Your command and count:

18. Find all files that are world-writable (any file where "others" have write permission). Exclude `/proc` and `/sys` to reduce noise.
    > Your command and results:

19. Search for files with permission `777` (rwxrwxrwx — fully open). What does it mean when you find these?
    > Your command and answer:

20. Find all files in `/tmp` that are readable, writable, and executable by everyone (`-perm 777`).
    > Your command and results:

21. Find all files owned by root AND have the SUID bit set.
    > Your command and results:

22. Find all world-writable files in `/home` only (exclude `/sys` and `/proc`). What security risk do these represent?
    > Your command and answer:

---

## Part 5 — Searching by Ownership

23. Find all files in your home directory that are NOT owned by you.
    > Your command and results:

24. Search for all files on the system that are NOT owned by root, user "nobody", or "daemon".
    > Your command and results:

25. Find all orphaned files (files with no valid owner) on the system. What could this indicate?
    > Your command and answer:

26. Find all files owned by a specific user (pick any valid user on your system). How many files do they own?
    > Your command and count:

---

## Part 6 — Time-Based Searching (Forensics)

27. Find all files in `/var/log` that were modified in the last 1 day.
    > Your command and results:

28. Find all files in `/tmp` that were modified in the last 60 minutes.
    > Your command and results:

29. Find all files modified more than 30 days ago in `/home`.
    > Your command and results:

30. Create a reference file with `touch`, then find all files on the system that are newer than that reference file.
    > Your commands and results:

31. Conduct a forensic search: Find all files anywhere on the system that were modified in the last 24 hours AND are larger than 1 MB. (Hint: use `-mtime` and `-size`)
    > Your command and results:

---

## Part 7 — Using -exec for Advanced Actions

32. Find all `.conf` files in `/etc` and display detailed information (using `ls -l`) for each one.
    > Your command and results (show first 5):

33. Find all shell scripts (`.sh` files) in `/home` and display how many lines each one has.
    > Your command and results:

34. Find all files in `/tmp` that end with `.tmp` and print their file type using the `file` command.
    > Your command and results:

35. Find all `.log` files in `/var/log` that were modified in the last 24 hours and count the total number of lines across all of them. (Hint: use `-exec wc -l`)
    > Your command and result:

---

## Part 8 — Complex Multi-Condition Searches

36. Find all regular files in `/etc` that are NOT world-readable. (Hint: `-not -perm -o+r`)
    > Your command and results:

37. Find all SUID binaries in `/usr/bin` that are owned by root and were modified in the last 30 days.
    > Your command and results:

38. Find all files in `/tmp` that are both world-writable AND have been modified in the last hour.
    > Your command and results:

39. Find all files in `/home` that do NOT end with `.txt` AND do NOT end with `.pdf`.
    > Your command and results:

40. Find all symbolic links in `/usr/bin` that point to files outside of `/usr`.
    > Your command and results:

---

## Part 9 — Real-World Security Scenarios

41. You suspect a backdoor was installed in `/tmp` in the last 4 hours. Search for any executable files that were recently modified.
    > Your command and results:

42. Audit your system: Find all world-writable files that should NOT be writable by everyone. List them and explain the security risk.
    > Your command and analysis:

43. Search for all SSH private keys (`id_rsa`, `id_dsa`, `id_ecdsa`, etc.) anywhere on the system. Suppress permission errors.
    > Your command and results:

44. Find all password-related files (containing "password" or "passwd" in the name) and check their permissions. Are any world-readable?
    > Your command and analysis:

45. Locate all `.bash_history` files across the system. What information could an attacker extract if they found these?
    > Your command and answer:

---

## Part 10 — Challenge Questions

46. Security incident response: A file was uploaded to `/tmp` by an attacker. You know it was uploaded sometime in the last 2 hours. Build a `find` command to locate all files matching this pattern. What command would you run?
    > Your command:

47. You need to fix incorrect permissions on a group of files. Find all files in `/home` with permission `777` and use `-exec` to change them to `644` in a single command.
    > Your command:

48. Build a find command that searches for files with SUID set, owned by root, in `/usr/local/bin`. Suppress permission denied errors and show detailed information for each match.
    > Your command:

49. Flag — Find a hidden file (filename starts with a dot) in your `/home` directory that was modified in the last week. What is its full path?
    > Your command and flag:

50. Challenge — Create a single `find` command that:
    - Searches from `/home`
    - Finds regular files only
    - With permission 777
    - Modified in the last 30 days
    - AND owned by a non-root user
    - Execute `ls -la` on each result
    - Suppress all error messages
    > Your command:
