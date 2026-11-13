<div align="center">

# 🕵️ Write-Ups

**My notes, methods and walkthroughs from the hacking trenches.**

[![TryHackMe](https://img.shields.io/badge/TryHackMe-Crashkid-212C42?style=for-the-badge&logo=tryhackme&logoColor=white)](https://tryhackme.com/p/Crashkid)
[![Focus](https://img.shields.io/badge/Focus-Pentesting-red?style=for-the-badge)]()
[![Status](https://img.shields.io/badge/Status-Always_Learning-2ea44f?style=for-the-badge)]()

</div>

---

## 👋 About

Welcome! This folder collects my **write-ups for [TryHackMe](https://tryhackme.com/p/Crashkid)** rooms and challenges.

Every write-up documents how I approached a machine: what I enumerated, what worked, what didn't, and what I learned along the way. Writing things down helps me understand them better, and maybe it helps you too.

More about me: [www.itschrank.de](https://www.itschrank.de)

> ⚠️ **Spoiler warning:** These write-ups contain solutions. Try the room yourself first. Getting stuck is part of the fun, and it's where most of the learning happens.

---

## 🧰 Tools I use

| Phase | Tools |
|-------|-------|
| 🔍 **Recon & Scanning** | `nmap`, `rustscan`, `masscan` |
| 🌐 **Web Enumeration** | `gobuster`, `ffuf`, `dirsearch`, `nikto` |
| 🕸️ **Web Exploitation** | `Burp Suite`, `sqlmap`, `curl` |
| 💥 **Exploitation** | `Metasploit`, `searchsploit`, custom scripts |
| 🔑 **Password Attacks** | `Hydra`, `John the Ripper`, `Hashcat` |
| 📡 **Network Analysis** | `Wireshark`, `tcpdump`, `netcat` |
| ⬆️ **Privilege Escalation** | `LinPEAS`, `WinPEAS`, `GTFOBins`, `LOLBAS` |
| 🔬 **Forensics & Misc** | `CyberChef`, `strings`, `binwalk`, `exiftool` |

*The list grows with every room.*

---

## 🧭 My general methodology

1. **Reconnaissance:** scan ports, identify services and versions
2. **Enumeration:** dig deep into every open service (web, SMB, FTP, SSH ...)
3. **Initial Access:** find and use a weakness to get a foothold
4. **Privilege Escalation:** move from low-privilege user to root/admin
5. **Post-Exploitation:** collect flags, understand the impact
6. **Documentation:** write it all down (you're reading the result)

---

## 📚 Write-Ups

| Room | Difficulty | Topics | Link |
|------|:----------:|--------|:----:|
| [Surfer](https://tryhackme.com/room/surfer) | 🟢 Easy | web, weak credentials, SSRF, Burp Suite | [Write-Up](./surfer/surfer.md) |

<!--
Template for new entries:
| [Room Name](https://tryhackme.com/room/ROOM) | 🟢 Easy / 🟡 Medium / 🔴 Hard | nmap, web, privesc | [Write-Up](./room-name.md) |
-->

---

## 🗂️ Structure of a write-up

Each write-up follows roughly the same format:

- **Overview:** room, difficulty, goal
- **Enumeration:** what I found and how
- **Exploitation:** how I got in
- **Privilege Escalation:** how I got to root
- **Flags:** (redacted where sensible)
- **Lessons learned:** what I take away from it

---

## ⚖️ Disclaimer

All techniques shown here are used **only** in legal, authorized environments such as TryHackMe rooms. Do not use them against systems you don't own or don't have explicit permission to test. Hack responsibly. 🔒

---

## 📬 Contact

Questions, feedback or ideas for improvements? Get in touch:

- 📧 [crashkid@mailbox.org](mailto:crashkid@mailbox.org) · [PGP Key](https://www.itschrank.de/36596F92402B857BB53C9818D20210251972D067.asc)
- 🐘 [Mastodon: @crashkid@chaos.social](https://chaos.social/@crashkid)
- 🌐 [www.itschrank.de](https://www.itschrank.de)

<div align="center">

*Happy hacking, and don't forget your coffee ☕*

</div>
