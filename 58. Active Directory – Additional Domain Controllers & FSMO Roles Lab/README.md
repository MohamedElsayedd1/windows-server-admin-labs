# 🏛️ Active Directory – Additional Domain Controllers & FSMO Roles Lab

A comprehensive lab covering **Additional Domain Controllers (ADC)**, **Active Directory replication**, and **FSMO (Flexible Single Master Operations) role management** in a Windows Server environment. This lab teaches how to build a resilient AD infrastructure, understand each FSMO role's purpose, and manage planned transfers and emergency seizures.

---

## 📋 Table of Contents

1. [Background & Architecture](#-background--architecture)
2. [Lab Environment](#-lab-environment)
3. [Task 1 – Understand the Need for an Additional Domain Controller](#task-1--understand-the-need-for-an-additional-domain-controller)
4. [Task 2 – Install AD DS Role on the Secondary Server](#task-2--install-ad-ds-role-on-the-secondary-server)
5. [Task 3 – Promote the Secondary Server to an Additional Domain Controller](#task-3--promote-the-secondary-server-to-an-additional-domain-controller)
6. [Task 4 – Verify Active Directory Replication](#task-4--verify-active-directory-replication)
7. [Task 5 – Identify Current FSMO Role Holders](#task-5--identify-current-fsmo-role-holders)
8. [Task 6 – Transfer FSMO Roles (Planned Migration)](#task-6--transfer-fsmo-roles-planned-migration)
9. [Task 7 – Seize FSMO Roles (Emergency Takeover)](#task-7--seize-fsmo-roles-emergency-takeover)
10. [Task 8 – Verify FSMO Role Assignment After Transfer/Seizure](#task-8--verify-fsmo-role-assignment-after-transferseizure)
11. [Summary](#-summary)
12. [FSMO Roles Reference](#-fsmo-roles-reference)
13. [Key Concepts](#-key-concepts)
14. [Transfer vs Seizure Decision Guide](#-transfer-vs-seizure-decision-guide)
15. [Troubleshooting](#️-troubleshooting)

---

## 🏗️ Background & Architecture

### Why Additional Domain Controllers?

In a production Active Directory environment, relying on a **single Domain Controller (DC)** is a critical risk. If that server fails, the entire domain becomes unavailable — users cannot log in, computers cannot authenticate, and all domain services stop functioning. Recovery from a system state backup alone can take **several hours**, which is unacceptable in most business environments.

The solution is an **Additional Domain Controller (ADC)** — a second (or third) server running the AD DS role in the same domain. It maintains a **full replica** of the Active Directory database, enabling instant failover when the primary server goes down.

```
┌─────────────────────────────────────────────────────────┐
│                    AD Forest: company.local              │
│                                                         │
│   ┌──────────────────┐     Replication     ┌──────────────────┐  │
│   │   PDC (Primary)  │ ◄──────────────────► │  ADC (Secondary) │  │
│   │  192.168.1.1     │                     │  192.168.1.2     │  │
│   │                  │                     │                  │  │
│   │  FSMO Roles:     │                     │  Full AD Replica  │  │
│   │  - PDC Emulator  │                     │  - Users         │  │
│   │  - RID Master    │                     │  - Groups        │  │
│   │  - Infra Master  │                     │  - Computers     │  │
│   │  - Domain Naming │                     │  - Policies      │  │
│   │  - Schema Master │                     │  - GPOs          │  │
│   └──────────────────┘                     └──────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### FSMO Roles Overview

The **Five FSMO Roles** are specialized operations that only **one Domain Controller at a time** can perform. They prevent conflicts that would occur if multiple DCs tried to handle the same task simultaneously.

```
┌─────────────────────────────────────────────────────────────────┐
│                        FSMO ROLES                               │
├─────────────────────────────────┬───────────────────────────────┤
│         DOMAIN-LEVEL (3)        │        FOREST-LEVEL (2)       │
├─────────────────────────────────┼───────────────────────────────┤
│  1. PDC Emulator                │  4. Domain Naming Master      │
│     Passwords, time sync,       │     Add/remove domains        │
│     legacy auth                 │     in the forest             │
│                                 │                               │
│  2. RID Master                  │  5. Schema Master             │
│     Unique ID pool for          │     AD schema changes         │
│     all AD objects              │     and updates               │
│                                 │                               │
│  3. Infrastructure Master       │                               │
│     Cross-domain group          │                               │
│     membership updates          │                               │
└─────────────────────────────────┴───────────────────────────────┘
```

---

## 🖥️ Lab Environment

| Component | Details |
|-----------|---------|
| **Primary DC (PDC)** | SERVER-PDC — Windows Server 2022 |
| **Additional DC (ADC)** | SERVER-ADC — Windows Server 2022 |
| **Domain** | company.local |
| **PDC IP** | 192.168.1.1 |
| **ADC IP** | 192.168.1.2 |
| **Forest Functional Level** | Windows Server 2016 |
| **Domain Functional Level** | Windows Server 2016 |
| **Tools Used** | Server Manager, Active Directory Users and Computers, Active Directory Domains and Trusts, Active Directory Schema snap-in, PowerShell, `ntdsutil` |

---

## Task 1 – Understand the Need for an Additional Domain Controller

### 📖 Explanation
Before deploying an ADC, it is important to understand the **business justification** and technical benefits. A single DC environment creates a **Single Point of Failure (SPOF)** — if the PDC crashes, all of the following stop working:

- **User authentication** — no one can log in to any domain-joined machine
- **Computer authentication** — domain-joined computers cannot apply Group Policy
- **Kerberos ticket issuance** — all network resource access requires a working DC
- **Password changes** — users cannot change or reset passwords
- **DNS (if hosted on the DC)** — name resolution may fail for the entire network

An ADC eliminates this risk by maintaining a **live, synchronized replica** of the entire AD database. Users authenticate against whichever DC responds first — load is distributed and failover is instantaneous.

### 🔧 Steps (Planning)
1. Identify the **number of users and sites** — multiple physical locations typically need a DC per site
2. Determine **network bandwidth** between sites — replication traffic must be accounted for
3. Prepare a second Windows Server instance (physical or virtual) in the same network
4. Ensure the secondary server:
   - Has a **static IP address** (e.g., 192.168.1.2)
   - Points to the **PDC as its Primary DNS** (192.168.1.1)
   - Has network connectivity to the PDC
   - Has adequate resources (minimum 4 GB RAM, 40 GB disk for AD database)
5. Plan which FSMO roles will remain on the PDC vs be transferred to the ADC

### ✅ Solution / Expected Result
A deployment plan is in place: the secondary server (SERVER-ADC) is provisioned with a static IP, pointing to the PDC for DNS, and has network connectivity verified (`ping 192.168.1.1` succeeds from SERVER-ADC).

> **Best Practice:** Always have a **minimum of two Domain Controllers** per domain. For sites with more than 100 users or mission-critical applications, consider three or more DCs.

---

## Task 2 – Install AD DS Role on the Secondary Server

### 📖 Explanation
Before promoting a server to a Domain Controller, the **Active Directory Domain Services (AD DS)** role must be installed. Installing the role copies all the necessary binaries and management tools to the server but does **not yet make it a DC** — that happens during promotion (Task 3). This two-step process allows you to install the role offline and promote later, or verify the role installation before committing to promotion.

### 🔧 Steps

**Method 1: Via Server Manager (GUI)**
1. Log on to **SERVER-ADC** as a local Administrator
2. Open **Server Manager** → click **Manage** → **Add Roles and Features**
3. Click **Next** through: Before You Begin → Installation Type (Role-based) → Server Selection (SERVER-ADC)
4. On the **Server Roles** page, check **Active Directory Domain Services**
5. When prompted to add required features, click **Add Features**
6. Click **Next** through Features and AD DS information pages
7. Click **Install** and wait for completion
8. Do **NOT** click "Promote this server to a domain controller" yet — close the wizard

**Method 2: Via PowerShell**
```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools
```

### ✅ Solution / Expected Result
Server Manager shows the AD DS role as installed on SERVER-ADC with a status of **"Installation succeeded"**. A yellow notification flag appears in Server Manager prompting you to promote the server — this is the expected state before Task 3.

> **Note:** Installing the AD DS role also installs the following management tools automatically: Active Directory Users and Computers, Active Directory Sites and Services, Active Directory Domains and Trusts, and Group Policy Management Console.

---

## Task 3 – Promote the Secondary Server to an Additional Domain Controller

### 📖 Explanation
**Promotion** is the process that transforms a server with the AD DS role into an actual Domain Controller. During promotion, the server:
1. Connects to the existing PDC
2. **Replicates the entire AD database** (users, groups, computers, OUs, GPOs, DNS zones)
3. Installs the **SYSVOL** folder (Group Policy templates and scripts)
4. Configures itself as a DC in the existing domain
5. Optionally becomes a **DNS server** and/or **Global Catalog server**

Promotion joins the new DC into the existing domain as an **Additional DC** — it does not create a new domain.

### 🔧 Steps

**Via Server Manager (GUI):**
1. In Server Manager on SERVER-ADC, click the yellow **notification flag** → click **"Promote this server to a domain controller"**
2. The **Active Directory Domain Services Configuration Wizard** opens
3. On **Deployment Configuration**:
   - Select: **Add a domain controller to an existing domain**
   - Domain: `company.local`
   - Click **Change…** and provide **Domain Admin credentials** (COMPANY\Administrator)
   - Click **Next**
4. On **Domain Controller Options**:
   - ✅ **Domain Name System (DNS) server** — check this to make the ADC also a DNS server
   - ✅ **Global Catalog (GC)** — check this for full AD object replication
   - Set the **Directory Services Restore Mode (DSRM) password** — this is critical, store it safely
   - Click **Next**
5. On **DNS Options** — click **Next** (ignore delegation warning in lab)
6. On **Additional Options**:
   - **Replicate from:** Select the PDC (`SERVER-PDC.company.local`) or leave as "Any domain controller"
   - Click **Next**
7. On **Paths** — leave default paths for AD database, logs, and SYSVOL
8. Review the **Review Options** summary page
9. Click **Install** — the server will automatically **restart** after promotion

**Via PowerShell:**
```powershell
Install-ADDSDomainController `
    -DomainName "company.local" `
    -Credential (Get-Credential) `
    -InstallDns:$true `
    -NoGlobalCatalog:$false `
    -Force:$true
```

### ✅ Solution / Expected Result
After the automatic restart, SERVER-ADC is now a Domain Controller. Log in with **Domain Administrator** credentials. Open **Active Directory Users and Computers** → navigate to **Domain Controllers** OU — both `SERVER-PDC` and `SERVER-ADC` should appear as domain controllers. DNS zones should also be replicated and visible on SERVER-ADC.

> **DSRM Password:** The Directory Services Restore Mode password allows you to boot the DC into a recovery mode if AD becomes corrupted. It is separate from the domain admin password. Store it in a password manager — losing it can make recovery impossible.

---

## Task 4 – Verify Active Directory Replication

### 📖 Explanation
After promotion, **replication** should begin automatically. AD replication copies all directory objects (users, groups, computers, OUs, GPOs, DNS records) from the existing DCs to the new ADC, and keeps them synchronized as changes occur. Verifying replication confirms the ADC is fully functional and in sync with the PDC.

Replication in AD is **multi-master** — changes can be made on any DC and will replicate to all others. However, FSMO roles (Task 5) handle specific operations that require a single authoritative DC.

### 🔧 Steps

**Check replication status via PowerShell:**
```powershell
# Show all replication partners and their status
Get-ADReplicationPartnerMetadata -Target SERVER-ADC -Scope Domain

# Check for any replication failures
Get-ADReplicationFailure -Scope Domain

# Force immediate replication from all partners
repadmin /syncall /AdeP
```

**Check replication via command line:**
```cmd
:: Show replication summary for all DCs
repadmin /replsummary

:: Show inbound replication status for a specific DC
repadmin /showrepl SERVER-ADC

:: Show replication queue (pending changes)
repadmin /queue SERVER-ADC
```

**Check via GUI:**
1. Open **Active Directory Sites and Services** (`dssite.msc`)
2. Expand **Sites** → **Default-First-Site-Name** → **Servers**
3. Expand **SERVER-ADC** → **NTDS Settings**
4. Right-click the replication connection → **Replicate Now**
5. A success dialog confirms replication completed

### ✅ Solution / Expected Result
`repadmin /replsummary` shows both servers with **0 failures** and recent successful replication times. All AD objects created on SERVER-PDC (users, groups, OUs) are visible on SERVER-ADC in ADUC. DNS zones are replicated and identical on both servers.

> **Replication Intervals:** By default, AD replication occurs within a site every **15 seconds** for urgent changes (password changes) and every **5 minutes** for routine changes. Between sites, replication is configured by schedule in AD Sites and Services.

---

## Task 5 – Identify Current FSMO Role Holders

### 📖 Explanation
Before transferring or seizing FSMO roles, you must first **identify which DC currently holds each role**. By default, when a forest is created, the **first DC holds all five FSMO roles**. Knowing the current holders is essential for planning transfers and for diagnosing problems when certain AD operations fail.

Understanding each FSMO role's function helps determine the impact of its unavailability:

| FSMO Role | Scope | Impact if Unavailable |
|-----------|-------|----------------------|
| **PDC Emulator** | Domain | Password changes fail; time sync issues; legacy auth breaks |
| **RID Master** | Domain | Cannot create new AD objects after RID pool is exhausted |
| **Infrastructure Master** | Domain | Cross-domain group membership updates stale |
| **Domain Naming Master** | Forest | Cannot add or remove domains from the forest |
| **Schema Master** | Forest | Cannot modify the AD schema (e.g., install Exchange, Lync) |

### 🔧 Steps

**Method 1: PowerShell (Recommended)**
```powershell
# Show all five FSMO role holders at once
Get-ADDomain | Select-Object PDCEmulator, RIDMaster, InfrastructureMaster
Get-ADForest | Select-Object DomainNamingMaster, SchemaMaster

# Or combined in one output
$domain = Get-ADDomain
$forest = Get-ADForest
[PSCustomObject]@{
    PDCEmulator         = $domain.PDCEmulator
    RIDMaster           = $domain.RIDMaster
    InfrastructureMaster= $domain.InfrastructureMaster
    DomainNamingMaster  = $forest.DomainNamingMaster
    SchemaMaster        = $forest.SchemaMaster
}
```

**Method 2: `netdom` command**
```cmd
netdom query fsmo
```

**Method 3: GUI — via Active Directory Users and Computers**
1. Open **ADUC** (`dsa.msc`) → right-click the domain → **Operations Masters…**
2. Three tabs show: **RID**, **PDC**, and **Infrastructure** role holders

**Method 4: GUI — via Active Directory Domains and Trusts**
1. Open `domain.msc` → right-click the root → **Operations Master…**
2. Shows the **Domain Naming Master** holder

**Method 5: GUI — via Schema Master**
1. Register the Schema snap-in: `regsvr32 schmmgmt.dll`
2. Open MMC → Add Snap-in → **Active Directory Schema**
3. Right-click **Active Directory Schema** → **Operations Master…**

### ✅ Solution / Expected Result
All five FSMO roles are currently held by **SERVER-PDC.company.local** — which is expected immediately after ADC promotion. The output of `netdom query fsmo` or the PowerShell commands clearly lists each role and its holder.

**Example output:**
```
PDCEmulator          : SERVER-PDC.company.local
RIDMaster            : SERVER-PDC.company.local
InfrastructureMaster : SERVER-PDC.company.local
DomainNamingMaster   : SERVER-PDC.company.local
SchemaMaster         : SERVER-PDC.company.local
```

---

## Task 6 – Transfer FSMO Roles (Planned Migration)

### 📖 Explanation
**FSMO Role Transfer** is a **graceful, planned** move of one or more roles from the current holder to another DC. Transfer is used when:
- Upgrading or decommissioning the current role holder
- Migrating from an older Windows Server version (e.g., 2012 → 2022)
- Load balancing: distributing roles across multiple DCs
- Performing maintenance (patching, hardware replacement) on the PDC

**Transfer requires both DCs to be online and replicating.** The current role holder participates in the transfer, ensuring a clean handoff with no data loss.

> **Best Practice for role distribution:**
> - Keep **PDC Emulator** and **RID Master** together on the most powerful DC
> - **Infrastructure Master** should NOT be on a Global Catalog server (unless all DCs are GC servers)
> - **Schema Master** and **Domain Naming Master** can remain on the forest root DC

### 🔧 Steps

**Method 1: PowerShell (Recommended)**
```powershell
# Transfer all five roles to SERVER-ADC
Move-ADDirectoryServerOperationMasterRole `
    -Identity "SERVER-ADC" `
    -OperationMasterRole PDCEmulator, RIDMaster, InfrastructureMaster, DomainNamingMaster, SchemaMaster `
    -Confirm:$false

# Transfer a single role (e.g., PDC Emulator only)
Move-ADDirectoryServerOperationMasterRole `
    -Identity "SERVER-ADC" `
    -OperationMasterRole PDCEmulator
```

**Method 2: GUI — via Active Directory Users and Computers (RID, PDC, Infrastructure)**
1. Open **ADUC** on SERVER-ADC (the target DC you want to receive the role)
2. Right-click the domain → **Change Domain Controller…** → select **SERVER-PDC** (current holder)
3. Right-click the domain again → **Operations Masters…**
4. On the **RID** tab → click **Change** → click **Yes** to confirm
5. Repeat for **PDC** and **Infrastructure** tabs
6. Click **Close**

**Method 3: GUI — via Active Directory Domains and Trusts (Domain Naming Master)**
1. Open `domain.msc` → right-click the root → **Change Active Directory Domain Controller…** → select SERVER-PDC
2. Right-click the root → **Operations Master…**
3. Click **Change** → **Yes** → **Close**

**Method 4: GUI — via Schema snap-in (Schema Master)**
1. Open MMC with Active Directory Schema snap-in
2. Right-click **Active Directory Schema** → **Change Active Directory Domain Controller…** → select SERVER-PDC
3. Right-click **Active Directory Schema** → **Operations Master…**
4. Click **Change** → **Yes** → **Close**

### ✅ Solution / Expected Result
After transfer, running `netdom query fsmo` from any DC shows all (or selected) roles now held by **SERVER-ADC.company.local**. The transfer is logged in the **Directory Services** event log (Event ID 1458 for successful transfer).

---

## Task 7 – Seize FSMO Roles (Emergency Takeover)

### 📖 Explanation
**FSMO Role Seizure** is a **forced, emergency** takeover of FSMO roles when the current role holder is **permanently offline, crashed, or unrecoverable**. Seizure is a last resort — it is used only when:

- The PDC has crashed and **cannot be recovered**
- The PDC is being decommissioned due to hardware failure
- The PDC is permanently removed from the network

> ⚠️ **CRITICAL WARNING:** Never seize roles if there is **any possibility** the original holder will come back online. If the original role holder returns to the network after a seizure, it will believe it still holds the roles, causing a **split-brain scenario** that can corrupt the AD database. If there is any chance of recovery, **do not seize** — wait and transfer instead.

**The rule: Seize only when the original DC will NEVER return to the domain.**

After seizure, the original role holder must be **permanently removed** from the domain and never reconnected.

### 🔧 Steps

**Method 1: PowerShell**
```powershell
# Seize all roles (run on the target ADC that will receive the roles)
Move-ADDirectoryServerOperationMasterRole `
    -Identity "SERVER-ADC" `
    -OperationMasterRole PDCEmulator, RIDMaster, InfrastructureMaster, DomainNamingMaster, SchemaMaster `
    -Force `
    -Confirm:$false
```

**Method 2: `ntdsutil` (Classic method — works on all Windows Server versions)**
```
# Open ntdsutil
ntdsutil

# Enter FSMO maintenance context
ntdsutil: roles

# Connect to the target DC (the one that will seize the roles)
fsmo maintenance: connections
server connections: connect to server SERVER-ADC
server connections: quit

# Seize each role one by one
fsmo maintenance: seize PDC
fsmo maintenance: seize RID master
fsmo maintenance: seize infrastructure master
fsmo maintenance: seize domain naming master
fsmo maintenance: seize schema master

# Exit
fsmo maintenance: quit
ntdsutil: quit
```

### ✅ Solution / Expected Result
After seizure, `netdom query fsmo` confirms all roles are now held by **SERVER-ADC.company.local**. The event log on SERVER-ADC shows Event ID 1458 for each seized role. The original PDC (SERVER-PDC) must be **permanently removed** from the domain — never reconnect it without first demoting it and cleaning up its metadata.

**Clean up metadata of the failed DC:**
```powershell
# Remove the dead DC's metadata from AD
Remove-ADDomainController -Identity "SERVER-PDC" -ForceRemoval -Confirm:$false

# Or via ntdsutil:
# ntdsutil → metadata cleanup → remove selected server
```

---

## Task 8 – Verify FSMO Role Assignment After Transfer/Seizure

### 📖 Explanation
After any FSMO role transfer or seizure, it is essential to **verify** the new assignments are correct and that the new role holders are functioning properly. Verification involves:
1. Confirming the correct DC holds each role
2. Testing that AD operations dependent on each role work correctly
3. Checking the event logs for any errors related to role ownership

### 🔧 Steps

**Step 1: Confirm role holders**
```powershell
# Quick verification of all five roles
netdom query fsmo

# Detailed PowerShell verification
Get-ADDomain | Select-Object PDCEmulator, RIDMaster, InfrastructureMaster
Get-ADForest | Select-Object DomainNamingMaster, SchemaMaster
```

**Step 2: Functional testing per role**

```powershell
# Test PDC Emulator: Try changing a user password
Set-ADAccountPassword -Identity "testuser" -Reset `
    -NewPassword (ConvertTo-SecureString "NewP@ss123!" -AsPlainText -Force)

# Test RID Master: Create a new AD object (uses RIDs)
New-ADUser -Name "TestRIDUser" -SamAccountName "testriduser" `
           -Path "CN=Users,DC=company,DC=local"

# Test replication is healthy after role changes
repadmin /replsummary
Get-ADReplicationFailure -Scope Domain
```

**Step 3: Check event logs**
```powershell
# Check Directory Services log for FSMO-related events
Get-WinEvent -LogName "Directory Service" -MaxEvents 50 |
    Where-Object { $_.Id -in @(1458, 1459, 1460) } |
    Select-Object TimeCreated, Id, Message
```

| Event ID | Meaning |
|----------|---------|
| 1458 | FSMO role transfer/seizure succeeded |
| 1459 | FSMO role transfer/seizure failed |
| 1460 | FSMO role holder conflict detected |

### ✅ Solution / Expected Result
All five FSMO roles are confirmed on the correct DC. Password changes work, new users can be created, and replication shows 0 failures. No Event ID 1459 or 1460 errors in the event log. The domain continues to function normally with the new role holder.

---

## 📝 Summary

| # | Task | Tool Used | Key Action | Outcome |
|---|------|-----------|------------|---------|
| 1 | Plan ADC deployment | Capacity planning | Assess need, provision server | Secondary server ready with static IP pointing to PDC DNS |
| 2 | Install AD DS role | Server Manager / PowerShell | Install AD-Domain-Services feature | Role installed, server ready for promotion |
| 3 | Promote to ADC | AD DS Config Wizard / PowerShell | Join as additional DC in company.local | SERVER-ADC is a full Domain Controller with replicated AD |
| 4 | Verify replication | `repadmin`, PowerShell | Check replication status and sync | 0 replication failures, all objects synchronized |
| 5 | Identify FSMO holders | `netdom query fsmo` / PowerShell | Query all five role holders | All 5 roles confirmed on SERVER-PDC |
| 6 | Transfer FSMO roles | PowerShell / GUI | Move roles to SERVER-ADC gracefully | All roles now on SERVER-ADC; both DCs were online |
| 7 | Seize FSMO roles | `ntdsutil` / PowerShell `-Force` | Force-take roles from downed PDC | Roles seized by SERVER-ADC; original PDC metadata cleaned up |
| 8 | Verify role assignment | `netdom`, event logs, functional tests | Confirm roles, test operations | All roles confirmed; domain operations healthy |

---

## 📚 FSMO Roles Reference

### Domain-Level Roles (Per Domain)

#### 1. PDC Emulator
- **Full name:** Primary Domain Controller Emulator
- **Scope:** Per domain
- **Responsibilities:**
  - Acts as the authoritative source for **password changes** — when a password is changed on any DC, that change is immediately replicated to the PDC Emulator before other DCs
  - Provides **time synchronization** for all DCs in the domain (which get time from it; clients get time from their authenticating DC)
  - Handles **account lockout** processing
  - Manages **Group Policy** updates and conflicts
  - Supports legacy Windows NT 4.0 clients (backward compatibility)
- **Impact if down:** Password changes may fail; users may experience login failures due to stale password cache; time drift issues

#### 2. RID Master
- **Full name:** Relative Identifier Master
- **Scope:** Per domain
- **Responsibilities:**
  - Allocates **pools of Relative IDs (RIDs)** to each DC (typically 500 RIDs per pool)
  - Every security principal (user, group, computer) has a unique **SID = Domain SID + RID**
  - Without unique RIDs, duplicate SIDs could exist, causing security vulnerabilities
- **Impact if down:** DCs eventually run out of their local RID pool and cannot create new AD objects (users, groups, computers)

#### 3. Infrastructure Master
- **Full name:** Infrastructure Master
- **Scope:** Per domain
- **Responsibilities:**
  - Updates **cross-domain group membership references** — when a user from Domain A is a member of a group in Domain B, the Infrastructure Master keeps this reference current
  - Compares its data with the **Global Catalog** to detect stale references
- **Impact if down:** Cross-domain group memberships may appear stale; phantom objects may not be cleaned up
- **Important rule:** Should NOT be placed on a Global Catalog server (unless all DCs are GC servers)

### Forest-Level Roles (Forest Root Domain Only)

#### 4. Domain Naming Master
- **Full name:** Domain Naming Master
- **Scope:** Per forest (one for entire forest)
- **Responsibilities:**
  - Controls the addition and removal of **domains within the forest**
  - Ensures **unique domain names** across the entire forest namespace
  - Must be online when running `dcpromo` to add or remove a domain
- **Impact if down:** Cannot add or remove domains from the forest

#### 5. Schema Master
- **Full name:** Schema Master
- **Scope:** Per forest (one for entire forest)
- **Responsibilities:**
  - Controls all **modifications to the Active Directory schema** — the blueprint defining all object types and attributes
  - Schema changes replicate to all DCs in the entire forest
  - Required when installing applications that extend the AD schema (Microsoft Exchange, Lync/Teams, System Center)
- **Impact if down:** Cannot modify the AD schema; cannot run `adprep /forestprep` for OS upgrades

---

## 🔑 Key Concepts

| Concept | Description |
|---------|-------------|
| **Additional Domain Controller (ADC)** | A secondary DC that replicates AD data, providing redundancy and load distribution |
| **Single Point of Failure (SPOF)** | A component whose failure causes the entire system to stop working — eliminated by adding an ADC |
| **Multi-master Replication** | AD changes can be written to any DC and replicate to all others — no read-only replicas |
| **FSMO (Flexible Single Master Operations)** | Five specialized roles where only one DC can be authoritative at a time |
| **Transfer** | Graceful, planned FSMO role migration — both DCs online, clean handoff |
| **Seizure** | Forced FSMO role takeover — original holder is permanently offline |
| **RID (Relative Identifier)** | The unique suffix of a Security Identifier (SID) — allocated by the RID Master |
| **SID (Security Identifier)** | Unique identifier for every AD security principal: Domain SID + RID |
| **Global Catalog (GC)** | A DC that stores a partial replica of all objects across all domains in the forest |
| **SYSVOL** | Shared folder on all DCs containing Group Policy templates, scripts, and logon scripts |
| **DSRM** | Directory Services Restore Mode — emergency boot mode for DC recovery |
| **ntdsutil** | Command-line tool for AD database management, FSMO seizure, and metadata cleanup |
| **repadmin** | Command-line tool for monitoring and troubleshooting AD replication |
| **Kerberos** | The default network authentication protocol managed by the DC (specifically the PDC Emulator) |

---

## ⚖️ Transfer vs Seizure Decision Guide

```
                    PDC is DOWN
                        │
                        ▼
           Can the PDC be recovered?
          /                         \
        YES                          NO
         │                            │
         ▼                            ▼
    Wait for recovery           Will PDC EVER
    then TRANSFER                return to network?
                                /              \
                              YES               NO
                               │                │
                               ▼                ▼
                     DO NOT SEIZE!          SEIZE roles
                     Wait and transfer      on ADC, then
                     when PDC is back       permanently
                                           remove PDC
                                           metadata
```

| Scenario | Action | Command |
|---------|--------|---------|
| PDC going for planned maintenance | Transfer | `Move-ADDirectoryServerOperationMasterRole` |
| Upgrading PDC OS | Transfer before upgrade | `Move-ADDirectoryServerOperationMasterRole` |
| PDC hardware failure (recoverable) | Wait, then transfer | Wait for recovery |
| PDC hardware failure (unrecoverable) | Seize | `Move-ADDirectoryServerOperationMasterRole -Force` |
| PDC crashed, will be rebuilt | Seize, clean metadata, rebuild | `ntdsutil` seize + metadata cleanup |

---

## 🛠️ Troubleshooting

| Problem | Likely Cause | Solution |
|---------|-------------|----------|
| Promotion fails with DNS error | Server-ADC DNS not pointing to PDC | Set Server-ADC's primary DNS to PDC's IP (192.168.1.1) |
| Replication showing failures | Network connectivity or firewall | Check ports: TCP/UDP 389, 636, 3268, 49152-65535 are open between DCs |
| `repadmin /replsummary` shows errors | Replication broken | Run `repadmin /syncall /AdeP` to force sync; check event logs |
| FSMO transfer fails | DCs not replicating | Fix replication first, then retry transfer |
| After seizure, old PDC comes back | Seizure was premature | Immediately disconnect old PDC from network; clean up its metadata |
| Users cannot log in after PDC failure | PDC Emulator down | Seize PDC Emulator role on ADC (Task 7) |
| Cannot create new users/groups | RID pool exhausted and RID Master down | Seize RID Master on ADC (Task 7) |
| Cannot add new domain to forest | Domain Naming Master down | Seize Domain Naming Master if unrecoverable |
| Schema extension fails (Exchange install) | Schema Master down | Seize Schema Master if unrecoverable |
| `netdom query fsmo` shows wrong server | Roles not transferred properly | Re-run transfer commands; verify with PowerShell `Get-ADDomain` |

---

## 📋 Quick Reference Commands

```powershell
# ── CHECK ──────────────────────────────────────────────
# Show all FSMO role holders
netdom query fsmo

# Detailed FSMO info
Get-ADDomain | Select PDCEmulator, RIDMaster, InfrastructureMaster
Get-ADForest | Select DomainNamingMaster, SchemaMaster

# Check replication health
repadmin /replsummary
Get-ADReplicationFailure -Scope Domain

# ── TRANSFER ───────────────────────────────────────────
# Transfer all roles to SERVER-ADC (graceful)
Move-ADDirectoryServerOperationMasterRole -Identity "SERVER-ADC" `
    -OperationMasterRole PDCEmulator,RIDMaster,InfrastructureMaster,`
    DomainNamingMaster,SchemaMaster -Confirm:$false

# ── SEIZE ──────────────────────────────────────────────
# Seize all roles on SERVER-ADC (emergency, PDC permanently gone)
Move-ADDirectoryServerOperationMasterRole -Identity "SERVER-ADC" `
    -OperationMasterRole PDCEmulator,RIDMaster,InfrastructureMaster,`
    DomainNamingMaster,SchemaMaster -Force -Confirm:$false

# ── CLEANUP ────────────────────────────────────────────
# Remove metadata of the failed DC
Remove-ADDomainController -Identity "SERVER-PDC" -ForceRemoval -Confirm:$false

# Force AD replication
repadmin /syncall /AdeP
```

---

*Lab based on Windows Server 2022 | Domain: company.local | Tools: PowerShell, Server Manager, ntdsutil, repadmin, netdom*
