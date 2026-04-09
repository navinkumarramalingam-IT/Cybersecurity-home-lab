# 📊 Lab 2: Splunk SIEM — Security Monitoring & Log Analysis

📄 **[Download Full Lab PDF with Screenshots](./Splunk_SIEM_Lab_Portfolio.pdf)**

## Overview

This lab sets up Splunk Enterprise 10.2.2 as a SIEM platform on a Windows host machine, ingests multiple log sources including Windows Event Logs, performance metrics, and Linux authentication logs, and uses SPL (Splunk Processing Language) to search for and detect suspicious activity. The workflow mirrors what a SOC analyst does during a shift — onboarding data, writing detection queries, and investigating security events.

---

## 🎯 Objectives

- Install Splunk Enterprise 10.2.2 on Windows and access the Web UI
- Configure a receiving port (9997) for log forwarding
- Ingest multiple data sources: Windows Event Logs, Perfmon metrics, and Linux auth logs
- Write SPL queries to search for failed logins and SSH brute-force activity
- Investigate Windows Security Event Viewer logs
- Query and filter events across multiple indexes

---

## 🧰 Tools & Environment

| Component | Details |
|-----------|---------|
| **SIEM Platform** | Splunk Enterprise 10.2.2 for Windows |
| **Host Machine** | LAPTOP-HM3MTRGG |
| **Web UI** | localhost:8000 |
| **Receiving Port** | 9997 (enabled for log forwarding) |
| **Indexes Used** | `main`, `hoemlab`, `linux`, `sample_auth_log_1` |
| **Log Sources** | Windows Event Logs, Perfmon, tutorialdata.zip, auth.log, sample_auth_log.txt |
| **Sourcetypes** | WinEventLog:Security, Perfmon:CPU Load, Perfmon:Network Interface, Linux_auth, Linux Syslog, www1/secure |

---

## 🔬 Methodology

### Step 1 — Splunk Installation & Web UI Access

- Downloaded and installed **Splunk Enterprise 10.2.2 for Windows**
- Launched Splunk and accessed the Web UI at `localhost:8000`
- Logged in as Administrator — confirmed the Splunk Enterprise home dashboard loaded with Search & Reporting, Audit Trail, and other apps installed

> 📸 *Screenshot: `image2.png` — Splunk Enterprise home screen: "Hello, Administrator" showing installed apps*

---

### Step 2 — Configuring Settings & Receiving Port

- Navigated to **Settings** dropdown — explored available configuration areas:
  - Knowledge, Data, System, Distributed Environment, and Users & Authentication sections
  - Confirmed access to: Data inputs, Forwarding and receiving, Indexes, Source types

> 📸 *Screenshot: `image3.png` — Splunk Settings menu showing all configuration categories*

- Navigated to **Settings → Forwarding and Receiving → Receive data**
- Created a new receiving port: **9997**
- Confirmed the port was saved and showing as **Enabled**

> 📸 *Screenshot: `image4.png` — Receive data page showing port 9997 successfully saved and Enabled*

---

### Step 3 — Ingesting Data Sources

#### Data Source 1 — Tutorial Data (tutorialdata.zip)

- Used **Settings → Add Data → Upload** to upload `tutorialdata.zip`
- Configured input settings:
  - Source Type: Automatic
  - Host: `LAPTOP-HM3MTRGG` (Constant value)
  - Index: `hoemlab`

> 📸 *Screenshot: `image6.png` — Add Data wizard: selecting data source type (Upload, Monitor, Forward options)*

> 📸 *Screenshot: `image7.png` — Input Settings: Host field value set to LAPTOP-HM3MTRGG, Index set to Default*

- Reviewed the upload configuration before submitting:
  - Input Type: Uploaded File
  - File Name: tutorialdata.zip
  - Source Type: Automatic
  - Host: LAPTOP-HM3MTRGG
  - Index: hoemlab

> 📸 *Screenshot: `image8.png` — Add Data Review page confirming tutorialdata.zip upload settings*

- Submitted the upload — confirmed: **"File has been uploaded successfully"**

> 📸 *Screenshot: `image9.png` — "File has been uploaded successfully" confirmation with options to Start Searching or Build Dashboards*

---

### Step 4 — Searching & Querying Data

#### Search 1 — Verifying host data (Windows Event Logs + Perfmon)

Ran a broad search to confirm data was flowing in from `LAPTOP-HM3MTRGG`:

```spl
host="LAPTOP-HM3MTRGG"
```

- Returned **669 events** (4/5/26 7:00 AM to 4/6/26 7:28 AM)
- Events included Perfmon: Network Interface (Bytes Sent/sec, Bytes Received/sec) and Perfmon: CPU Load (% User Time)
- Confirmed multiple sourcetypes were ingesting correctly

> 📸 *Screenshot: `image5.png` — Search results for host="LAPTOP-HM3MTRGG" showing 669 events including Perfmon data*

#### Search 2 — Tutorial data: all events in hoemlab index

```spl
source="tutorialdata.zip:*" host="LAPTOP-HM3MTRGG" index="hoemlab"
```

- Returned **109,864 events** from the tutorial dataset
- Events included SSH failed password attempts from `mailsv1 sshd` against multiple invalid users (appserver, root, testuser, apache, mongodb, mail, games) all originating from `194.8.74.23`
- Source: `tutorialdata.zip:\mailsv\secure.log`, sourcetype: `www1/secure`

> 📸 *Screenshot: `image10.png` — 109,864 events in hoemlab index showing SSH failed password events from 194.8.74.23*

#### Search 3 — Filtering for failed events

```spl
source="tutorialdata.zip:*" host="LAPTOP-HM3MTRGG" index="hoemlab" fail*
```

- Filtered down to **33,253 events** matching `fail*`
- Results showed repeated SSH failed password attempts for multiple invalid users from the same attacker IP `194.8.74.23` — classic brute-force pattern

> 📸 *Screenshot: `image11.png` — 33,253 events matching fail* showing SSH brute-force pattern from 194.8.74.23*

#### Search 4 — Filtering SSH events specifically

```spl
source="tutorialdata.zip:*" host="LAPTOP-HM3MTRGG" index="hoemlab" ssh*
```

- Returned **38,950 events** matching `ssh*`
- Mix of failed password attempts and session events (pam_unix session closed for nsharpe)

> 📸 *Screenshot: `image12.png` — 38,950 SSH events showing both failed logins and session activity*

#### Search 5 — Windows Security Event Viewer

- Opened **Windows Event Viewer** on the host machine
- Navigated to **Windows Logs → Security**
- Observed **28,911 security events** — all showing as "Audit Success"
- Key event IDs visible: 4799 (group membership enumeration), 4672 (Special Logon), 4624 (Logon), 5382 (User Account Management)
- Expanded Event ID **4799**: "A security-enabled local group membership was enumerated"
  - Subject: `LAPTOP-HM3MTRGG\sugan`, Group: `BUILTIN\Administrators`
  - Process: `C:\Windows\System32\taskhostw.exe`

> 📸 *Screenshot: `image13.png` — Windows Event Viewer Security log showing 28,911 events with Event 4799 expanded (group membership enumeration by sugan)*

#### Search 6 — Searching all events in main index

```spl
index=main
```

- Returned **59,056 events** (first run) and **107,257 events** (after more data loaded)
- Sources included: Perfmon:Network Interface, Perfmon:CPU Load, Perfmon:Available Memory, WinEventLog:Security, WinEventLog:Application, WinEventLog:System

> 📸 *Screenshot: `image14.png` — index=main showing 59,056 events with Perfmon and WinEventLog sourcetypes*

> 📸 *Screenshot: `image15.png` — index=main showing 107,257 events with full sourcetype list in left panel*

#### Search 7 — Linux Authentication Log (auth.log)

```spl
source="auth.log" host="LAPTOP-HM3MTRGG" index="linux" sourcetype="Linux_auth"
```

- Returned **5,548 events**
- Log source: `auth.log` from a Linux host (`ip-10-77-20-248`)
- Events included:
  - `sudo` privilege escalation: `session opened for user root by ubuntu`
  - SSH brute-force: `Failed password for root from 73.231.4.205 port 54966 ssh2`
  - Authentication failures from `sshd`
  - CRON session events

> 📸 *Screenshot: `image16.png` — 5,548 Linux auth events showing sudo escalation, SSH failures, and CRON sessions from ip-10-77-20-248*

#### Search 8 — Real-time index=main monitoring

```spl
index=main
```

- Monitored live events as they streamed in — **72,158 events** visible during active ingestion
- Confirmed Perfmon sourcetypes (CPU Load, Network Interface, Available Memory) were updating in real time

> 📸 *Screenshot: `image17.png` — Live index=main search showing 72,158 events actively ingesting with Feb 3, 2026 highlighted in timeline*

#### Search 9 — Keyword search: "Failed Password" (exact phrase)

```spl
"Failed Password"
```

- Returned **0 events** — demonstrated that SPL is case-sensitive for exact phrase matching

> 📸 *Screenshot: `image18.png` — Search for "Failed Password" returning 0 events (case-sensitive, capital P)*

#### Search 10 — Keyword search: "Failed" (correct case)

```spl
"Failed"
```

- Returned **13,628 events**
- Events spanned multiple sourcetypes: WinEventLog:Security (failed logon attempts), WinEventLog:Application (HP Display Control Service installation failed), WinEventLog:System (TPM hardware failure)
- Highlighted the importance of correct SPL syntax and case sensitivity

> 📸 *Screenshot: `image19.png` — 13,628 events for "Failed" across Security, Application, and System logs*

#### Search 11 — Linux Syslog: Failed password in sample_auth_log

```spl
source="sample_auth_log.txt" host="LAPTOP-HM3MTRGG" index="sample_auth_log_1" sourcetype="Linux Syslog" "Failed password"
```

- Returned **6 events** — targeted search on a specific log file
- Events showed SSH failed password attempts for multiple invalid users:
  - `mysql` from 185.234.217.22 port 22
  - `oracle` from 185.234.217.22 port 22
  - `postgres` from 185.234.217.22 port 22
  - `guest` from 10.0.0.5 port 22
  - `root` from 192.168.1.10 port 22
  - `admin` from 192.168.1.10 port 22
- Multiple attackers targeting different service accounts — consistent with credential stuffing or automated scanning

> 📸 *Screenshot: `image20.png` — 6 events in sample_auth_log_1 index showing SSH failed password attempts for mysql, oracle, postgres, guest, root, admin*

---

## 🔍 Key SPL Queries Used

```spl
-- Verify host data is arriving
host="LAPTOP-HM3MTRGG"

-- Search all tutorial data
source="tutorialdata.zip:*" host="LAPTOP-HM3MTRGG" index="hoemlab"

-- Filter for failure events
source="tutorialdata.zip:*" host="LAPTOP-HM3MTRGG" index="hoemlab" fail*

-- Filter for SSH events
source="tutorialdata.zip:*" host="LAPTOP-HM3MTRGG" index="hoemlab" ssh*

-- Search all events in main index
index=main

-- Search Linux auth logs
source="auth.log" host="LAPTOP-HM3MTRGG" index="linux" sourcetype="Linux_auth"

-- Exact phrase search (case-sensitive)
"Failed Password"   -- returns 0 results (wrong case)
"Failed"            -- returns 13,628 results (correct case)

-- Targeted Linux Syslog search
source="sample_auth_log.txt" host="LAPTOP-HM3MTRGG" index="sample_auth_log_1" sourcetype="Linux Syslog" "Failed password"
```

---

## 🔍 Key Findings from Log Analysis

| Finding | Detail |
|---------|--------|
| SSH brute-force attack | IP `194.8.74.23` attempted logins against 8+ invalid usernames via `mailsv1 sshd` |
| Linux credential stuffing | Multiple IPs (`185.234.217.22`, `192.168.1.10`, `10.0.0.5`) targeting service accounts (mysql, oracle, postgres, admin, root) |
| Privilege escalation (Linux) | `ubuntu` user opened sudo session as root on `ip-10-77-20-248` |
| Windows group enumeration | Event 4799: user `sugan` enumerated `BUILTIN\Administrators` group membership |
| SPL case sensitivity | `"Failed Password"` = 0 results; `"Failed"` = 13,628 results — important for accurate detection queries |
| Perfmon monitoring active | CPU Load, Network Interface, and Available Memory all ingesting successfully from `LAPTOP-HM3MTRGG` |

---

## ✅ Results & Outcomes

| Task | Result |
|------|--------|
| Splunk Enterprise 10.2.2 installed and accessible at localhost:8000 | ✅ Success |
| Receiving port 9997 configured and enabled | ✅ Success |
| tutorialdata.zip uploaded to hoemlab index | ✅ Success |
| Windows Perfmon and Event Logs ingesting into main index | ✅ Success |
| Linux auth.log ingested into linux index | ✅ Success |
| Linux Syslog (sample_auth_log.txt) ingested into sample_auth_log_1 | ✅ Success |
| SSH brute-force pattern identified in tutorialdata | ✅ Success |
| Credential stuffing pattern identified in sample_auth_log | ✅ Success |
| Windows Security Event 4799 investigated in Event Viewer | ✅ Success |
| SPL case sensitivity understood and demonstrated | ✅ Success |

---

## 💡 Key Takeaways

- **Data onboarding is step one** — Before any detection is possible, logs must be correctly indexed and searchable. Getting the source, host, index, and sourcetype right is fundamental to everything that follows.
- **SPL is case-sensitive for exact phrases** — Searching `"Failed Password"` vs `"Failed"` returned 0 vs 13,628 results. This is a critical lesson: a detection query written with the wrong case silently misses every event.
- **Brute-force attacks leave clear patterns** — The tutorialdata showed one IP (`194.8.74.23`) hammering multiple usernames (appserver, root, testuser, apache, mongodb) within seconds — exactly the kind of pattern SOC analysts look for in SIEM tools.
- **Multiple log sources = better visibility** — Combining Windows Event Logs, Perfmon metrics, and Linux auth logs gives a broader picture of what's happening across the environment. A single source is never enough.
- **Targeted searches are more powerful** — Adding `index=`, `source=`, `sourcetype=`, and `host=` filters dramatically narrows results and speeds up investigation. Learning to be specific is key to working efficiently as a SOC analyst.
- **Event IDs matter** — Seeing Event 4799 (group membership enumeration) in Windows Event Viewer and knowing what it means (potential reconnaissance) is exactly the kind of contextual knowledge that separates a good SOC analyst from someone just running searches.

---

## 🔗 Related Certifications

This lab directly maps to objectives from:
- **CompTIA Security+** — Threat detection, log analysis, SIEM tools, incident investigation
- **CompTIA Network+** — Log sources, syslog, network monitoring, SSH
- **CompTIA A+** — Windows Event Viewer, system logs, OS administration

---

[← Back to Portfolio](../../README.md)
