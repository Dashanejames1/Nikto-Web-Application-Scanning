# Nikto-Web-Application-Scanning

**Author:** Dashane James  
**Lab Environment:** VMware Workstation | Kali Linux | Metasploitable 2  
**Purpose:** Use Nikto to scan the Metasploitable web server and document what an HTTP-focused scanner finds that general-purpose tools miss.  
**Status:** 🔵 Completed

---

## 📋 Overview

This repository documents four assessments using Nikto against the Metasploitable 2 web server. The goal was to build hands-on proficiency with the tool and understand what it surfaces compared to a general-purpose scanner like Nmap.

Nikto is a web application scanner with a narrow scope by design. The moment you point it at a host, it automatically runs its entire signature database against the target and returns a structured list of HTTP-layer findings: outdated software versions, missing security headers, exposed admin interfaces, dangerous server-side features, and browsable directories. That depth is what makes it valuable in an assessment, and understanding what each finding means matters as much as running the scan itself.

The scan confirmed what version detection had already suggested: the Metasploitable web server is running end-of-life Apache and PHP with no meaningful hardening at the HTTP layer.

---

## 🧪 Lab Environment

| Component | Details |
|---|---|
| Hypervisor | VMware Workstation (Host-Only Network) |
| Attacker Machine | Kali Linux 2026.1, `192.168.79.129` |
| Target Machine | Metasploitable 2, `192.168.79.130` |
| Network Type | Host-Only (isolated, no internet exposure) |
| Host OS | Windows 11, ASUS Vivobook 14 |

> ⚠️ **Note:** All activity was performed in a controlled, isolated lab environment against a deliberately vulnerable machine. No unauthorized access to live networks was performed.

---

## 🛠️ Tools Used

- **Nikto:** A free, open-source web server scanner that tests for dangerous files, outdated software, and misconfigurations. Unlike Nmap, it runs its complete signature database automatically on launch with no additional flags required.
- **Metasploitable 2:** Intentionally vulnerable Linux target, providing a realistic web server attack surface for safe, legal practice.

---

## 🔬 Tasks / Assessments Performed

### 1. Run a basic Nikto scan against port 80

```bash
nikto -h 192.168.79.130
```

<img width="322" height="256" alt="Screenshot 2026-08-16 132425" src="https://github.com/user-attachments/assets/71f95ab4-bb16-47b7-9b3c-306d6fc28448" />

<img width="323" height="256" alt="Screenshot 2026-08-16 132503" src="https://github.com/user-attachments/assets/57c38e59-6600-4841-acc5-78e55202f6f9" />

<img width="323" height="257" alt="Screenshot 2026-08-16 132532" src="https://github.com/user-attachments/assets/befafe39-6b48-4501-8ae2-5621ae5a14b4" />

The scan opened by displaying target metadata: IP address, hostname, port, server platform, and timestamp. Total output was 30 findings across 8,244 requests, with 18 errors logged during the run.

The findings break down into four categories. The server is running end-of-life software: Apache 2.2.8 and PHP 5.2.4 are both EOL, meaning no security patches are being released for known vulnerabilities in either. The HTTP security header set is almost entirely absent; the server does not send X-Frame-Options, X-XSS-Protection, X-Content-Type-Options, Strict-Transport-Security, or Content-Security-Policy, which means the browser applies no additional enforcement rules against content manipulation or clickjacking. Several directories are browsable rather than returning a 403 or 404, which exposes the directory structure to any visitor. Finally, two server-side features are active that should be disabled on any production system: mod_negotiation with MultiViews enabled, and HTTP TRACE.

**Key observation:** Nikto stayed entirely scoped to port 80. It found meaningful issues there, but the most severe vulnerability in this environment (the vsftpd 2.3.4 backdoor on port 21) would not appear in a Nikto-only assessment. That comparison comes back in Task 4.

---

### 2. Save results to a file

```bash
nikto -h 192.168.79.130 -p 80 -o ~/nikto_results.txt
cat ~/nikto_results.txt.txt
```

<img width="326" height="257" alt="Screenshot 2026-08-16 222906" src="https://github.com/user-attachments/assets/fbb4f2e4-1517-46b8-ada5-bcf1d64f33a1" />

<img width="326" height="259" alt="Screenshot 2026-08-16 222825" src="https://github.com/user-attachments/assets/814b4678-4e7f-48ae-8900-dbbab5cd6f1e" />

The `-o` flag saves scan output to a file, but Nikto appended its own format extension to whatever filename was typed, resulting in `nikto_results.txt.txt` rather than `nikto_results.txt`. Nikto also gives no confirmation message when the save succeeds, so verifying the file afterward is a necessary step.

**Finding:** The double-extension behavior is a minor but consistent Nikto quirk worth knowing. If output files will be referenced in a script or a report, account for the appended extension or rename the file after the scan completes.

---

### 3. Research three vulnerabilities Nikto identified

**End-of-life software (Apache 2.2.8, PHP 5.2.4)**

EOL software is not a specific CVE but is consistently the most common root cause category in real-world vulnerability assessments. When a version goes EOL, patches stop. Any vulnerability discovered after that date has no fix coming, and any known exploit documented before that date remains valid against the installed version. An attacker does not need to discover anything new; they look up documented exploits for the specific version and run them directly.

**HTTP TRACE and Cross-Site Tracing (XST)**

TRACE is a built-in HTTP debugging method that makes the web server echo the exact request it received, headers included, back to the sender. In isolation, it is just a debugging feature. The danger comes from pairing it with an XSS vulnerability. If an attacker can run malicious script inside a victim's browser on the same site, that script can trigger a TRACE request and read the echoed response, including the session cookie. The same-origin policy that is supposed to prevent cross-origin access to cookies does not block this because the TRACE response originates from the same domain. The result is that an attacker can read session cookies even when the HttpOnly flag is set. PCI-DSS explicitly requires TRACE to be disabled for this reason.

**mod_negotiation/MultiViews**

Apache's content negotiation feature guesses which file to serve when a request does not include a file extension, so a link to `/page` works even if the underlying file is `page.html`, `page.php`, or `page.txt`. The convenience feature becomes a file enumeration tool in the wrong hands. Rather than returning a standard "Not Found" for nonexistent files, MultiViews returns a different response code for files that do exist versus those that do not. An attacker can brute-force directory and filename guesses and get a binary yes/no existence confirmation on each request until they map out files that were never intended to be discoverable.

---

### 4. Compare Nikto results vs. Nmap results

```bash
nikto -h 192.168.79.130
sudo nmap -sV -Pn --disable-arp-ping -n -p 21,22,23,80,3306 192.168.79.130
sudo nmap --script ftp-anon -Pn --disable-arp-ping -n -p 21 192.168.79.130
sudo nmap --script smb-vuln* -Pn --disable-arp-ping -n -p 445 192.168.79.130
```

Both tools independently fingerprinted the same Apache version on port 80. After that, their coverage diverges completely. Nikto stayed on port 80 and surfaced application-layer findings: EOL Apache/PHP, missing headers, browsable directories, exposed phpMyAdmin, and the TRACE/MultiViews issues. Nmap covered the entire host and found anonymous FTP on vsftpd 2.3.4, cleartext Telnet, weak SMB configuration, and exposed MySQL.

The scope difference matters for severity. Nikto's findings on port 80 are real and worth fixing, but the vsftpd 2.3.4 backdoor Nmap flagged on port 21 is exploitable as an unauthenticated root shell. That is a meaningfully higher severity finding, and Nikto would never find it because Nikto does not speak FTP.

The other structural difference is how vulnerability checks are triggered. Nikto automatically runs its entire signature database the moment the scan launches. Nmap does not run vulnerability checks by default; the ftp-anon and smb-vuln checks above had to be added explicitly as NSE scripts. A basic Nmap scan can miss application-layer checks that Nikto runs automatically.

**Finding:** Neither tool replaces the other. Nikto is a specialist that digs deep into HTTP. Nmap surveys the whole host across any service. A complete assessment needs both.

---

## 📊 Key Findings Summary

| Port/Service | Tool | Risk Level | Notes |
|---|---|---|---|
| 80/tcp HTTP | Nikto | 🟠 High | Apache 2.2.8 and PHP 5.2.4, both EOL; no patches available for either version. |
| 80/tcp HTTP | Nikto | 🟠 High | phpMyAdmin accessible without IP restriction; exposes the MySQL admin interface directly to the network. |
| 80/tcp HTTP | Nikto | 🟡 Medium | HTTP TRACE enabled; enables Cross-Site Tracing (XST) when combined with an XSS vulnerability. |
| 80/tcp HTTP | Nikto | 🟡 Medium | Security headers absent (X-Frame-Options, X-XSS-Protection, Content-Security-Policy, X-Content-Type-Options). |
| 80/tcp HTTP | Nikto | 🟡 Medium | Multiple directories browsable; directory listings expose the file structure to unauthenticated visitors. |
| 80/tcp HTTP | Nikto | 🔵 Low | mod_negotiation/MultiViews active; enables file existence enumeration via differing response codes. |

---

## 🗺️ MITRE ATT&CK Mapping

| Action Performed | Tactic | Technique ID | Technique Name |
|---|---|---|---|
| Nikto web application scan against port 80 | Reconnaissance | T1595.002 | Active Scanning: Vulnerability Scanning |
| Researching Nikto findings against CVE and OWASP documentation | Reconnaissance | T1588.006 | Obtain Capabilities: Vulnerabilities |

---

## 🛡️ Defensive Recommendations

1. **Upgrade Apache and PHP immediately (High, port 80):** Apache 2.2.8 and PHP 5.2.4 are end-of-life with years of unpatched CVEs. Replace with currently supported releases. Every other hardening step is undermined by running software that cannot be patched.

2. **Restrict phpMyAdmin access (High, port 80):** phpMyAdmin should not be reachable from the general network. Restrict access to known administrative IP addresses and require authentication before the interface loads. If phpMyAdmin is not operationally necessary, removing it is the better option.

3. **Disable HTTP TRACE (Medium, port 80):** Add `TraceEnable Off` to the Apache configuration. TRACE has no legitimate use on a production web server and its removal has no user-facing impact.

4. **Add security response headers (Medium, port 80):** Configure Apache to send X-Frame-Options, X-Content-Type-Options, X-XSS-Protection, and Content-Security-Policy with every response. These instruct the browser to enforce protections the server itself cannot enforce.

5. **Disable directory listing (Medium, port 80):** Set `Options -Indexes` in the Apache configuration to return a 403 for directories without an index file instead of listing their contents.

6. **Disable mod_negotiation/MultiViews (Low, port 80):** Remove `MultiViews` from the Apache Options directive. For most sites, content negotiation provides no real benefit, and the file enumeration exposure is not worth keeping it active.

---

## 🔑 Technical Notes

**LAMP stack context**

Most of what Nikto found on port 80 connects to the LAMP stack: Linux, Apache, MySQL, and PHP. Understanding how these four components interact clarifies why each finding matters.

Apache is the web server: the program listening on port 80 that receives HTTP requests and sends back responses. PHP is the server-side scripting language that generates dynamic content on the server before it is sent to the browser. Apache hands `.php` files to PHP; PHP generates HTML and Apache ships it out. MySQL stores and manages the structured data the PHP application reads and writes. The LAMP stack is so common it has a name, and Metasploitable runs a complete instance of all four.

vsftpd (very secure FTP daemon) is the FTP server: a completely separate service from Apache that handles file transfers over port 21. It has no connection to the web layer. Samba implements Windows-style file and printer sharing (SMB protocol) on Linux, operating on ports 139 and 445.

**Key terms**

HTTP TRACE is a debugging method that echoes the full request, including headers, back to the sender. When paired with an XSS vulnerability it can enable session cookie theft. MultiViews is an Apache feature that auto-selects the best matching file when a request does not specify an extension. Cross-site scripting (XSS) is an injection attack where malicious script is embedded in a web page and runs in another user's browser.

```bash
# Basic scan against default port 80
nikto -h 192.168.79.130

# Save output to file (Nikto appends its own extension)
nikto -h 192.168.79.130 -p 80 -o ~/nikto_results.txt

# Verify saved file
ls -la ~/ | grep -i nikto
cat ~/nikto_results.txt.txt
```

---

## 📌 About This Project

This lab is part of the Week 2 vulnerability scanning and exploitation phase, covering the web layer that general-purpose scanners cannot reach on their own. The Nikto scan sits at the intersection of vulnerability discovery and exploitation preparation: it confirmed what version detection had flagged and added web-specific findings that network-focused tools do not surface. Understanding what Nikto finds, and what it cannot find, is part of developing the judgment needed to scope a real assessment properly.

The comparison with Nmap in Task 4 is the key takeaway: Nikto goes deep on HTTP, Nmap goes wide across all services. A complete vulnerability scanning picture requires both, and both feed into the assessment and exploitation decisions that follow.

**Related repositories (in order):**

- `Nmap-Host-Discovery-and-Lab-Baseline`: Established the initial host profile and port list that later scans built on
- `Service-Version-Detection-and-OS-Fingerprinting`: Identified Apache 2.2.8 on port 80 by version; Nikto independently confirmed that finding and expanded it into a full web-layer assessment
- `Nmap-Scripting-Engine`: NSE scripts confirmed and demonstrated exploitability of the vsftpd backdoor on port 21; that finding is outside what Nikto's HTTP scope can reach
- `OpenVAS-GVM-Vulnerability-Scan`: Automated scanner confirmed the Apache/PHP findings and added CVE-level scoring for the web vulnerabilities Nikto surfaced
- `Metasploit-First-Exploitation`: The vsftpd backdoor (the severity gap between what Nikto and Nmap each find) becomes the exploitation target in the next phase
- `MITRE-ATT-CK-Full-Kill-Chain-Mapping`: Maps all reconnaissance activity from both tools into a unified ATT&CK-framed kill chain

---

## 👤 Author

**Dashane James**  
Senior Field Service Technician → Cybersecurity Analyst  
📍 Yonkers, NY  
🎓 B.S. Information Technology, SUNY Canton  
🏆 CompTIA Security+  
🔗 [GitHub](https://github.com/Dashanejames1) | [Zero Trust Cyber Security Brand](https://www.instagram.com/zerotrust_cybersecurity)

*This repository is part of an active portfolio demonstrating hands-on cybersecurity skills. All lab work performed in isolated environments for educational purposes.*
