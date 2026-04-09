# 🖥️ Lab 1: Windows Active Directory — Enterprise Identity Management

## Overview

This lab builds a fully functional Active Directory domain from scratch using Windows Server 2025 and Windows 11 in a VMware Workstation Pro environment. It covers server configuration, AD DS installation, OU/user/group management, Group Policy, network drive mapping, USB restriction, and account lockout — all tasks performed daily by IT administrators and help desk staff.

---

## 🎯 Objectives

- Deploy Windows Server 2025 and configure it as a Domain Controller
- Install and promote Active Directory Domain Services (AD DS)
- Configure a static IP address and DNS
- Create an Organisational Unit (OU) structure mirroring a real company
- Join a Windows 11 client machine to the domain
- Configure Group Policy Objects (GPOs) for security enforcement
- Test account lockout policy by simulating failed logins
- Unlock a locked user account via Active Directory Users and Computers

---

## 🧰 Tools & Environment

| Component | Details |
|-----------|---------|
| **Hypervisor** | VMware Workstation Pro 25H2 (Build 24995812) |
| **Server OS** | Windows Server 2025 Standard Evaluation |
| **Client OS** | Windows 11 Pro (Version 25H2) |
| **Server Name** | FileServer01 |
| **Domain** | NavinHomeLab.Local |
| **Server IP** | 192.168.195.128 (Static) |
| **Client Name** | SYDCOMP01 |
| **Client IP** | 192.168.195.129 (DHCP) |
| **DNS** | 127.0.0.1 (server self-referencing) |

---

## 📐 Lab Topology

```
+-----------------------------+         +-----------------------------+
|   Windows Server 2025       |         |   Windows 11 Pro            |
|   FileServer01              |◄───────►|   SYDCOMP01                 |
|   NavinHomeLab.Local        |         |   SYDCOMP01.NavinHomeLab.Local
|   IP: 192.168.195.128       |         |   IP: 192.168.195.129       |
|   Roles: AD DS, DNS         |         |   User: navinram            |
+-----------------------------+         +-----------------------------+
              |
     [NavinHomeLab.Local]
         Domain Controller
         FileServer01.NavinHomeLab.Local
```

---

## 🔬 Methodology

### Step 1 — Windows Server 2025 Installation & Initial Setup

- Installed Windows Server 2025 Standard Evaluation in VMware Workstation Pro (Build 24995812)
- Server name set to `FileServer01`
- Opened **Server Manager → Dashboard** — confirmed server was running with File and Storage Services role

> 📸 *Screenshot: `image1.png` — Server Manager Dashboard showing FileServer01 running in VMware*

- Immediately ran **Windows Update** to apply all available patches before configuring any roles:
  - Security Intelligence Update for Microsoft Defender (KB2267602)
  - 2026-03 Security Update (KB5078740)
  - Windows Security platform update (KB5007651)
  - Windows Malicious Software Removal Tool (KB890830)

> 📸 *Screenshot: `image2.png` — Windows Update downloading 4 security patches*

---

### Step 2 — Enabling Remote Management

- Navigated to **Server Manager → Local Server**
- Confirmed Remote Management was **Disabled** by default

> 📸 *Screenshot: `image3.png` — Local Server properties showing Remote management: Disabled*

- Toggled Remote Management to **Enabled** and confirmed the change in Server Manager

> 📸 *Screenshot: `image4.png` — Local Server properties showing Remote management: Enabled*

---

### Step 3 — Installing Active Directory Domain Services

- Clicked **Manage → Add Roles and Features** in Server Manager
- Selected **Role-based or feature-based installation**

> 📸 *Screenshot: `image5.png` — Add Roles and Features Wizard: Select installation type*

- Selected `FileServer01` (192.168.195.128) from the server pool

> 📸 *Screenshot: `image6.png` — Server selection showing FileServer01 with Windows Server 2025*

- Checked **Active Directory Domain Services** from the server roles list

> 📸 *Screenshot: `image7.png` — Server Roles list with Active Directory Domain Services selected*

- Completed the wizard and promoted the server to a Domain Controller
- Selected **Add a new forest** and set the root domain name to `NavinHomeLab.Local`

> 📸 *Screenshot: `image8.png` — AD DS Configuration Wizard: Deployment Configuration showing NavinHomeLab.Local*

- Server restarted — logged back in as `NAVINHOMELAB\Administrator`, confirming the domain was active

> 📸 *Screenshot: `image9.png` — Windows login screen showing NAVINHOMELAB\Administrator*

- Verified all AD management tools were installed via **Windows Tools**:
  - Active Directory Users and Computers (ADUC)
  - Active Directory Domains and Trusts
  - Group Policy Management
  - DNS Manager

> 📸 *Screenshot: `image10.png` — Windows Tools showing all AD management tools installed*

---

### Step 4 — Configuring Static IP & DNS

- Opened **Settings → Network & Internet → Ethernet**
- IP was initially set to Automatic (DHCP)

> 📸 *Screenshot: `image11.png` — Ethernet settings showing IP assignment: Automatic (DHCP)*

- Ran `ipconfig` in Command Prompt to note current IP details:
  - IPv4: `192.168.195.128` | Subnet: `255.255.255.0` | Gateway: `192.168.195.2`

> 📸 *Screenshot: `image12.png` — ipconfig output confirming current IP configuration*

- Edited the adapter to set a **static IP**:
  - IP: `192.168.195.128`, Subnet: `255.255.255.0`, Gateway: `192.168.195.2`
  - DNS: `127.0.0.1` (server points to itself)

> 📸 *Screenshot: `image13.png` — Ethernet settings confirming static IP and manual DNS set to 127.0.0.1*

---

### Step 5 — OU Structure & Domain Join

- Opened **Active Directory Users and Computers (ADUC)**
- Created a top-level OU: `Nav-Aus` under `NavinHomeLab.Local`
- Inside `Nav-Aus`, created:
  - `Users` OU → sub-OUs: `IT`, `Finance`, `HRD`
  - `Systems` OU → for computer accounts

> 📸 *Screenshot: `image14.png` — ADUC showing Nav-Aus OU with Users (IT, Finance, HRD) and Systems*

- On the **Windows 11 client** (`SYDCOMP01`), confirmed system specs before domain join:
  - Device: SYDCOMP01 | RAM: 4GB | CPU: Intel Core Ultra 5 125H

> 📸 *Screenshot: `image15.png` — Windows 11 System About page showing SYDCOMP01 pre-domain join*

- Set client DNS to `192.168.195.128` (the DC) to enable domain discovery

> 📸 *Screenshot: `image16.png` — Client Ethernet settings showing DNS manually set to 192.168.195.128*

- Joined `SYDCOMP01` to `NavinHomeLab.Local` via System Properties
- After restart, logged in as domain user `navinram@NavinHomeLab.Local`
- Verified full domain name: `SYDCOMP01.NavinHomeLab.Local`

> 📸 *Screenshot: `image17.png` — Windows 11 About page showing SYDCOMP01.NavinHomeLab.Local, user navinram, Windows 11 Pro 25H2*

- Confirmed in ADUC that `SYDCOMP01` appeared as a Computer object under `Nav-Aus → Systems`

> 📸 *Screenshot: `image18.png` — ADUC showing SYDCOMP01 computer object under Nav-Aus → Systems*

---

### Step 6 — Group Policy Configuration

- Opened **Group Policy Management Console (GPMC)**
- Verified domain structure: Forest: NavinHomeLab.Local → Domains → NavinHomeLab.Local → Nav-Aus

> 📸 *Screenshot: `image19.png` — GPMC showing full domain structure with Nav-Aus OU*

#### GPO 1 — Default Domain Controllers Policy
- Reviewed the Default Domain Controllers Policy structure in the Group Policy Management Editor

> 📸 *Screenshot: `image20.png` — GPME showing Default Domain Controllers Policy for FILESERVER01.NAVINHOMELAB.LOCAL*

#### GPO 2 — Password Policy

- Navigated to: **Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy**
- Reviewed default state (all Not Defined)

> 📸 *Screenshot: `image21.png` — Account Policies node showing Password Policy, Account Lockout Policy, and Kerberos Policy*

> 📸 *Screenshot: `image22.png` — Password Policy with all settings at "Not Defined" before configuration*

- Configured the following settings:

| Policy | Setting |
|--------|---------|
| Enforce password history | 5 passwords remembered |
| Maximum password age | 60 days |
| Minimum password age | 30 days |
| Minimum password length | 12 characters |
| Password must meet complexity requirements | Enabled |

> 📸 *Screenshot: `image23.png` — Password Policy fully configured: 12 char minimum, complexity enabled, history 5, 60/30 day age*

#### GPO 3 — Drive Map (Network Drive)

- Created a GPO named `Drive Map`
- Navigated to: **User Configuration → Preferences → Windows Settings → Drive Maps**
- Created a new mapped drive:
  - Action: Update | Location: `\\Navinhomelab\ActiveDirectory` | Drive Letter: Z:

> 📸 *Screenshot: `image24.png` — Drive Maps GPO showing empty state before adding the drive*

> 📸 *Screenshot: `image25.png` — New Drive Properties: Z: drive mapped to \\Navinhomelab\ActiveDirectory, Action: Update*

> 📸 *Screenshot: `image26.png` — Drive Maps showing Z: drive configured with Action: Update and path \\Navinhomelab\ActiveDi...*

#### GPO 4 — Disable USB

- Created a GPO named `Disable USB` linked to `NavinHomeLab.Local`
- Navigated to: **Computer Configuration → Policies → Administrative Templates → System → Removable Storage Access**
- Enabled: **All Removable Storage classes: Deny all access**
- Security Filtering applied to: Authenticated Users

> 📸 *Screenshot: `image27.png` — GPMC showing Disable USB GPO linked to NavinHomeLab.Local, Removable Storage Access panel visible*

> 📸 *Screenshot: `image28.png` — Removable Storage Access: "All Removable Storage classes: Deny all access" set to Enabled*

---

### Step 7 — Account Lockout Policy & Testing

- Created a GPO named `Account Lockout Policy`
- Navigated to: **Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Account Lockout Policy**

> 📸 *Screenshot: `image29.png` — GPME showing Account Lockout Policy selected under Account Policies*

- Configured the following settings:

| Policy | Setting |
|--------|---------|
| Account lockout duration | 10 minutes |
| Account lockout threshold | 5 invalid logon attempts |
| Allow Administrator account lockout | Enabled |
| Reset account lockout counter after | 10 minutes |

> 📸 *Screenshot: `image30.png` — Account Lockout Policy configured: 10 min duration, 5 attempts, reset after 10 min*

- **Tested the policy** by entering the wrong password 5+ times on the Windows 11 client as `Navinkumar Ramalingam`
- The account locked and displayed: **"Invalid credentials, delaying next attempt..."**

> 📸 *Screenshot: `image31.png` — Windows 11 login screen showing "Invalid credentials, delaying next attempt..." for Navinkumar Ramalingam*

- On the Domain Controller, used ADUC **Find** to locate user `Navin`
- Opened user properties → **Account** tab → checked **Unlock account**
- Used **Reset Password** to set a new password for the user

> 📸 *Screenshot: `image32.png` — ADUC showing Navinkumar Ramalingam user properties with Unlock account checkbox*

> 📸 *Screenshot: `image33.png` — Reset Password dialog showing Account Lockout Status: Unlocked on Domain Controller FileServer01*

- Logged back into the Windows 11 client — **Welcome** screen confirmed successful login

> 📸 *Screenshot: `image34.png` — Windows 11 Welcome screen for Navinkumar Ramalingam confirming successful domain login*

---

## ✅ Results & Outcomes

| Task | Result |
|------|--------|
| Windows Server 2025 installed and patched | ✅ Success |
| Remote Management enabled | ✅ Success |
| AD DS installed, NavinHomeLab.Local domain created | ✅ Success |
| Static IP (192.168.195.128) and DNS configured | ✅ Success |
| OU structure: Nav-Aus → Users (IT/Finance/HRD) + Systems | ✅ Success |
| Windows 11 client SYDCOMP01 joined to domain | ✅ Success |
| SYDCOMP01 visible in ADUC under Systems OU | ✅ Success |
| Password Policy GPO: 12 char min, complexity, history | ✅ Success |
| Drive Map GPO: Z: → \\Navinhomelab\ActiveDirectory | ✅ Success |
| Disable USB GPO: All removable storage denied | ✅ Success |
| Account Lockout Policy: 5 attempts, 10 min lockout | ✅ Success |
| Account lockout triggered and confirmed on client | ✅ Success |
| Locked account unlocked and password reset via ADUC | ✅ Success |

---

## 💡 Key Takeaways

- **Patch before you configure** — Applying Windows Update before installing any roles follows real enterprise build standards and reduces attack surface from day one.
- **DNS is the backbone of Active Directory** — Setting the server DNS to `127.0.0.1` (itself) and pointing the Windows 11 client to the DC IP (`192.168.195.128`) were both critical. Without correct DNS, domain joins and Kerberos authentication fail.
- **OU design enables precise GPO targeting** — Separate OUs for IT, Finance, and HRD means policies can be applied per department. The Systems OU keeps computer objects organised and separately manageable.
- **GPO separation is good practice** — Having individual GPOs for Password Policy, Drive Map, USB restriction, and Account Lockout makes troubleshooting and auditing far easier than one monolithic policy.
- **Account lockout is a core help desk skill** — Configuring the lockout threshold AND knowing how to unlock accounts in ADUC and reset passwords are tasks that help desk staff perform every day.
- **USB restriction = data loss prevention** — The "Deny all removable storage" GPO is a real DLP control used by organisations to prevent data exfiltration and stop malware being introduced via USB devices.

---

## 🔗 Related Certifications

This lab directly maps to objectives from:
- **CompTIA A+** — OS configuration, domain environments, user account management
- **CompTIA Network+** — DNS, DHCP, static IP, network services
- **CompTIA Security+** — Identity management, Group Policy, access control, account lockout policies

---

[← Back to Portfolio](../../README.md)
