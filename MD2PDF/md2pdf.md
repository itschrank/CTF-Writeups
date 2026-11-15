# TryHackMe – MD2PDF Write-up

**Platform:** TryHackMe
**Room:** [MD2PDF](https://tryhackme.com/room/md2pdf)
**Topics:** Local File Inclusion via PDF rendering, Server-Side Request Forgery (SSRF), enumeration with Gobuster, metadata analysis with ExifTool

> **Note:** Flags are intentionally omitted from this write-up. The machine crashed once during testing and had to be rebooted, so the target IP address changes between some of the screenshots.

---

## Overview

The target exposes a web interface on port 80 that converts text into a PDF. By identifying the tool behind this conversion and the internal services it can reach, it is possible to render an internally-restricted admin page into a PDF and retrieve the flag.

---

## 1. First Look at the Web Interface

Browsing to the machine on port 80 shows a web interface that generates a PDF from text input. As a first test, I entered the string `1214` and generated a PDF from it. This worked without any issues, confirming that the feature is functional and worth investigating further.

---

## 2. Enumeration with Gobuster

Since the web page itself did not reveal any further information, I ran a **Gobuster** scan against the web server. This turned up a hidden path: `/admin`.

<img width="656" height="346" alt="conv1" src="https://github.com/user-attachments/assets/858bd8ce-b33c-464c-9c6f-34e4d5d2cde6" />


---

## 3. Admin Path is Internal-Only

Trying to access `/admin` directly resulted in an error stating that this page can only be reached internally, on **port 5000**. This immediately stood out as the likely path to the flag, since it meant the admin page was reachable from the server itself, just not from the outside.

<img width="608" height="185" alt="conv5" src="https://github.com/user-attachments/assets/3b9734bd-f09f-417c-9b5b-8e4743f42664" />


---

## 4. Identifying the PDF Engine

Going back to the PDF I generated earlier from the text `1214`, I downloaded it and inspected its metadata with **ExifTool**. This revealed that the PDF had been created using **`wkhtmltopdf 0.12.5`**.

<img width="489" height="331" alt="conv3" src="https://github.com/user-attachments/assets/7573bda3-16ad-472b-960a-44f4e307dc88" />


A short search confirmed that `wkhtmltopdf` renders HTML input into a PDF, which means any HTML I enter into the text field gets interpreted and rendered server-side.



---

## 5. Exploiting the PDF Rendering (SSRF)

Since the server itself renders HTML before converting it to a PDF, and the admin page is reachable internally on port 5000, I tried embedding an iframe pointing at the internal admin page instead of entering plain text:

```html
<iframe src="http://localhost:5000/admin"></iframe>
```

<img width="855" height="466" alt="conv6" src="https://github.com/user-attachments/assets/fb812a27-48a4-461e-93a6-38ad303a193a" />


Because the rendering happens on the server, the request to `localhost:5000/admin` originates from the machine itself, bypassing the "internal only" restriction.

---

## 6. Flag Retrieved

The resulting PDF did not just contain the raw HTML of the admin page, it rendered the page itself, flag included.

<img width="754" height="305" alt="conv2" src="https://github.com/user-attachments/assets/d20eca4d-b161-46a1-8347-cd7d175709b4" />


Room complete.

---

## Lessons Learned

- **Trust the rendering engine, not just the input:** A feature that converts user input into another format (here: HTML into PDF) can expose the capabilities of the underlying tool, including the ability to fetch other resources.
- **"Internal only" restrictions are not enough on their own:** Binding a service to localhost or an internal port does not help if another process on the same host can be tricked into making requests to it. Such sensitive pages still need proper authentication.
- **Metadata can reveal the tech stack:** A simple ExifTool check on a generated file exposed the exact tool and version used server-side, which directly pointed to the next step (looking up known behaviors of `wkhtmltopdf`).
- **Enumeration pays off:** The Gobuster scan surfacing `/admin` was the detail that made the rest of the chain possible.

---

## Tools Used

- Web browser
- Gobuster (directory enumeration)
- ExifTool (metadata analysis)
- `wkhtmltopdf` behavior/knowledge (identified via metadata, exploited via iframe/SSRF)
