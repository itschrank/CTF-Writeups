# TryHackMe – GameZone Write-up

**Platform:** TryHackMe
**Room:** [GameZone](https://tryhackme.com/room/gamezone)
**Topics:** OSINT (reverse image search), SQL Injection, Burp Suite, SQLmap, hash cracking with John the Ripper, SSH tunneling, Metasploit, Remote Code Execution

> **Note:** Flags, credentials and hash values are intentionally omitted from this write-up.

---

## Overview

This is a great beginner-friendly room: it gives quick wins early on while still touching a nice variety of techniques, from OSINT and SQL injection all the way to tunneling and exploitation with Metasploit. Definitely worth doing if you want a room that covers a broad range of skills without feeling overwhelming.

---

## 1. OSINT – Identifying the Avatar

The first question didn't take long to answer, since I'm a long-time video game fan and recognized the avatar right away. For anyone who doesn't recognize it, a reverse image search (e.g. via [Google Images](https://images.google.com/?hl=de)) is an easy way to identify it.

---

## 2. SQL Injection on the Login Form

Connecting to the web server over HTTP presents a web interface. As described in the room, this login form is vulnerable to **SQL injection**. Entering the following as the username, with the password left empty, bypasses the login:

```
' or 1=1 -- -
```
<img width="418" height="593" alt="screen1" src="https://github.com/user-attachments/assets/09b27f7c-807b-4ce1-8491-4dcb27ccd04f" />



After a successful login, we get access to the portal.

<img width="780" height="303" alt="screen2" src="https://github.com/user-attachments/assets/d480e6e4-4c7e-4031-bf8e-783ebd7b5d9d" />


---

## 3. Capturing the Search Request with Burp Suite

For the next step we need **Burp Suite** and **SQLmap**, two tools that save a lot of manual work. We issue a search query on the portal and intercept it with Burp, then save the captured request as `request.txt`.

<img width="868" height="827" alt="screen3" src="https://github.com/user-attachments/assets/1a1446b6-1d87-45f4-afe6-a8f48053e49f" />


---

## 4. Dumping the Database with SQLmap

With the saved request, we switch to the terminal and run:

```bash
sqlmap -r request.txt --dbms=mysql --dump
```

Whenever SQLmap asks a follow-up question, we confirm with `yes`. The tool then does its work and dumps new information from the database, including a password hash.

<img width="662" height="266" alt="screen4" src="https://github.com/user-attachments/assets/30827be2-49b0-4eb4-945e-b2b63b19ac0e" />


---

## 5. Cracking the Hash with John the Ripper

We save the hash found in the previous step as `hash.txt` and crack it with **John the Ripper**, using the `rockyou.txt` wordlist:

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256
```

We let Kali do its work, and after a while John recovers the plaintext password.

<img width="782" height="192" alt="screen5" src="https://github.com/user-attachments/assets/7afb06d2-01a0-4dab-b949-c42bbb3fc34b" />


---

## 6. Tunneling to the Internal Web Interface

At this point we have a username and the matching password. According to the task, we now need to reach a second web interface that sits behind a firewall and isn't directly reachable. To get to it, we tunnel our traffic through SSH:

```bash
ssh -L 10000:localhost:10000 a*****@10.114.146.27
```

<img width="658" height="388" alt="screen6" src="https://github.com/user-attachments/assets/36c61589-5f53-425b-a1db-eb152bd8511c" />


With the tunnel in place, the internal web interface becomes reachable through `localhost:10000`.

<img width="795" height="354" alt="screen7" src="https://github.com/user-attachments/assets/c137573e-d52e-4f54-8264-c0a4c79e1eed" />


---

## 7. Logging In

I usually try a few default credentials before pulling out any tooling. A handful of common default logins did not work, but the username/password combination we recovered earlier in step 5 worked immediately.

<img width="831" height="595" alt="screen8" src="https://github.com/user-attachments/assets/7ad99a12-3a99-4793-b48d-60b8f14d1812" />


---

## 8. Gaining Root with Metasploit

The task now asks us to take over the server using **Metasploit** and find the root flag. I start Metasploit and look for an exploit matching the web interface:

```
search webmin
```

This immediately turns up a **Remote Code Execution** module. I load the exploit and pick a reverse shell payload.

<img width="896" height="633" alt="screen9" src="https://github.com/user-attachments/assets/b68183dc-16eb-49e1-be6f-7642fc44d7de" />


Running the exploit against the web interface gives us a reverse shell, and the service appears to be running as **root**.

<img width="515" height="167" alt="screen10" src="https://github.com/user-attachments/assets/a679182e-ce66-44e0-b08a-74bab918bd8a" />


---

## 9. Finding the Flag

Navigating the filesystem, the flag isn't hidden very deep and turns up directly in the root directory.

<img width="306" height="103" alt="screen11" src="https://github.com/user-attachments/assets/7a3b2a76-9718-4668-ac4a-6e17c63a40ff" />


Room complete. This one was a lot of fun, thanks for a great room!

---

## Lessons Learned

- **SQL injection on login forms:** A classic `' or 1=1 -- -` payload can still bypass authentication when user input is concatenated directly into SQL queries instead of using parameterized queries.
- **Automated tools save time, but understanding matters:** SQLmap and John the Ripper did the heavy lifting here, but knowing *why* the SQL injection works and *how* to intercept the right request with Burp was necessary to use them effectively.
- **Weak password hashing and reuse:** A crackable hash and credential reuse between services turned one low-privilege foothold into access on a second, firewalled system.
- **Internal services still need to be secured:** Putting a service behind a firewall isn't enough if an attacker can tunnel through SSH once they have valid credentials.
- **Keep known vulnerable software patched:** The Webmin RCE used here is a known, public exploit. Outdated admin panels running as root are a high-value target.

---

## Tools Used

- Web browser
- Burp Suite (intercepting the search request)
- SQLmap (database enumeration and dumping)
- John the Ripper + rockyou.txt (hash cracking)
- SSH (local port forwarding / tunneling)
- Metasploit (Webmin RCE exploitation)
