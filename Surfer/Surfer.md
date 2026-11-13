# TryHackMe – Surfer Write-up

**Platform:** TryHackMe
**Room:** [Surfer](https://tryhackme.com/room/surfer)
**Topics:** Weak credentials, Server-Side Request Forgery (SSRF), PDF export abuse

> **Note:** Flags and credentials are intentionally omitted from this write-up.

---

## Overview

The room asks us to navigate to the target's web interface. This write-up walks through how I gained access, discovered the PDF export feature, and used it to read a page that is only accessible locally.

---

## 1. Initial Look at the Web Interface

After loading the web interface, I inspected the page source to look for anything that could give me a foothold, such as hidden comments, hardcoded credentials or interesting endpoints. At first glance, I found nothing useful.

<img width="813" height="614" alt="screen1" src="https://github.com/user-attachments/assets/c6c63990-d5e9-4ea2-8021-5d0ecff2e65d" />


---

## 2. Gaining Access

Since the source code was a dead end, I tried a few common weak passwords manually. It did not take long before I was able to log in. No tools were required for this step.

<img width="529" height="602" alt="screen2" src="https://github.com/user-attachments/assets/9c9b5b7b-e600-4052-b0b9-1238e8f0899b" />


> **Takeaway:** Weak or default credentials are still one of the easiest ways in. Always try the obvious before reaching for heavy tooling.

---

## 3. Discovering the PDF Export Function

Once logged in, I noticed a function to **export a report as a PDF**. I tried it out and generated a report. While looking at the result, I noticed that the report was built from a **local file, `server-info.php`**, which the server reads and renders into the PDF.

<img width="1313" height="917" alt="screen3" src="https://github.com/user-attachments/assets/d007e0af-bf30-48f3-b344-b19d84e2a164" />


This was interesting: the server itself fetches a resource on my behalf.

---

## 4. Finding the Internal Admin Page

On the admin page of the web interface, I found a hint that the system flag is located at:

```
/internal/admin.php
```

When I tried to open this path directly in the browser, I found that it is **only accessible locally** (i.e. from the server itself), so I was blocked.

<img width="774" height="181" alt="screen7" src="https://github.com/user-attachments/assets/015175ae-3d52-4164-86b2-1542ba1f6106" />


---

## 5. Exploiting the PDF Export (SSRF)

This gave me an idea: if the PDF export makes the **server** fetch a local resource, I might be able to make it fetch `admin.php` instead. Since the request then originates from the server itself, the "local only" restriction should not apply.

I intercepted the PDF export request with **Burp Suite** and inspected it more closely. Then I modified the parameter that defines which page is exported, pointing it at the internal admin page instead of `server-info.php`.

<img width="1570" height="410" alt="screen5" src="https://github.com/user-attachments/assets/4cce91c6-421c-4336-b850-4bb0e3dc6d7a" />


After sending the manipulated request, the server returned a PDF containing the content of `admin.php`, including the flag.

<img width="1355" height="336" alt="screen6" src="https://github.com/user-attachments/assets/1b4eaafa-1172-4cb3-94f4-b0eb4075d871" />


---

## Lessons Learned

- **Weak credentials:** Simple passwords can give access without any tooling. Strong, unique passwords and lockout or rate limiting would have prevented this.
- **Server-Side Request Forgery (SSRF):** Features that let the server fetch resources based on user input (like a PDF renderer) can be abused to reach internal-only pages. Such input should be validated strictly, ideally against an allowlist.
- **Do not rely on "local only" restrictions alone:** If the server can be tricked into making requests to itself, IP-based protection is bypassed. Sensitive pages should require proper authentication and authorization as well.
- **Read the request carefully:** Intercepting and understanding what a feature sends to the server (here with Burp Suite) often reveals the real attack surface.

---

## Tools Used

- Web browser (page source inspection)
- Burp Suite (intercepting and modifying the PDF export request)
