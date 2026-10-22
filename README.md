# 🔍 Nikto-Web-Application-Scanning

**Author:** Dashane James  
**Lab Environment:** VMware Workstation | Kali Linux | Metasploitable 2 
**Purpose:** The goal in this repository is to use Nikto Web Application scanner to scan Metasploitable's web server in order to identify, document, and study vulnerabilities.  
**Status:** 🔵 Completed

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

---

## 🔬 Tasks / Assessments Performed

###1. Run a basic Nikto scan against port 80

I ran a basic Nikto scan on port 80 of the vulnerable Metasploitable server (which is the default port when using Nikto). I then recorded and reviewed the results. 

# Command used
nikto -h 192.168.79.130

<img width="322" height="256" alt="Screenshot 2026-08-16 132425" src="https://github.com/user-attachments/assets/71f95ab4-bb16-47b7-9b3c-306d6fc28448" />

<img width="323" height="256" alt="Screenshot 2026-08-16 132503" src="https://github.com/user-attachments/assets/57c38e59-6600-4841-acc5-78e55202f6f9" />

<img width="323" height="257" alt="Screenshot 2026-08-16 132532" src="https://github.com/user-attachments/assets/befafe39-6b48-4501-8ae2-5621ae5a14b4" />

Output

Command first displays data about the target such as the target's IP, hostname, port number, platform, and time the scan was executed. Next, the exact web server being used is displayed, followed by several vulnerabilities found in the target's web sever. The exact method in which an attacker might exploit these vulnerabilities is explained as well. In total, their was 8244 requests, 30 findings, 18 errors, and 1 host tested.

Findings: 

Overall,  what i found was that the target's web server was essentially an unpatched and over-exposed target with no meaningful hardening at the web server level. I discovered that the web sever appears to be running outdated software such as Apache 2.2.8 and PHP 5.2.4, with nearly the entire header set missing (the instructions the server sends to the browser telling it to enforce extra safety rules.). I also found multiple information-disclosure issues, several browsable directories, and the server has mod_negotiation/MultiViews(An Apache feature that auto picks the best matching file when a request doesn't specify an extension.) enabled, and HTTP TRACE was also active.

2. Save Results to a file
   
Use command prompt to save the results from the Nikto scan that was performed in the previous task onto a file that can be accessed whenever needed.

# Commands used
nikto -h 192.168.79.130 -p 80 -o ~/nikto_reslts.txt
cat ~/nikto_reslts.txt.txt

<img width="326" height="257" alt="Screenshot 2026-08-16 222906" src="https://github.com/user-attachments/assets/fbb4f2e4-1517-46b8-ada5-bcf1d64f33a1" />

<img width="326" height="259" alt="Screenshot 2026-08-16 222825" src="https://github.com/user-attachments/assets/814b4678-4e7f-48ae-8900-dbbab5cd6f1e" />

Output

The output in the first screenshot shows the web vulnerability scan being run on the Metasploitable server and the command requests for the results to be saved to file "nikto_reslts.txt". However, Nikto actually saved the file as "nikto_reslts.txt.txt" because it adds its own format extension regardless of what was typed. The second screenshot shows the cat command accessing and displaying the contents of nikto_reslts.txt.txt.

Findings: 

Nikto created a double extension (nikto_reslts.txt.txt) because Nikto adds its own file extension to the end regardless of what was already typed.
I found Nikto gives no confirmation message that the save succeeded, so verifying the file afterward is a necessary step rather than an option.

3. Look up two vulnerabilities that Nikto finds
   
I Researched three of the vulnerabilities found in the Nikto scan. This gave me a better understanding of the types of vulnerabilities that are open in Metaspoloitable. It also gave me an understanding on how and why they are exploited.


Research Summary 

Outdated/EOL software (Ex: Apache 2.2.8, PHP 5.2.4) - Not a specific CVE, but it's the single most common category of vulnerability finding in the industry. The cause and dangers are pretty straight forward in that once a version of software becomes outdated/EOL there are no longer any patches being made to protect it from new attacks which leaves the software completely vulnerable. What makes this vulnerability one of the most dangerous is that an attacker doesn't have to find anything new they can just look up known, already documented exploits for the specific version and use them directly.

HTTP TRACE/Cross-site Tracing (XST) - OWASP-documented attack class that shows up constantly in real pen test reports, compliance scans (PCI-DSS specifically flags TRACE), and security training material. TRACE is a legitimate, built-in web server debugging feature. When activated, the server just echoes the exact request it received right back to the sender including all headers. The danger comes when its paired with a separate more common flaw called XSS, where an attacker manages to get malicious code to run inside a victims browser on that site. Since TRACE echoes the original request back, cookies included, and injected script is allowed to read that echoed response. So TRACE essentially hands the attacker a side door around a browser protection that's supposed to be unbreakable.

mod_negotiations/MultiViews - A built in Apache feature that checks for which file you meant when a file doesn't specify an exact name/path. It exists purely for convenience, so a link doesn't crash a site if a site quietly switches from .html to .php and so on. However, this convenient feature becomes a discovery tool in the wrong hands. An attacker can utilize this tool to receive a yes or no response to whether or not a file exists on the sever, rather than the usual "not found" response. This is concerning because an attacker can continuously guess file names until the server confirms if it exists or not.

4. Compare Nikto results vs Nmap results
In this task, I compared the result i got from running a Nikto scan to the results I received when running Nmap to find vulnerabilities in the same target Metasploitable machine. I did this to understand what each tool is actually built to find and how they can complement or differentiate from each other in a real assessment.

# Commands used

nikto -h 192.168.79.130
sudo nmap -sV -Pn --disable-arp-ping -n -p 21,22,23,80,3306 192.168.79.130
sudo nmap --script ftp-anon -Pn --disable -arp-ping -n -p 21 192.168.79.130
sudo nmap --script smb-vuln* -Pn --disable -arp-ping -n -p 445 21 192.168.79.130

Output

In terms of finding vulnerabilities, Nikto stayed scoped entirely to port 80, finding application-layer issues: outdated Apache/PHP, missing headers, browsable directories, exposed phpMyAdmin, and TRACE/XST. While Nmap covered the whole host, finding anonymous FTP on vsftpd 2.3.4, cleartext Telnet, weak SMB config, and exposed MySQL. Both tools independently fingerprinted the same Apache version. 


Finding: 

I found that Nmap and Nikto cover different scopes. I also found that Nmap isn't tied to any specific protocol, it surveys the whole host against any service. While Nikto is a specialist that only speaks HTTP/HTTPS and digs deep into it. Also when using Nmap, I had to specifically add ftp-anon and smb-vuln* to run these vulnerability checks while Nikto automatically runs it's entire signature database the moment you launch it. That difference is what led to me to be able to use nmap to find vsftpd 2.3.4, a  vulnerability with a known backdoor, which is far more severe than anything found on port 80 using Nikto even if only the version was confirmed and not an actual exploit.

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


🔑 Technical Notes


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

Cross-site Scripting(XSS) - 

# Any important commands or workarounds

ls -la ~/ | grep -i nikto
-o 
cat

📌 About This Project
[1-2 sentences about how this fits into your overall portfolio and career goals.]

👤 Author
Dashane James
Senior Field Service Technician → Cybersecurity Analyst
📍 Yonkers, NY
🎓 B.S. Information Technology — SUNY Canton
🏆 CompTIA Security+ | CySA+ (In Progress)
🔗 GitHub | Zero Trust Cyber Security Brand

This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.

