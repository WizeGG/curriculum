# 04 — Piping & Redirection: Drills

- **Type**: Drills
- **Challenge type**: Solo

---

## Part 1 — Understanding the Three Streams

1. Run this command: `ls /home /nonexistent`
   - Which output appears on stdout (normal results)?
   - Which output appears on stderr (errors)?
   > Your answer:

2. Run the same command but suppress errors: `ls /home /nonexistent 2>/dev/null`
   - What changed in the output?
   > Your answer:

3. Run this command: `find /home -name "*.txt" 2>/dev/null`
   - What does the `2>/dev/null` do in this context?
   > Your answer:

---

## Part 2 — Basic Piping

4. Using a pipe, list all running processes and filter only those related to `ssh`.
   > Your command:

5. Using pipes, find all lines in `/etc/passwd` that contain the word `bash`.
   > Your command:

6. List all files in `/usr/bin` and count how many there are (use pipe and `wc -l`).
   > Your command and count:

7. Display the content of `/etc/passwd` piped through `sort` to show usernames alphabetically.
   > Your command:

8. Create a pipeline that:
   - Reads `/etc/passwd`
   - Extracts usernames (field 1)
   - Counts the total number of users
   > Your command and count:

9. Chain three commands: `cat /etc/hostname | tr a-z A-Z | wc -c`
   - What does each command do?
   - What is the final output?
   > Your answer:

---

## Part 3 — Output Redirection (>)

10. Redirect the output of `ls -la /var/log` to a file called `log_audit.txt` in your home directory.
    > Your command:

11. Display the contents of your newly created `log_audit.txt` file. Did redirection work?
    > Your command and result:

12. Create a new file called `system_info.txt` containing the output of `uname -a`.
    > Your command:

13. Redirect the output of `date` to a file called `timestamp.txt`. Check its contents.
    > Your command and result:

---

## Part 4 — Append Redirection (>>)

14. Append the output of `whoami` to `log_audit.txt` (without overwriting previous contents).
    > Your command:

15. Append the output of `date` to `log_audit.txt`.
    > Your command:

16. Display the entire contents of `log_audit.txt`. Does it contain both appended lines?
    > Your command and result:

17. Create two separate commands:
    - First, create a file with `echo "line 1" > myfile.txt`
    - Then, append to it with `echo "line 2" >> myfile.txt`
    - Display the result
    > Your commands and result:

---

## Part 5 — Error Redirection (2>)

18. Run this command that produces an error: `find /root -type f 2> errors.txt`
    - Your stdout (results) should appear on screen
    - Your stderr (errors) should go to errors.txt
    - Check the contents of `errors.txt`
    > Your command and result:

19. Run `find / -name "secret" 2>/dev/null` (without saving errors)
    - Compare this to `find / -name "secret"` (without suppression)
    - What is different?
    > Your answer:

20. Redirect only errors (not output) to a file:
    `ls /home /nonexistent 2> error_output.txt`
    - What appears on your screen?
    - What appears in error_output.txt?
    > Your answer:

---

## Part 6 — Combined Redirection (> and 2>)

21. Run a command that produces both output and errors, redirecting them to separate files:
    `find /home /root -name "*.txt" 2> errors.txt 1> results.txt`
    - Check what's in results.txt
    - Check what's in errors.txt
    > Your commands and results:

22. Redirect both stdout and stderr to the SAME file:
    `find / -name "*.log" 2>&1 > all_output.txt`
    - Display the first 10 lines of the file
    > Your command and result:

23. Using the shorthand syntax `&>`, redirect everything:
    `ls /tmp /nonexistent &> combined.txt`
    - Display the contents
    > Your command and result:

---

## Part 7 — Input Redirection (<)

24. Create a file called `numbers.txt` with these contents:
    ```
    5
    2
    8
    1
    9
    ```
    Then use `sort` with input redirection: `sort < numbers.txt`
    > Your command and result:

25. Count the lines in `numbers.txt` using input redirection:
    `wc -l < numbers.txt`
    > Your command and result:

---

## Part 8 — Piping Errors (stderr through a pipe)

26. This command does NOT work as expected: `find / -name "id_rsa" | grep "id_rsa"`
    - Why don't the error messages get piped to grep?
    > Your answer:

27. Fix the command so that both stdout AND stderr are piped:
    `find / -name "id_rsa" 2>&1 | grep -v "Permission denied"`
    - What does `2>&1` do here?
    > Your answer:

---

## Part 9 — Named Pipes (FIFOs)

28. Create a named pipe (FIFO) called `TestPipe`:
    > Your command:

29. In one terminal, write to the named pipe in the background:
    `echo "Hello from pipe" > TestPipe &`
    In another terminal, read from it:
    `cat < TestPipe`
    - Did you receive the message?
    > Your commands and result:

30. Clean up by removing the named pipe:
    `rm TestPipe`
    > Your command:

---

## Part 10 — tee (Save AND Display)

31. Run a command and use `tee` to save output while displaying it:
    `echo "test message" | tee output_and_display.txt`
    - Did you see it on screen?
    - Check that it also saved to the file
    > Your commands and result:

32. Use `tee` with append flag to add multiple lines to a file:
    ```bash
    echo "line 1" | tee result.txt
    echo "line 2" | tee -a result.txt
    cat result.txt
    ```
    - What is the difference between `tee` and `tee -a`?
    > Your answer:

33. Pipe the output of `grep "Accepted" /var/log/auth.log` through `tee` to both display AND save to `accepted_logins.txt`.
    > Your command:

---

## Part 11 — Complex Pipelines (Real Security Analysis)

34. Build a pipeline that counts the number of running processes:
    `ps aux | wc -l`
    - How many processes are running?
    > Your command and count:

35. Find all listening TCP ports:
    `ss -tuln | grep LISTEN`
    - List them
    > Your command and result:

36. Extract and sort unique usernames from `/etc/passwd`:
    `cat /etc/passwd | cut -d: -f1 | sort | uniq`
    - Show the first 5
    > Your command and result:

37. Build a pipeline to find the most common HTTP status codes in a web log:
    `awk '{print $9}' /var/log/apache2/access.log | sort | uniq -c | sort -rn | head -5`
    - What are the top 5 status codes?
    > Your command and result:

---

## Part 12 — Security Analysis Pipelines

38. From `/var/log/auth.log`, extract all IP addresses that had failed login attempts, count occurrences, and show top 5:
    `grep "Failed" /var/log/auth.log | grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | sort | uniq -c | sort -rn | head -5`
    > Your command and result:

39. Count how many times each user appears in `/var/log/auth.log` using a pipeline:
    `grep "user=" /var/log/auth.log | grep -oE "user=[^ ]*" | sort | uniq -c | sort -rn | head -10`
    > Your command and result:

40. Find all open files in `/etc` that contain "ssh" and show their details:
    `find /etc -name "*ssh*" -type f 2>/dev/null | head -10`
    > Your command and result:

41. Create a security report: Find all users with bash shells, count them, and save to a file:
    `grep "/bin/bash" /etc/passwd | cut -d: -f1 | tee bash_users.txt | wc -l`
    - How many bash users are there?
    > Your command and count:

---

## Part 13 — Combining Multiple Techniques

42. Build a command that:
    - Reads `/var/log/auth.log`
    - Finds successful logins
    - Extracts the IP address
    - Counts unique IPs
    - Sorts by frequency
    - Shows only the top 3
    > Your command and result:

43. Find all `.conf` files in `/etc`, count them, and save the result:
    `find /etc -name "*.conf" -type f 2>/dev/null | tee conf_files.txt | wc -l`
    > Your command and count:

44. Compare two versions of a file and show only the differences:
    `diff /etc/passwd.bak /etc/passwd 2>/dev/null | tee changes.txt | head -10`
    > Your command and result:

---

## Part 14 — Real-World Scenarios

45. You need to audit SSH activity. Build a pipeline that:
    - Reads `/var/log/auth.log`
    - Finds all SSH login attempts (successful and failed)
    - Extracts unique IPs
    - Sorts and counts them
    - Shows top 10
    > Your command and result:

46. Find all files modified in the last 24 hours, save them to a report, AND display:
    `find /home -type f -mtime -1 2>/dev/null | tee recent_files.txt | wc -l`
    > Your command and count:

47. Security incident response: Extract all unique user accounts from `/etc/passwd`, count them, and identify which accounts are service accounts (shell is `/bin/false` or `/usr/sbin/nologin`):
    > Your commands and result:

48. Analyze web server logs to find the most common HTTP methods used:
    `awk '{print $6}' /var/log/apache2/access.log | cut -d' ' -f2 | sort | uniq -c | sort -rn | head -5`
    > Your command and result:

---

## Part 15 — Challenge Exercises

49. Build a single pipeline that:
    - Reads `/var/log/auth.log`
    - Finds all failed password attempts
    - Extracts the IP address (field 11)
    - Counts occurrences
    - Sorts by frequency (highest first)
    - Displays top 5 AND saves to `attack_report.txt` simultaneously
    > Your command:

50. Create a system security audit command that:
    - Finds all SUID binaries (from module 02: `find / -perm -4000`)
    - Saves them to `suid_binaries.txt`
    - Counts how many there are
    - Shows dangerous ones (owned by root)
    > Your commands:

51. Build a complete IOC extraction pipeline that:
    - Reads a log file
    - Extracts all IP addresses
    - Removes duplicates
    - Counts frequency
    - Sorts by highest occurrence
    - Saves to `extracted_iocs.txt` AND displays
    > Your command:

52. Monitor for suspicious file creation: Find all files created in the last 30 minutes in `/tmp`, show details, and save to a report:
    `find /tmp -type f -mmin -30 2>/dev/null | xargs ls -la | tee suspicious_files.txt`
    > Your command and result:

---

## Part 16 — Comprehensive Challenge

53. Flag — Build a multi-step security analysis that combines modules 02, 03, and 04:
    - Find all `.log` files in `/var/log` (module 02 skill)
    - Extract failed SSH attempts from them (module 03 skill)
    - Count attacking IPs and sort by frequency (module 03 skill)
    - Save results to `security_audit.txt` AND display top 5 (module 04 skill)

    > Your command(s):

54. Challenge — Create a complete network reconnaissance report:
    - Find all listening ports using `ss -tuln`
    - Extract port numbers
    - Identify which services are listening
    - Save to a report file
    - Display results
    > Your commands:

55. Advanced — Build a forensic timeline: Extract modification times from `/var/log`, sort chronologically, and identify suspicious patterns:
    > Your command(s):
