# 🔍 Nikto-Web-Application-Scanning

**Author:** Dashane James  
**Lab Environment:** [e.g. VMware Workstation | Kali Linux | Metasploitable 2]  
**Purpose:** The goal in this repository is to use Nikto Web Application scanner to scan Metasploitable's web server in order to identify, document, and study vulnerabilities.  
**Status:** 🟢 Active / 🟡 In Progress / 🔵 Completed

---

## 📋 Overview

This repository documents a series of 4 assessments performed in my isolated VMware home lab, which in this case is targeting a Metasploitable server on my PC. The goal was to build hands-on proficiency using Nikto Web Application scanner to identify vulnerabilities and compare it to the results of other scanners that I have practiced with so far, particularly Nmap. 

---

## 🧪 Lab Environment
| Component | Details |
|---|---|
| Hypervisor | VMware Workstation (Host-Only Network) |
| Attacker Machine | Kali Linux 2026.1 — `192.168.79.129` |
| Target Machine | [Metasploitable] — `192.168.79.130` |
| Network Type | Host-Only (isolated, no internet exposure) |
| Host OS | Windows 11 — ASUS Vivobook 14 |

> ⚠️ **Note:** All activity was performed in a controlled, isolated lab environment against deliberately vulnerable machines. No unauthorized access to live networks was performed.

---

## 🛠️ Tools Used

- **[Nikto]** — a free, open-source web server scanner that tests web servers for dangerous files, outdated software, and misconfigurations.
- **[Metasploitable]** — Acts as a target machine with many intentional vulnerabilities to practice penetration testing, ethical hacking, and securely auditing safely.


---

## 📁 Repository Structure

```
[Repo-Name]/
├── README.md
├── [folder-1]/
│   ├── [file1.txt]        # Description of file
│   └── [file2.md]         # Description of file
├── [folder-2]/
│   ├── [file3.md]         # Description of file
│   └── [file4.txt]        # Description of file
└── reports/
    └── [final-report.md]  # Full assessment report
```

---

## 🔬 Tasks / Assessments Performed

###1. Run a basic Nikto scan against port 80

I ran a basic Nikto scan on port 80 of the vulnerable Metasploitable server (which is the default port when using Nikto). I then recorded and reviewed the results. )

# Command used
[nikto -h 192.168.79.130]

<img width="322" height="256" alt="Screenshot 2026-08-16 132425" src="https://github.com/user-attachments/assets/71f95ab4-bb16-47b7-9b3c-306d6fc28448" />

<img width="323" height="256" alt="Screenshot 2026-08-16 132503" src="https://github.com/user-attachments/assets/57c38e59-6600-4841-acc5-78e55202f6f9" />

<img width="323" height="257" alt="Screenshot 2026-08-16 132532" src="https://github.com/user-attachments/assets/befafe39-6b48-4501-8ae2-5621ae5a14b4" />

Output
Command first displays data about the target such as the target's IP, hostname, port number, platform, and time the scan was executed. Next, the exact web server being used is displayed, followed by several vulnerabilities found in the target's web sever. The exact method in which an attacker might exploit these vulnerabilities is explained as well. In total, their was 8244 requests(what kind of requests?, 30 findings, 18 errors, and 1 host tested.

Findings: Overall,  what i found was that the target's web server was essentially an unpatched and over-exposed target with no meaningful hardening at the web server level. I discovered that the web sever appears to be running outdated software such as Apache 2.2.8 and PHP 5.2.4, with nearly the entire header set missing (the instructions the server sends to the browser telling it to enforce extra safety rules.). I also found multiple information-disclosure issues, several browsable directories, and the server has mod_negotiation/MultiViews(An Apache feature that auto picks the best matching file when a request doesn't specify an extension.) enabled, and HTTP TRACE was also active (an XST Risk ???) 

2. [Save Results to a file]
[Use command prompt to save the results from the Nikto scan that was performed in the previous task onto a file that can be accessed whenever needed.]

# Commands used
[nikto -h 192.168.79.130 -p 80 -o ~/nikto_reslts.txt]
[cat ~/nikto_reslts.txt.txt]

<img width="326" height="257" alt="Screenshot 2026-08-16 222906" src="https://github.com/user-attachments/assets/fbb4f2e4-1517-46b8-ada5-bcf1d64f33a1" />

<img width="326" height="259" alt="Screenshot 2026-08-16 222825" src="https://github.com/user-attachments/assets/814b4678-4e7f-48ae-8900-dbbab5cd6f1e" />

Output
The output in the first screenshot shows the web vulnerability scan being run on the Metasploitable server and the command requests for the results to be saved to file "nikto_reslts.txt". However, Nikto actually saved the file as "nikto_reslts.txt.txt" because it adds its own format extension regardless of what was typed. The second screenshot shows the cat command accessing and displaying the contents of nikto_reslts.txt.txt.

Finding: 
Nikto created a double extension (nikto_reslts.txt.txt) because Nikto adds its own file extension to the end regardless of what was already typed.
I found Nikto gives no confirmation message that the save succeeded, so verifying the file afterward is a necessary step rather than an option.

3. [Look up two vulnerabilities that Nikto finds]
[Brief description of what you did and why]

# Command used
[your command here]

Output

Finding: [What did you discover?]

4. [Task Name]
[Brief description of what you did and why]

# Command used
[your command here]

Output

Finding: [What did you discover?]

📊 Key Findings Summary
Port/Service	Tool Used	Risk Level	Notes
[e.g. 21/tcp FTP]	[e.g. Nmap]	🔴 Critical	[Notes]
[e.g. 23/tcp Telnet]	[e.g. Wireshark]	🔴 Critical	[Notes]
[e.g. 80/tcp HTTP]	[e.g. Nikto]	🟠 High	[Notes]
[e.g. 3306/tcp MySQL]	[e.g. OpenVAS]	🟠 High	[Notes]
[e.g. 22/tcp SSH]	[e.g. Nmap]	🟡 Medium	[Notes]


Risk Levels: 🔴 Critical | 🟠 High | 🟡 Medium | 🟢 Low

🗺️ MITRE ATT&CK Mapping
Action Performed	ATT&CK Tactic	Technique ID	Technique Name
[e.g. Port scanning]	[e.g. Reconnaissance]	[e.g. T1595]	[e.g. Active Scanning]
[Action]	[Tactic]	[ID]	[Technique]
[Action]	[Tactic]	[ID]	[Technique]
[Action]	[Tactic]	[ID]	[Technique]
🛡️ Defensive Recommendations
Based on findings, the following remediations would be recommended in a real environment:

[Finding 1] — [Recommendation]
[Finding 2] — [Recommendation]
[Finding 3] — [Recommendation]
[Finding 4] — [Recommendation]
[Finding 5] — [Recommendation]
📚 CySA+ Exam Relevance
This lab directly maps to the following CompTIA CySA+ (CS0-003) exam domains:

Domain	Coverage
Security Operations (33%)	[What this lab covers in this domain]
Vulnerability Management (30%)	[What this lab covers in this domain]
Incident Response (20%)	[What this lab covers in this domain]
Reporting & Communication (17%)	[What this lab covers in this domain]
🔑 Technical Notes
[Any important notes about your lab setup, workarounds, or lessons learned. Example:

APACHE - The web server software itself. It is the program listening  on port 80 that receves a browser's request and sends back the response. AKA the delivery engine.

PHP - A scripting language that runs on the sever (not in the browser) to build dynamic content (pulling data, processing forms, generating pages on the fly.) APACHE hands .php files to PHP, PHP generates HTML, Apache ships it back out. So APACHE is the courier, while PHP is what assembles the package. 

HTML -  The markup language that structures what a browser displays(headings, text, links, images, forms. Its not a programming language because it doesnt run or process anything it just describes what the pages should look like. Static pages serve HTML straight from disk, while dynamic pageslike PHP files, generate  HTML on thefly before sending it out.

HTML VS PHP ?
HTML defines what a page looks like and structures content like texts, links, images, and forms but doesn't execute or process anything itself. PHP is the server side scripting language that defines what happens before the page is sent, executing logic and typically producing HTML as its output.

MySQL - A database server. Its job is storing and managing structured data (user accounts, records, whatever an app needsto save.) A typical setup has Apache receive the request, PHP handle the logic, and PHP query MySQL when it needs data. (APACHE,MySQL,PHP together on Linux is such a common combo it has a name. It is called the "LAMP stack"(LINUX, APACHE, MYSQL, PHP.) Port 3306 

vsftpd (very secure ftp daemon) - An FTP sever. Its role is direct file upload/download over the FTP protocol, nothing to do with serving web pages. Completely separate from Apache. Uses Port 21

Samba - A file and print server. Implements WINDOWS-style file/printer sharing (SMB protocol) on Linux, so folders can be browsed like a network drive. Port 139/445.

FTP VS SMB - ftp moves files between two points. SMB is the dominant protocol for local network file sharing and makes a remote file system feel like its on your own machine.

The Server Message Block (SMB) protocol - is a client-server network communication protocol used to share files, printers, and serial ports across a local network. It lets a user open, read, write, and manage files on a remote computer or server as if those files lived on their own local hard drive.

HTTP TRACE - A debugging method that echoes your request straight back to you (headers included.) Which when paired with an XSS vulnerability can possibly become an even bigger vulnerability.

# Any important commands or workarounds
[command here]
ls -la ~/ | grep -i nikto
-o 
cat
📌 About This Project
[1-2 sentences about how this fits into your overall portfolio and career goals.]

Related repositories:

[Repo Name] — [Brief description]
[Repo Name] — [Brief description]
[Repo Name] — Coming soon
👤 Author
Dashane James
Senior Field Service Technician → Cybersecurity Analyst
📍 Yonkers, NY
🎓 B.S. Information Technology — SUNY Canton
🏆 CompTIA Security+ | CySA+ (In Progress)
🔗 GitHub | Zero Trust Cyber Security Brand

This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.

