# TryHackMe – RootMe Write-up

**Platform:** TryHackMe
**Room:** [RootMe](https://tryhackme.com/room/rootme)
**Topics:** nmap, Gobuster, file upload bypass, reverse shell, SUID privilege escalation, GTFOBins

> **Note:** Flags are intentionally omitted from this write-up.

---

## Overview

A simple but genuinely fun room. It's a good showcase of the classic path from enumeration to a file upload bypass, a reverse shell, and SUID-based privilege escalation, without taking too long to complete.

---

## 1. Initial Enumeration with Nmap

After booting the machine, we start by gathering basic information with an **nmap** scan.

<img width="808" height="222" alt="screen1" src="https://github.com/user-attachments/assets/ee6b7603-cd8a-4b59-a975-adcd9a37d6c7" />


---

## 2. Directory Enumeration with Gobuster

Next, we need to find out which directories and paths exist on the web server, so we run **Gobuster** against it.

<img width="676" height="476" alt="screen2" src="https://github.com/user-attachments/assets/8bd7cf73-2e30-405d-b092-706880b78d72" />


---

## 3. First Upload Attempt

The scan reveals an upload feature. I grab the `php-reverse-shell.php` from `/usr/share/webshells/php` on my Kali machine, configure it with my IP and port, and start a listener (`penelope`) to catch the shell. Unfortunately, the upload is rejected, `.php` files are blocked.

<img width="636" height="556" alt="screen3" src="https://github.com/user-attachments/assets/67198a00-0983-41f3-8b89-181c82029a23" />


---

## 4. Bypassing the Upload Filter

I rename the file to `shell.phtml`, and this time the upload succeeds. I navigate to the upload directory and open the uploaded file to trigger it.

<img width="548" height="372" alt="screen4" src="https://github.com/user-attachments/assets/cd650e26-45ff-4c2e-81d1-a050a478aaac" />


This gives us a working shell, and we're logged in.

<img width="944" height="218" alt="screen5" src="https://github.com/user-attachments/assets/63007416-edc9-45fb-91ba-7f4a66333940" />


---

## 5. Finding the User Flag

I look for the user flag but can't find it right away, so I keep exploring the filesystem. Checking back in the web server's directory, the `user.txt` turns up there.

<img width="421" height="60" alt="screen6" src="https://github.com/user-attachments/assets/ef87e0ed-30cb-457b-8878-b39ffbdef6b7" />


---

## 6. Privilege Escalation via SUID

To escalate privileges, the room hints that this should work via a SUID binary. I list all SUID binaries with:

```bash
find / -perm -4000 2>/dev/null
```

`python` immediately stands out. Checking [GTFOBins](https://gtfobins.org/) for Python's SUID escalation technique gives the following command:

```bash
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

Running it gives a root shell.

<img width="495" height="74" alt="screen7" src="https://github.com/user-attachments/assets/d2b47da7-68a6-4d9d-9c1a-e7ccf8c38d2d" />


The flag is, as usual, located under `/root/`.

<img width="247" height="65" alt="screen8" src="https://github.com/user-attachments/assets/a69e11c1-c52c-4bcb-9740-25fa335aa1fa" />


Room complete.

---

## Lessons Learned

- **Blocked extensions aren't the whole story:** When `.php` is blocked, trying alternative extensions that are still executed by the server (`.phtml`, `.php5`, `.pht`, etc.) is a quick and often effective bypass.
- **Don't assume the flag's exact location:** The user flag wasn't in the first place I checked. Backtracking to the web server's own directory (where the shell was uploaded) paid off.
- **SUID binaries are a classic escalation vector:** `find / -perm -4000` is a fast way to list SUID binaries, and GTFOBins is an excellent reference for turning a SUID binary into a shell.
- **Keep a local webshell collection and listener ready:** Having `php-reverse-shell.php` and a listener (e.g. `penelope`) prepared ahead of time speeds up this kind of workflow noticeably.

---

## Tools Used

- Nmap (enumeration)
- Gobuster (directory enumeration)
- PHP reverse shell (`php-reverse-shell.php`) + listener (`penelope`)
- GTFOBins (SUID privilege escalation reference)
