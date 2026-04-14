# Cybersecurity Portfolio — Network Traffic Analysis
**Analyst:** [Your Name]  
**Lab Environment:** VMware Workstation 25H2  
**Date:** April 2026  
**Tools:** Wireshark, nmap, Kali Linux, Metasploitable 2  

---

## Lab Environment

| Machine | OS | IP Address | Role |
|---|---|---|---|
| Attacker | Kali Linux 2026.1 | 192.168.128.130 | Attack machine |
| Target | Metasploitable 2 | 192.168.128.129 | Vulnerable target |
| Victim | Windows 11 x64 | 192.168.128.131 | Victim workstation |
| Analyst | Windows (Host) | — | Wireshark capture (VMnet1) |

**Network:** VMware Host-Only (VMnet1) — fully isolated, no internet access  
**Capture interface:** VMware Network Adapter VMnet1 (local Wireshark)

---

## Project 1 — Port Scan Detection

### Objective
Detect and analyse a TCP SYN port scan performed by an attacker against a vulnerable target, identify indicators of compromise, and document the network signature of the attack.

### Attack Simulation

Tool used: **nmap 7.x** on Kali Linux  
Command run:
```bash
nmap -sS 192.168.128.129
```

A SYN scan (also called a half-open scan) sends TCP SYN packets to each port but never completes the three-way handshake. This makes it faster and stealthier than a full connect scan.

### Detection in Wireshark

**Capture interface:** VMnet1  
**Display filter applied:**
```
tcp.flags.syn==1 && tcp.flags.ack==0
```

**What was observed:**
- Source IP `192.168.128.130` (Kali) sent SYN packets to `192.168.128.129` (Metasploitable) across 1000+ destination ports
- All packets were sent within a few seconds — consistent with automated scanning
- No TCP ACK responses completing the handshake — confirming half-open SYN scan technique
- Statistics → Conversations → TCP tab showed over 1000 unique port conversations from a single source IP

**Key packet characteristics:**

| Field | Value |
|---|---|
| Source IP | 192.168.128.130 |
| Destination IP | 192.168.128.129 |
| TCP Flag | SYN only (0x002) |
| Packet size | 60 bytes (consistent) |
| Time between packets | Milliseconds |
| Unique destination ports | 1000+ |

### Findings — Indicators of Compromise (IOCs)

| Indicator | Value |
|---|---|
| Attacker IP | 192.168.128.130 |
| Target IP | 192.168.128.129 |
| Scan type | TCP SYN (half-open) |
| Ports probed | 1–1000+ |
| Tool fingerprint | nmap default timing |
| Duration | ~30–60 seconds |
| Packets generated | 1000+ SYN packets |

### Open Ports Discovered on Metasploitable 2

| Port | Service | Risk |
|---|---|---|
| 21 | FTP (vsFTPd 2.3.4) | Critical — backdoor vulnerability |
| 22 | SSH (OpenSSH 4.7) | Medium — outdated version |
| 23 | Telnet | High — plaintext protocol |
| 80 | HTTP (Apache 2.2) | High — DVWA web app |
| 139/445 | SMB (Samba 3.x) | Critical — known exploits |
| 3306 | MySQL 5.0 | High — no root password |
| 5432 | PostgreSQL 8.3 | Medium |

### Recommendations

1. Deploy an IDS (Suricata or Snort) with SYN flood detection rules
2. Configure firewall to rate-limit inbound SYN packets per source IP
3. Alert on any host contacting more than 50 unique ports within 10 seconds
4. Block port scanning tools at the network perimeter
5. Patch or decommission all services running vulnerable versions

---

## Project 2 — Credential Sniffing (FTP)

### Objective
Demonstrate that credentials transmitted over unencrypted protocols (FTP) are fully visible to a network observer, and capture the plaintext username and password using Wireshark.

### Attack Simulation

Tool used: **ftp** client on Kali Linux  
Command run:
```bash
ftp 192.168.128.129
```

Logged in with default Metasploitable credentials:
```
Username: msfadmin
Password: msfadmin
```

Ran `ls` to list directory contents, then `quit` to close the session.

### Detection in Wireshark

**Capture interface:** VMnet1  
**Display filter applied:**
```
ftp
```

**Follow TCP Stream output:**
```
220 (vsFTPd 2.3.4)
USER msfadmin
331 Please specify the password.
PASS msfadmin
230 Login successful.
SYST
215 UNIX Type: L8
FEAT
211-Features:
 EPRT EPSV MDTM PASV REST STREAM SIZE TVFS UTF8
211 End
EPSV
229 Entering Extended Passive Mode (|||56598|)
LIST
150 Here comes the directory listing.
226 Directory send OK.
QUIT
221 Goodbye.
```

**What was observed:**
- Username (`msfadmin`) transmitted in plaintext — visible in `USER` command
- Password (`msfadmin`) transmitted in plaintext — visible in `PASS` command
- Full session including directory listing captured without any decryption
- Server banner (`vsFTPd 2.3.4`) exposed — this version has a known backdoor (CVE-2011-2523)

**Colour coding in TCP stream:**
- Red text = data sent by client (Kali attacker)
- Blue text = responses from server (Metasploitable)

### Findings — Indicators of Compromise (IOCs)

| Indicator | Value |
|---|---|
| Attacker IP | 192.168.128.130 |
| Target IP | 192.168.128.129 |
| Protocol | FTP (port 21) |
| Captured username | msfadmin |
| Captured password | msfadmin |
| Server software | vsFTPd 2.3.4 |
| CVE | CVE-2011-2523 (vsFTPd backdoor) |
| Encryption | None — fully plaintext |

### Why This Is Dangerous

FTP transmits all data including credentials in plaintext. Any attacker with access to the network path between client and server — via a compromised switch, ARP spoofing, or a rogue access point — can capture valid credentials without the user knowing. These credentials can then be reused across other services (credential stuffing).

### Recommendations

1. Disable FTP entirely — replace with SFTP (SSH File Transfer Protocol) or FTPS
2. Never reuse credentials across services
3. Implement network monitoring to alert on FTP traffic on production networks
4. Deploy 802.1X port authentication to prevent unauthorised network taps
5. Patch vsFTPd immediately — version 2.3.4 contains a backdoor on port 6200

---

## Skills Demonstrated

| Skill | Evidence |
|---|---|
| Network traffic capture | Wireshark on VMnet1 |
| Display filter usage | `tcp.flags.syn==1`, `ftp` filters |
| Attack simulation | nmap SYN scan, FTP login |
| IOC identification | Attacker IP, credentials, CVEs |
| Threat analysis | Explained attack techniques |
| Remediation advice | Specific, actionable recommendations |
| Lab setup | VMware, 3-VM isolated network |

---

## Files in This Repository

```
/wireshark-analysis/
  README.md                        ← this file
  port-scan-detection.pcapng       ← Wireshark capture (Project 1)
  credential-sniffing.pcapng       ← Wireshark capture (Project 2)
  /screenshots/
    project1-syn-filter.png
    project1-conversations.png
    project2-ftp-stream.png
```

---

## Next Projects (Coming Soon)

- [ ] C2 beaconing simulation using Metasploit Meterpreter
- [ ] DNS tunneling detection
- [ ] Lateral movement via SMB
- [ ] Suricata IDS alert correlation

---

*This portfolio was built in an isolated VMware lab environment. All attacks were performed against intentionally vulnerable machines. No real systems were harmed.*
