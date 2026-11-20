# TryHackMe – RevilCorp Write-up

**Platform:** TryHackMe
**Room:** [RevilCorp](https://tryhackme.com/room/revilcorp)
**Topics:** Malware analysis / Digital Forensics and Incident Response (DFIR), Mandiant Redline, ransomware triage

> **Note:** Flags, hash values and other sensitive answers are intentionally omitted from this write-up.

---

## Overview

This room provides a Windows VM with a pre-collected triage file on the desktop, and asks us to analyze a ransomware incident using **Mandiant Redline**. The goal is to reconstruct what happened on the compromised machine: which file the user opened, what the ransomware changed, and what traces the attacker left behind.

---

## 1. Starting Point

After starting the VM, the triage/analysis file is sitting on the desktop, ready to be opened in Redline.

<img width="784" height="335" alt="screen1" src="https://github.com/user-attachments/assets/23a96f81-cea9-45ed-b05f-720f16f563c6" />


---

## 2. Opening the Collection in Redline

Redline opens with a few different starting options. We choose **"I am Reviewing a Triage Collection from HX"** to load the collected data.

<img width="1917" height="559" alt="screen2" src="https://github.com/user-attachments/assets/c09d1ebf-d854-4020-a4d0-b02dd3e99e5f" />


---

## 3. Identifying the Compromised User and OS

The first questions ask for the name of the compromised employee and which operating system was in use. Checking the **System Information** section in Redline provides both pieces of information directly.

<img width="788" height="863" alt="screen3" src="https://github.com/user-attachments/assets/f8c99444-834d-4352-a120-1abe4d768e1a" />


---

## 4. Finding the Malicious File and Download URL

Next, we need to determine which malicious file the user opened and the URL it was downloaded from. The **File Download History** section answers both questions.

<img width="1827" height="548" alt="screen4" src="https://github.com/user-attachments/assets/c2ed983d-146b-40c5-8294-e1b110f44de1" />


---

## 5. MD5 Hash and File Size of the Binary

We then need the MD5 hash and the file size of the binary. For this, we go to the **File System** view, locate the file inside the Downloads folder, and find all the required details there.

<img width="1909" height="865" alt="screen5" src="https://github.com/user-attachments/assets/eeeca92d-718f-411a-8ab2-479c939be7c9" />


---

## 6. Identifying the Ransomware's File Extension

Next, we need to find out which file extension the ransomware appended to encrypted files. Continuing to browse the file system, we notice that several files have been renamed from `*.txt` to a different extension.

<img width="604" height="168" alt="screen6" src="https://github.com/user-attachments/assets/058ed831-dc9e-47b8-8de3-526be51bb0cd" />


---

## 7. Counting the Affected Files

To find out how many files were modified by the malware, we switch to the **Timeline** view and filter for **Modified/Changed** files using the extension identified in the previous step. This lists every affected file.

<img width="1359" height="774" alt="screen7" src="https://github.com/user-attachments/assets/dfd0e7fe-4af6-4e35-8d1a-f302182c6b1c" />


---

## 8. Finding the Changed Desktop Wallpaper

The next question asks which wallpaper image the attacker set. This one was not straightforward. I ended up filtering for `.bmp` files and manually going through the resulting list; after three or four attempts, I found the correct file. There is likely a cleaner way to filter for this directly, which I'd like to look into afterwards.

<img width="1565" height="710" alt="screen8" src="https://github.com/user-attachments/assets/39ad00aa-72e8-470c-a53e-a4e356b3e606" />


---

## 9. File Left Behind on the Desktop

To find out which file the attacker left on the desktop, we go back to the file system, filter for the Desktop folder, and look through its contents. This one turned up quickly.

<img width="1202" height="674" alt="screen9" src="https://github.com/user-attachments/assets/fd8b2e7b-a0c6-4eb0-b6b3-5e64cc26b694" />


---

## 10. A File Containing a Spanish Term

The attacker also left a file under `C:\Users\John Coleman\Favorites\Links for United States\`, which is supposed to contain a Spanish term. Since my Spanish isn't great, I simply looked for any file names that didn't look like standard, recognizable terms. This narrowed it down quickly.

<img width="1222" height="718" alt="screen10" src="https://github.com/user-attachments/assets/7d66e23a-30f9-41e0-97ed-5af6625d7543" />


---

## 11. Finding the Empty .lock File

We also need to find a 0 KB file on the desktop with a `.lock` extension. This was located the same way, by filtering the file system.

<img width="1512" height="676" alt="screen11" src="https://github.com/user-attachments/assets/1bf123f2-ba83-4d84-a5e7-6d23b85c2775" />


---

## 12. The Decryptor Downloaded by the User

Since the user didn't know how else to help themselves, they downloaded a decryptor. This file is also on the desktop, and we need its MD5 hash.

<img width="766" height="682" alt="screen12" src="https://github.com/user-attachments/assets/57645d7c-d8c1-481d-a0d5-1179030d79e5" />


---

## 13. Finding the Decryption URL in Browser History

The ransom note also referenced a URL reachable over the regular internet, used for decryption. We find it in the **browser history**, sorted by most recently visited, and scanning through it reveals the URL.

<img width="1208" height="676" alt="screen13" src="https://github.com/user-attachments/assets/f56ebb47-3c9f-4796-b2fb-f37e6862386e" />


---

## 14. Attribution via the File Hash

The final question asks for three names associated with this ransomware. For this, a general web search helps: searching for the file hash directly brought up the answer via an AI-generated summary. Whether that was the intended approach is hard to say, but it got the job done.

<img width="804" height="466" alt="screen14" src="https://github.com/user-attachments/assets/bb13fc70-8269-4e52-af26-28d1e240959a" />


Room complete.

---

## Lessons Learned

- **Triage collections centralize a lot of evidence:** Redline's System Information, File Download History, File System, and Timeline views cover most of what's needed to reconstruct a ransomware incident without touching raw disk images.
- **Timelines are powerful for scoping impact:** Filtering modified/changed files by extension in the Timeline view is a fast way to see exactly what a piece of ransomware touched.
- **Not every artifact is obvious:** Finding the changed wallpaper took manual filtering and trial and error; a more targeted search approach (e.g. by file size or last-write time around the infection timestamp) is worth exploring for next time.
- **OSINT complements forensics:** A hash lookup via a search engine can quickly reveal public attribution or reporting on a known malware sample, useful for confirming identification without needing to reverse-engineer the sample yourself.
- **Victims often make things worse:** Downloading an unverified "decryptor" from a ransom note is risky, but it's a realistic artifact to look for during an investigation.

---

## Tools Used

- Mandiant Redline (triage collection analysis: System Information, File Download History, File System, Timeline)
- Web browser / search engine (hash lookup and attribution)
