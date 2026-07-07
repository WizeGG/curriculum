# 03 — Text Manipulation: Drills

- **Type**: Drills
- **Challenge type**: Solo

---

## Part 1 — Reading Files

1. Display the last 25 lines of `/var/log/auth.log`.
   > Your command:

2. Display the first 10 lines of `/etc/passwd`.
   > Your command:

3. Open `/var/log/auth.log` in live view mode. What command are you running? (You don't need to wait, just show the command.)
   > Your command:

4. Count the total number of lines in `/var/log/auth.log`.
   > Your command and count:

5. Count the total number of lines in `/var/log/syslog`.
   > Your command and count:

---

## Part 2 — grep Basics

6. Find all lines in `/var/log/auth.log` that contain the word `"Failed"`.
   > Your command:

7. Find all lines in `/var/log/auth.log` that contain `"Accepted"` (successful logins).
   > Your command:

8. Search `/etc/ssh/sshd_config` for all lines that do NOT start with `#` (non-comment lines).
   > Your command:

9. Search for the word `"error"` (case-insensitive) in `/var/log/syslog`.
   > Your command:

10. Count how many lines in `/var/log/auth.log` contain the word `"sudo"`.
    > Your command and count:

---

## Part 3 — grep with Context

11. Find all lines containing `"Failed"` in `/var/log/auth.log` and display 2 lines AFTER each match.
    > Your command:

12. Find all lines containing `"Accepted"` in `/var/log/auth.log` and display 1 line BEFORE each match.
    > Your command:

13. Find all lines containing `"CRON"` in `/var/log/syslog` with 3 lines of context (before and after).
    > Your command:

---

## Part 4 — grep with Regular Expressions (Regex)

14. Find all lines in `/etc/passwd` that start with a lowercase letter.
    > Your command:

15. Find all lines in `/etc/passwd` that end with `"/bash"` (users with bash shell).
    > Your command and count:

16. Find lines in `/etc/passwd` that contain a username starting with `"a"` and ending with `"t"`.
    > Your command:

17. Using extended regex (`-E`), find all IPv4 addresses in `/var/log/auth.log`. (Hint: pattern like `[0-9]{1,3}\.[0-9]{1,3}`)
    > Your command:

18. Find all lines that contain either the word `"error"` OR `"warning"` (case-insensitive) in `/var/log/syslog`. (Hint: use `-E` with `|`)
    > Your command:

---

## Part 5 — cut for Field Extraction

19. Extract only the usernames from `/etc/passwd` (field 1, delimiter `:`).
    > Your command:

20. Extract the username and home directory from `/etc/passwd` (fields 1 and 6).
    > Your command:

21. Extract the username and login shell from `/etc/passwd` (fields 1 and 7).
    > Your command:

22. Identify all users who have `/bin/bash` as their login shell.
    > Your command and count:

23. Identify all service accounts whose shell is set to `/bin/false` or `/usr/sbin/nologin`.
    > Your command and results:

24. Extract the first field (IP address) from a web server access log. (Hint: delimiter is space, field 1)
    > Your command:

---

## Part 6 — sort and uniq

25. Sort the usernames from `/etc/passwd` alphabetically.
    > Your command:

26. Extract usernames from `/etc/passwd`, sort them, and remove duplicates.
    > Your command:

27. Create a file with the following content:
    ```
    apple
    banana
    apple
    cherry
    banana
    apple
    ```
    Then count how many times each fruit appears using `sort | uniq -c`.
    > Your command and results:

28. From the fruit file above, use `sort | uniq -c | sort -rn` to show fruits sorted by frequency (most common first).
    > Your command and results:

29. Show only the fruits that appear more than once.
    > Your command and results:

---

## Part 7 — wc (Word/Line Count)

30. Count the total number of lines in `/var/log/auth.log`.
    > Your command and count:

31. Count the number of words in `/etc/hostname`.
    > Your command and count:

32. Count how many failed SSH login attempts appear in `/var/log/auth.log`.
    > Your command and count:

33. Count how many successful SSH logins appear in `/var/log/auth.log`.
    > Your command and count:

---

## Part 8 — sed (Find and Replace)

34. Create a file called `secrets.txt` with the following content:
    ```
    username=admin password=Sup3r$ecret123
    username=bob password=MyP@ssw0rd
    ```
    Using `sed`, redact all passwords. Replace `password=anything` with `password=REDACTED`.
    > Your command and result:

35. Display lines 5 to 10 from `/var/log/auth.log` using `sed`.
    > Your command:

36. Using `sed`, remove all comment lines (starting with `#`) from `/etc/ssh/sshd_config`. How many non-comment lines are there?
    > Your command and count:

37. Using `sed`, replace all occurrences of `"root"` with `"REDACTED"` in `/etc/passwd` (output to screen, don't modify the file).
    > Your command:

38. Using `sed`, delete all blank lines from a file. (Hint: `/^$/d`)
    > Your command:

---

## Part 9 — awk Field Extraction

39. Extract the first field (IP address) from `/var/log/apache2/access.log` using `awk`. Show the first 5.
    > Your command:

40. Extract the username (field 1, delimiter `:`) from `/etc/passwd` using `awk -F ":"`. Show first 5.
    > Your command:

41. Extract both username (field 1) and UID (field 3) from `/etc/passwd`. Use `awk -F ":"` and print both fields separated by a tab.
    > Your command:

42. Using `awk -F ":"`, find all users whose shell is `/bin/bash`.
    > Your command:

43. Using `awk` on `/etc/passwd`, find all users whose UID is greater than 1000 (regular users, not system accounts).
    > Your command:

---

## Part 10 — awk Pattern Matching

44. From `/var/log/auth.log`, extract lines containing `"Failed"` and print field 11 (the IP address).
    > Your command:

45. From a web server log, extract all lines with HTTP status code `"404"` (Not Found) and print the IP address (field 1).
    > Your command:

46. Count how many lines in `/var/log/syslog` contain the word `"error"`.
    > Your command and count:

---

## Part 11 — awk Aggregation and Counting

47. Create a file `access.log` with this content:
    ```
    192.168.1.100 GET /index.html 200
    192.168.1.101 GET /admin.php 403
    192.168.1.100 GET /blog.html 200
    192.168.1.102 GET /index.html 200
    192.168.1.100 GET /secret.txt 403
    ```
    Using `awk`, count how many times each IP appears. (Hint: `awk '{count[$1]++} END {...}`)
    > Your command and results:

48. From `/var/log/auth.log`, count how many times each unique IP tried to log in. Show the top 5 most active IPs.
    > Your command:

49. Using `awk`, sum all the numbers in a file (file contains one number per line).
    > Your command:

---

## Part 12 — Complex Pipelines

50. From `/var/log/auth.log`, find all failed login attempts, extract the source IP address, count occurrences, and show the top 10 most active attacking IPs.
    > Your command:

51. From `/etc/passwd`, list all users who have a bash shell, sorted alphabetically.
    > Your command:

52. Count how many unique IP addresses attempted to log in to your system (from `/var/log/auth.log`).
    > Your command and count:

53. Extract all failed login IPs, count them, and output the result sorted by count (highest first). Save to `~/attack_report.txt`.
    > Your command:

---

## Part 13 — diff (Comparing Files)

54. Create two files:
    - `config.old`: A copy of `/etc/ssh/sshd_config`
    - `config.new`: Another copy of the same file

    Then compare them with `diff`. What do you see?
    > Your command and result:

55. Create a modified version of `/etc/passwd` where you change one line, then use `diff` to identify what changed.
    > Your command and result:

---

## Part 14 — Real-World Security Scenarios

56. You suspect someone has been repeatedly trying to log in as the `"admin"` user. Extract all failed login attempts for `"admin"` from `/var/log/auth.log`.
    > Your command:

57. Security audit: Find all users with a real login shell (not `/bin/false` or `/usr/sbin/nologin`) and list them.
    > Your command:

58. Detect privilege escalation attempts: Find all `"sudo"` commands executed in `/var/log/auth.log`. Who ran sudo most frequently?
    > Your command:

59. Create a CSV file with mock server logs:
    ```
    timestamp,ip,port,status
    2024-01-15T10:00:00,192.168.1.100,22,success
    2024-01-15T10:01:00,192.168.1.101,22,failed
    2024-01-15T10:02:00,192.168.1.100,22,success
    2024-01-15T10:03:00,203.0.113.50,80,blocked
    ```
    Using `awk -F ","`, extract IP addresses and status where status is `"failed"`.
    > Your command:

60. Build a one-liner pipeline that:
    - Reads `/var/log/auth.log`
    - Finds all failed login attempts
    - Extracts unique IP addresses
    - Counts how many times each IP appears
    - Sorts by frequency (highest first)
    - Shows only the top 5
    > Your command:

---

## Part 15 — Challenge Questions

61. Write a `sed` command that redacts all email addresses in a file. Replace them with `USER@REDACTED.COM`.
    > Your command:

62. Using `awk`, create a command that takes a CSV file and outputs only rows where the 3rd column (field 3) is greater than 100.
    > Your command:

63. Write a single command using `grep`, `cut`, `sort`, and `uniq -c` to find the most common web server status codes in an Apache access log.
    > Your command:

64. Flag — In `/var/log/auth.log`, find the IP address that has made the most failed login attempts. How many attempts did it make?
    > Your command and answer:

65. Challenge — Create a complete security analysis of your system:
    - Find all users with a real bash shell
    - For each, count how many times they appear in `/var/log/auth.log`
    - Show results sorted by frequency
    - (Hint: this requires nested commands or multiple steps)
    > Your command(s):
