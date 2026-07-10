# 🔗 Joining a Windows Machine to a Domain

> Step-by-step lab guide covering pre-join configuration, the AAA security model, DNS requirements, Kerberos authentication, and login methods after joining a Windows domain.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2019-0078D4?style=flat-square&logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active%20Directory-Domain%20Join-4CAF50?style=flat-square)
![Course](https://img.shields.io/badge/Session-08-blueviolet?style=flat-square)
![Level](https://img.shields.io/badge/Level-Beginner-orange?style=flat-square)

---

## 📖 Overview

This is **Session 8** of the Windows Server 2019 course. Building on the Domain Controller and Active Directory setup from previous sessions, this lab walks through joining a Windows client machine (named **Windows-PC**) to the `test.local` domain — covering every technical step from pre-join configuration to post-join login methods.

> 📌 **Pre-requisite:** A working Domain Controller with AD DS installed and the `test.local` domain configured. The client machine must be able to reach the DC on the network.

---

## 🎯 What This Lab Covers

| Topic | Description |
|---|---|
| Pre-join configuration | Date/time, static IP, computer name |
| DNS requirement | Why DNS must point to the DC |
| Initiating the domain join | Switching from workgroup to domain |
| AAA security model | Authentication, Authorization, Accounting |
| Kerberos protocol | How credentials are verified |
| Computer account placement | Default Computers container vs OUs |
| Post-join login | Domain login vs local login syntax |

---

## 🖧 Lab Network Setup

```
┌─────────────────┐        ┌──────────────┐
│  Windows-PC     │        │  Switch      │
│  192.168.1.30   ├────────┤              ├─────── DC1  192.168.1.10
│  DNS: 192.168.1.10       │              ├─────── DC2  192.168.1.11
└─────────────────┘        └──────────────┘

Domain: test.local
```

All machines must be on the **same subnet** (`192.168.1.x / 255.255.255.0`) to communicate.

---

## ⚙️ Step 1 — Pre-Join Configuration (3 Required Steps)

Before attempting to join the domain, three configurations must be completed on the client machine. Skipping any one of them will cause the join to fail.

### 1. Set Date, Time, and Timezone

```
Right-click taskbar clock → Adjust date/time
└── Set correct timezone, date, and time
```

> ⚠️ Kerberos authentication requires the client and DC clocks to be within **5 minutes** of each other. A time mismatch will cause authentication to fail.

### 2. Assign a Static IP Address

DHCP is not yet active in this environment, so the IP must be set manually.

```
Settings → Network → Change adapter options
→ Right-click adapter → Properties
→ IPv4 → Use the following IP address:

  IP Address:      192.168.1.30
  Subnet Mask:     255.255.255.0
  Default Gateway: 192.168.1.1
  DNS Server:      192.168.1.10   ← must point to the DC
```

> ⚠️ The **DNS server must be set to the Domain Controller's IP address** — not Google DNS or any external server. The domain name `test.local` only exists in the DC's DNS zone, so external DNS servers cannot resolve it.

### 3. Set the Computer Name

```
Right-click This PC → Properties → Rename this PC
└── Set a clear, unique name (e.g., Windows-PC)
→ Restart when prompted
```

---

## 🌐 Why DNS is Critical

When you type `test.local` during the domain join, Windows needs to find the IP address of the Domain Controller. This translation from name → IP is handled entirely by **DNS**.

```
Client types: test.local
        ↓
DNS query sent to: 192.168.1.10 (the DC)
        ↓
DC DNS responds: "test.local = 192.168.1.10"
        ↓
Client contacts DC and begins join process
```

If DNS is misconfigured or points to an external server, the client cannot resolve `test.local` and the join fails immediately — even if the network connection is working fine.

---

## 🔗 Step 2 — Initiate the Domain Join

```
Right-click This PC → Properties
→ Change settings (next to computer name)
→ Change...
        ↓
Member of:
  ○ Workgroup: WORKGROUP
  ● Domain:    test.local        ← switch to Domain, enter domain name
        ↓
Click OK
        ↓
Credential prompt appears → enter Domain Admin credentials:
  Username: Administrator  (or any user with join rights)
  Password: ••••••••
        ↓
"Welcome to the test.local domain" message appears
        ↓
Restart the machine
```

---

## 🔐 Step 3 — The AAA Security Model

When the client attempts to join the domain, the Domain Controller runs three sequential security checks. All three must pass for the join to succeed.

```
Client → DC: "I want to join the domain"
        ↓
┌─────────────────────────────────────────────────────────┐
│  1. AUTHENTICATION — Who are you?                        │
│     Verify username + password against AD database       │
│     Protocol: Kerberos                                   │
├─────────────────────────────────────────────────────────┤
│  2. AUTHORIZATION — Are you allowed to do this?          │
│     Check if user has permission to join computers        │
│     Default: regular users can join up to 10 machines    │
│     Admin: no limit                                      │
├─────────────────────────────────────────────────────────┤
│  3. ACCOUNTING — Is the computer name unique?            │
│     Check AD for existing computer accounts              │
│     Duplicate names → join rejected                      │
│     Unique name → computer account created in AD         │
└─────────────────────────────────────────────────────────┘
        ↓
All 3 pass → Join completes → Restart
```

### Authentication vs Authorization — Analogy

| Concept | University analogy | Domain join equivalent |
|---|---|---|
| **Authentication** | Proving you are a registered student at the university | Username and password verified in Active Directory |
| **Authorization** | Being allowed to enter a specific faculty or lab | User account has permission to join computers to domain |
| **Accounting** | Your student ID is unique — no duplicate records | Computer name must not already exist in AD |

### Kerberos Protocol

Kerberos is the authentication backbone of Active Directory. It works using **tickets** rather than transmitting passwords over the network:

```
1. Client sends encrypted request to DC (Key Distribution Center)
2. DC verifies identity and issues a Ticket Granting Ticket (TGT)
3. Client uses TGT to request a Service Ticket for the domain join
4. DC grants access — join proceeds
```

> DNS service records (`_kerberos`, `_ldap`) must be present in the DC's DNS zone for Kerberos to function correctly. These are created automatically when AD DS is installed.

---

## 🗂️ Step 4 — Computer Account Placement in Active Directory

When the join completes, a **computer account** is automatically created in Active Directory.

### Default Behavior

```
AD object created at: DC.local/Computers   ← default container, NOT an OU
```

> ⚠️ The default `Computers` container **does not support Group Policy**. Policies applied to an OU will not affect machines sitting in the default Computers container.

### Best Practice — Move to an OU

```
Active Directory Users and Computers (ADUC)
→ Locate computer in Computers container
→ Right-click → Move → select target OU (e.g., IT-Computers)
```

### Pre-creating Computer Accounts

Administrators can create the computer account in the correct OU **before** the machine joins. When the machine joins, it finds the pre-created account and slots into the right OU automatically — inheriting the correct Group Policies immediately.

```
ADUC → Right-click IT-Computers → New → Computer
→ Enter computer name (must match exactly)
→ OK
```

| Approach | OU placement | Group Policy applied |
|---|---|---|
| Default join (no pre-creation) | `Computers` container | ❌ Not immediately |
| Move after join | Target OU | ✅ After move + gpupdate |
| Pre-create account in OU | Target OU | ✅ Immediately on join |

---

## 💻 Step 5 — Post-Join Login Methods

After restart, the machine is domain-joined. The user can log in with either a **domain account** or a **local account**.

### Domain Login

```
Format 1:   DOMAIN\username      →   test\ahmed.saad
Format 2:   username@domain      →   ahmed.saad@test.local
```

### Local Login

```
Format:     .\local_username     →   .\ITAdmin
            or
            MACHINENAME\username →   WINDOWS-PC\ITAdmin
```

The `.\` prefix explicitly tells Windows to look at the **local machine's account database** instead of the domain.

### Login Type Comparison

| Login type | Format | Uses | Group Policy applied |
|---|---|---|---|
| Domain account | `test\ahmed.saad` | Daily user access | ✅ Yes — from DC |
| Domain admin | `test\Administrator` | Domain management | ✅ Yes |
| Local account | `.\ITAdmin` | Local machine management only | ❌ No — local only |

> 💡 Local administrator accounts (e.g., `.\ITAdmin`) should be managed **exclusively by IT**. End users should not know the local admin password. This account is a fallback for when the machine cannot reach the domain.

---

## ✅ Lab Completion Checklist

- [ ] Date, time, and timezone set correctly on client machine
- [ ] Static IP assigned in same subnet as DC (`192.168.1.x`)
- [ ] DNS server set to DC's IP (`192.168.1.10`)
- [ ] Computer name set and machine restarted
- [ ] Domain join initiated via System Properties
- [ ] Domain admin credentials entered successfully
- [ ] "Welcome to the domain" message received
- [ ] Machine restarted after join
- [ ] Computer account visible in ADUC under Computers container
- [ ] Computer account moved to correct OU (e.g., IT-Computers)
- [ ] Domain login tested: `test\ahmed.saad`
- [ ] Local login tested: `.\ITAdmin`
- [ ] VM snapshot taken after successful join

---

## 📚 Terminology Reference

| Term | Definition |
|---|---|
| **Kerberos** | Authentication protocol used by Active Directory; uses encrypted tickets instead of transmitting passwords |
| **DNS** | Translates domain names (test.local) to IP addresses; must point to the DC |
| **Authentication** | Verifying who a user is — username and password check against AD |
| **Authorization** | Verifying what a user is allowed to do — permission to join computers |
| **Accounting** | Ensuring object uniqueness — no duplicate computer names in the domain |
| **AAA** | Authentication, Authorization, Accounting — the three-step security model for domain join |
| **Workgroup** | Peer-to-peer network model with no central authentication |
| **Computer account** | AD object representing a machine joined to the domain |
| **OU** | Organizational Unit — container in AD that supports Group Policy application |
| **TGT** | Ticket Granting Ticket — issued by Kerberos after successful authentication |

---

## 🔭 Next Session Preview

- Applying **Group Policy Objects (GPOs)** to OUs
- How policies propagate from DC to domain-joined machines
- Using `gpupdate /force` and `gpresult` to verify policy application

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
