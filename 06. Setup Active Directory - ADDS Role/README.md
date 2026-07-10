# 🏢 Setting Up Active Directory Domain Services (AD DS)

> Step-by-step lab guide for installing AD DS, promoting a Windows Server 2019 machine to a Domain Controller, and understanding functional levels and DNS integration.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2019-0078D4?style=flat-square&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active%20Directory-AD%20DS-4CAF50?style=flat-square)
![DNS](https://img.shields.io/badge/DNS-Integrated-blueviolet?style=flat-square)
![Level](https://img.shields.io/badge/Level-Beginner-orange?style=flat-square)

---

## 📖 Overview

This lab transforms a freshly installed Windows Server 2019 machine into a fully functioning **Domain Controller (DC)**. You will install the AD DS role, promote the server, configure DNS, set functional levels, and verify the domain environment — all from Server Manager and PowerShell.

> 📌 **Pre-requisite:** Complete the [Windows Server 2019 installation lab](./README-lecture-winserver2019.md) first — static IP, computer rename, and correct date/time must already be configured.

---

## 🎯 What This Lab Covers

| Topic | Description |
|---|---|
| Domain vs Domain Controller | Key conceptual distinction before starting |
| Installing server roles | AD DS, DHCP, Windows Server Update Services |
| Promoting to Domain Controller | Running the AD DS Configuration Wizard |
| Domain name & NetBIOS | Naming conventions for internal networks |
| Forest & Domain functional levels | Version compatibility settings |
| DNS integration | Automatic DNS setup alongside AD DS |
| PowerShell automation | Exporting and running the deployment script |
| Post-install tasks | Restart, login, and verify DC promotion |

---

## 📜 Key Concept — Domain vs Domain Controller

| Term | Definition |
|---|---|
| **Domain** | A logical collection of objects — users, computers, printers — managed under a single administrative boundary |
| **Domain Controller (DC)** | The physical (or virtual) server running Windows Server that hosts AD DS and manages authentication and policies for the domain |
| **Forest** | The top-level container; one or more domains sharing a schema and global catalog |
| **Global Catalog** | A DC that holds a partial replica of all objects in the forest, enabling cross-domain searches |

> A domain is a concept. A Domain Controller is the machine that makes it real.

---

## 🔧 Step 1 — Install Required Server Roles

Open **Server Manager → Add Roles and Features** and install the following roles:

| Role | Purpose |
|---|---|
| **Active Directory Domain Services** | Core role — enables domain creation and management |
| **DNS Server** | Installed automatically with AD DS; handles name resolution |
| **DHCP Server** | Optional for labs — auto-assigns IPs to client machines |
| **Windows Server Update Services** | Optional — manages Windows updates across the domain |

```
Server Manager → Manage → Add Roles and Features
        ↓
Role-based or feature-based installation → Next
        ↓
Select destination server → Next
        ↓
Check "Active Directory Domain Services"
→ Click "Add Features" when prompted (includes management tools)
        ↓
Proceed through Features and AD DS info pages → Next
        ↓
Click Install → Wait for completion
        ↓
Do NOT close — click the yellow flag notification:
"Promote this server to a domain controller"
```

---

## ⚙️ Step 2 — AD DS Configuration Wizard

### Deployment Configuration

```
Select: "Add a new forest"
Root domain name: DC.local
        ↓
(Use .local for internal lab networks — never expose to public DNS)
```

### Domain Controller Options

```
Forest Functional Level:  Windows Server 2016
Domain Functional Level:  Windows Server 2016
        ↓
Options:
  ✅ Domain Name System (DNS) server
  ✅ Global Catalog (GC)
  ☐  Read-only domain controller (RODC)   ← leave unchecked
        ↓
Set DSRM password (Directory Services Restore Mode)
→ Use a strong password and store it safely
```

> ⚠️ The **DSRM password** is used to recover AD if the domain becomes unavailable. It is separate from the Administrator password.

### DNS Options

```
Create DNS delegation: No   ← correct for a new internal forest
```

### Additional Options

```
NetBIOS domain name: DC    ← auto-populated from domain name
```

### Paths (leave as defaults)

| Path | Default value |
|---|---|
| Database (NTDS) | `C:\Windows\NTDS` |
| Log files | `C:\Windows\NTDS` |
| SYSVOL | `C:\Windows\SYSVOL` |

### Review Options

The wizard summarizes all selections before installation. This matches the screenshot from the lab:

```
✔ Configure as first DC in a new forest
✔ Domain name: DC.local
✔ NetBIOS name: DC
✔ Forest Functional Level: Windows Server 2016
✔ Domain Functional Level: Windows Server 2016
✔ Global Catalog: Yes
✔ DNS Server: Yes
✔ Create DNS Delegation: No
```

> 💡 Click **"View script"** on the Review Options page to export the full PowerShell deployment script (see below).

---

## 💻 PowerShell Automation Script

The wizard generates a PowerShell script that can reproduce this exact deployment. This is the script exported from the lab (as seen in the screenshot):

```powershell
#
# Windows PowerShell script for AD DS Deployment
#

Import-Module ADDSDeployment
Install-ADDSForest `
    -CreateDnsDelegation:$false `
    -DatabasePath "C:\Windows\NTDS" `
    -DomainMode "WinThreshold" `
    -DomainName "DC.local" `
    -DomainNetbiosName "DC" `
    -ForestMode "WinThreshold" `
    -InstallDns:$true `
    -LogPath "C:\Windows\NTDS" `
    -NoRebootOnCompletion:$false `
    -SysvolPath "C:\Windows\SYSVOL" `
    -Force:$true
```

> `WinThreshold` is the internal code name for **Windows Server 2016** functional level.

To run this script on a new server instead of using the GUI:

```powershell
# Run PowerShell as Administrator
Set-ExecutionPolicy RemoteSigned
# Paste or run the script above
# The server will reboot automatically when done
```

---

## 🔑 Understanding Functional Levels

Functional levels determine the **minimum Windows Server version** that all Domain Controllers in the forest/domain must run. Setting it to Windows Server 2016 unlocks all modern AD features.

| Functional Level | Min DC version required | Key features unlocked |
|---|---|---|
| Windows Server 2008 R2 | Server 2008 R2 | Recycle Bin (basic) |
| Windows Server 2012 R2 | Server 2012 R2 | Protected Users group |
| Windows Server 2016 | Server 2016 | PAM trust, Azure AD support |

> For a new lab environment with no legacy DCs, always choose **Windows Server 2016** as both forest and domain functional level.

---

## 🔄 Step 3 — Prerequisites Check & Installation

```
Wizard runs Prerequisites Check automatically
        ↓
Review any warnings (yellow) — most are informational and safe to ignore
Resolve any errors (red) before proceeding
        ↓
Click Install
        ↓
Server installs AD DS binaries and configures the domain
        ↓
Server reboots automatically
```

---

## ✅ Step 4 — Post-Install Verification

After reboot, log in with the domain Administrator account:

```
Username: DC\Administrator    (or just Administrator)
Password: <your admin password>
```

Then verify the promotion was successful:

```powershell
# Check domain info
Get-ADDomain

# Check DC info
Get-ADDomainController

# Verify DNS is running
Get-Service DNS

# Verify AD DS is running
Get-Service ADWS, NTDS, Netlogon, DFSR
```

From Server Manager, you should now see:
- **AD DS** listed as an installed role
- **DNS** listed as an installed role
- The server name shown with domain suffix (e.g., `PDC22.DC.local`)

### Post-Install Checklist

- [ ] Server rebooted after promotion
- [ ] Login with domain Administrator credentials succeeds
- [ ] AD DS role visible in Server Manager
- [ ] DNS role visible in Server Manager
- [ ] `DC.local` domain resolvable via `nslookup DC.local`
- [ ] Active Directory Users and Computers (ADUC) opens without error
- [ ] VM snapshot taken after successful DC promotion

---

## 📊 Server Manager After DC Promotion

```
Server Manager Dashboard
├── AD DS        ✅ Running
├── DNS          ✅ Running
├── DHCP         ✅ Running (if installed)
└── Local Server
    ├── Computer Name:  PDC22.DC.local
    ├── Domain:         DC.local
    └── IP Address:     192.168.1.10 (static)

Tools menu (new entries after promotion):
├── Active Directory Users and Computers
├── Active Directory Sites and Services
├── Active Directory Domains and Trusts
├── DNS Manager
└── Group Policy Management
```

---

## 🧪 Lab Configuration Reference

| Setting | Value |
|---|---|
| Server name | `PDC22` |
| Domain name | `DC.local` |
| NetBIOS name | `DC` |
| Forest functional level | Windows Server 2016 |
| Domain functional level | Windows Server 2016 |
| Global Catalog | Yes |
| DNS Server | Yes (integrated) |
| DNS Delegation | No |
| NTDS path | `C:\Windows\NTDS` |
| SYSVOL path | `C:\Windows\SYSVOL` |

---

## 📝 Key Takeaways

- A **domain** is a logical boundary; a **Domain Controller** is the server that enforces it.
- Always set a **static IP** and correct **date/time** before promoting to DC — these cannot be easily changed after.
- The **DSRM password** is critical; store it securely outside the server.
- **DNS is inseparable from AD DS** — every domain controller is also a DNS server.
- Use **"View script"** in the wizard to save a PowerShell script for repeatable, automated deployments.
- Functional levels cannot be lowered after being set — choose **Windows Server 2016** for new environments.

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
