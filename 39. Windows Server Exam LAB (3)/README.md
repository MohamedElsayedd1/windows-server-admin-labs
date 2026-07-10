# 🖥️ Windows Server Revision LAB 3 — Full Solution Guide

**Domain:** `rev.local`  
**Server Name:** `PDC` | **Server IP:** `192.168.1.2`  
**Client Machine:** `HRPC01` (Windows 10)

---

## 📋 Table of Contents

1. [Create the Domain Environment](#1-create-the-domain-environment)
2. [Create OUs, Users & Groups](#2-create-ous-users--groups)
3. [Join Windows 10 Client](#3-join-windows-10-client)
4. [Add IT-Group as Local Administrator via GPO](#4-add-it-group-as-local-administrator-via-gpo)
5. [Remove Properties from This PC Context Menu](#5-remove-properties-from-this-pc-context-menu)
6. [Disable Command Line & Control Panel (HR & Sales)](#6-disable-command-line--control-panel-hr--sales)
7. [Disable External Storage for HR (Exclude IT-Group)](#7-disable-external-storage-for-hr-exclude-it-group)
8. [Configure Password Policy](#8-configure-password-policy)
9. [Configure Account Lockout Policy](#9-configure-account-lockout-policy)
10. [Create HR Desktop URL Shortcut](#10-create-hr-desktop-url-shortcut)
11. [Configure DHCP Scope with Exclusion & Reservation](#11-configure-dhcp-scope-with-exclusion--reservation)
12. [DNS Round-Robin Load Balancing](#12-dns-round-robin-load-balancing)
13. [Create Public Mapped Drive with Disk Quota](#13-create-public-mapped-drive-with-disk-quota)
14. [Create HR Shared Folder with Restricted Permissions](#14-create-hr-shared-folder-with-restricted-permissions)
15. [Map HR Network Drive via GPO](#15-map-hr-network-drive-via-gpo)
16. [Block Media and Executable Files via FSRM](#16-block-media-and-executable-files-via-fsrm)

---

## 1. Create the Domain Environment

1. Open **Server Manager → Add Roles and Features**
2. Install **Active Directory Domain Services (AD DS)**
3. Click **Promote this server to a domain controller**
4. Select **Add a new forest** → Root domain name: `rev.local`
5. Set DSRM password → complete wizard → server reboots
6. After reboot, rename the computer:
   - Right-click **This PC → Properties → Change settings → Change**
   - Computer name: `PDC`
7. Set static IP:
   - Network Adapter → IPv4 → IP: `192.168.1.2`, Subnet: `255.255.255.0`
   - Preferred DNS: `192.168.1.2` (point to itself)

---

## 2. Create OUs, Users & Groups

### Create Organizational Units

Open **Active Directory Users and Computers (ADUC)** → right-click `rev.local` → **New → Organizational Unit**

Create: `HR`, `Sales`, `IT`

### Create Users

Right-click each OU → **New → User** — create at least one user per department.

| OU | Example Users |
|---|---|
| HR | hr.user1, hr.user2 |
| Sales | sales.user1, sales.user2 |
| IT | it.user1, it.user2 |

### Create Groups and Add Members

Right-click each OU → **New → Group** → Scope: **Global**, Type: **Security**

| Group | OU | Members |
|---|---|---|
| HR-Group | HR | All HR users |
| Sales-Group | Sales | All Sales users |
| IT-Group | IT | All IT users |

To add members: double-click group → **Members tab → Add**

---

## 3. Join Windows 10 Client

On **HRPC01**:

1. Set static or DHCP IP, DNS pointing to `192.168.1.2`
2. Right-click **This PC → Properties → Change settings → Change**
3. Select **Domain** → enter `rev.local`
4. Enter domain admin credentials → restart

---

## 4. Add IT-Group as Local Administrator via GPO

**GPO path:** `Computer Configuration → Preferences → Control Panel Settings → Local Users and Groups`

1. Open **Group Policy Management** → right-click domain root → **Create a GPO** → name: `IT-LocalAdmin`
2. Edit → navigate to path above → right-click → **New → Local Group**
3. Configure:
   - Action: **Update**
   - Group name: **Administrators (built-in)**
   - Members → Add: `rev\IT-Group`
4. Link GPO to domain root (applies to all computers)
5. Run `gpupdate /force` on client

---

## 5. Remove Properties from This PC Context Menu

**GPO path:** `User Configuration → Policies → Administrative Templates → Desktop`

1. Create GPO → name: `Remove-ThisPC-Properties` → link to domain root
2. Enable **Remove Properties from the Computer icon context menu**

---

## 6. Disable Command Line & Control Panel (HR & Sales)

### Disable Command Prompt

**GPO path:** `User Configuration → Policies → Administrative Templates → System`

- Enable **Prevent access to the command prompt**
- Set **Disable the command prompt script processing also** → **Yes**

### Disable Control Panel

**GPO path:** `User Configuration → Policies → Administrative Templates → Control Panel`

- Enable **Prohibit access to Control Panel and PC Settings**

### Apply per OU

1. Create GPO → name: `Disable-CMD-ControlPanel-HR` → link to `HR` OU → configure both settings
2. Create GPO → name: `Disable-CMD-ControlPanel-Sales` → link to `Sales` OU → same settings

---

## 7. Disable External Storage for HR (Exclude IT-Group)

**GPO path:** `Computer Configuration → Policies → Administrative Templates → System → Removable Storage Access`

1. Create GPO → name: `Disable-ExternalStorage-HR` → link to `HR` OU
2. Enable **All Removable Storage classes: Deny all access**
3. To **exclude IT-Group**:
   - Select GPO → **Delegation tab → Advanced**
   - Add `IT-Group` → check **Deny: Apply Group Policy**
   - IT members will not have this restriction applied

---

## 8. Configure Password Policy

**GPO path:** `Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Password Policy`

> Must be configured in **Default Domain Policy** to apply domain-wide.

| Setting | Value |
|---|---|
| Maximum password age | 60 days |
| Minimum password length | 6 characters |
| Password must meet complexity requirements | **Enabled** |
| Enforce password history | 3 passwords remembered |
| Minimum password age | 1 day (prevents immediate re-use) |

---

## 9. Configure Account Lockout Policy

**GPO path:** `Computer Configuration → Policies → Windows Settings → Security Settings → Account Policies → Account Lockout Policy`

Edit **Default Domain Policy** → Account Lockout Policy:

| Setting | Value |
|---|---|
| Account lockout threshold | 5 invalid logon attempts |
| Account lockout duration | 30 minutes |
| Reset account lockout counter after | 30 minutes |

---

## 10. Create HR Desktop URL Shortcut

**GPO path:** `User Configuration → Preferences → Windows Settings → Shortcuts`

1. Create GPO → name: `HR-Desktop-Shortcut` → link to `HR` OU
2. Edit → navigate to path above → right-click → **New → Shortcut**
3. Configure:
   - Action: **Create**
   - Name: `HR App`
   - Target type: **URL**
   - Location: **Desktop**
   - Target URL: `http://hrapp.rev.local`

---

## 11. Configure DHCP Scope with Exclusion & Reservation

### Install & Authorize DHCP

1. **Server Manager → Add Roles and Features** → install **DHCP Server**
2. Run **DHCP Post-Install Configuration Wizard** → authorize with `rev\Administrator`

### Create Scope

In **DHCP Manager → IPv4 → New Scope**:

| Setting | Value |
|---|---|
| Scope name | Scope_Rev |
| Start IP | 192.168.1.40 |
| End IP | 192.168.1.230 |
| Subnet mask | 255.255.255.0 (/24) |
| Exclusion range | 192.168.1.80 – 192.168.1.85 |
| Lease duration | 8 days |
| Default gateway | 192.168.1.1 |
| DNS servers | 192.168.1.2 |
| Domain name | rev.local |

### Create IP Reservation for HRPC01

1. Get MAC address of HRPC01: run `ipconfig /all` on the client → note **Physical Address**
2. In DHCP Manager → expand Scope → **Reservations** → right-click → **New Reservation**
3. Configure:
   - Reservation name: `HRPC01`
   - IP address: `192.168.1.200`
   - MAC address: (MAC of HRPC01, format `XX-XX-XX-XX-XX-XX`)
   - Supported types: **Both**

---

## 12. DNS Round-Robin Load Balancing

DNS round-robin distributes traffic across two servers by creating two A records for the same hostname pointing to different IPs.

1. Open **DNS Manager** → expand `rev.local` forward lookup zone
2. Right-click → **New Host (A or AAAA)**:
   - Name: `www`
   - IP address: `192.168.1.8`
   - Click **Add Host**
3. Right-click again → **New Host (A or AAAA)**:
   - Name: `www`
   - IP address: `192.168.1.9`
   - Click **Add Host**

Both records now exist for `www.rev.local`. DNS will alternate between `192.168.1.8` and `192.168.1.9` for each query — distributing the load across both servers.

**Verify:**
```cmd
nslookup www.rev.local
```
The response should show both IP addresses alternating on repeated queries.

---

## 13. Create Public Mapped Drive with Disk Quota

### Create & Share the Folder

1. Create folder `E:\Public` on the server
2. Right-click → **Properties → Sharing → Advanced Sharing**
   - Share name: `Public`
   - Permissions → add **Domain Users** → grant **Full Control, Change, Read**
3. **Security tab (NTFS)** → add **Domain Users** → grant **Modify** (includes create, edit, delete)

### Apply Disk Quota

1. Open **File Server Resource Manager → Quota Management → Quotas → Create Quota**
2. Quota path: `E:\Public`
3. Define custom properties → Hard limit: **2 GB**
4. Click **Create**

### Map Drive via GPO for All Users

**GPO path:** `User Configuration → Preferences → Windows Settings → Drive Maps`

1. Create GPO → name: `Map-Public-Drive` → link to domain root
2. Right-click → **New → Mapped Drive**
3. Configure:
   - Action: **Create**
   - Location: `\\PDC\Public`
   - Drive letter: (choose available letter, e.g., `P:`)
   - Label as: `Public`
   - Reconnect: checked

---

## 14. Create HR Shared Folder with Restricted Permissions

HR-Group needs **Create and Edit** but **NOT Delete**.

### Create & Share

1. Create folder `E:\HR` on the server
2. Share as `HR` → give **Domain Users Full Control** at share level

### NTFS Permissions (no delete)

1. Right-click `E:\HR` → **Properties → Security → Advanced → Disable inheritance → Convert**
2. Remove all non-admin entries
3. Add `HR-Group`:
   - Click **Advanced → Add → Select principal: HR-Group**
   - Type: **Allow**
   - Check the following permissions individually:
     - ✅ List folder / read data
     - ✅ Read attributes
     - ✅ Create files / write data
     - ✅ Create folders / append data
     - ✅ Write attributes
     - ❌ Delete subfolders and files — **leave unchecked**
     - ❌ Delete — **leave unchecked**
4. Apply → OK

> This grants HR-Group the ability to create and edit files but denies the ability to delete them.

---

## 15. Map HR Network Drive via GPO

**GPO path:** `User Configuration → Preferences → Windows Settings → Drive Maps`

1. Create GPO → name: `Map-HR-Drive` → link to `HR` OU
2. Right-click → **New → Mapped Drive**
3. Configure:
   - Action: **Create**
   - Location: `\\PDC\HR`
   - Drive letter: **H:**
   - Label as: `HR`
   - Reconnect: checked

HR users will automatically have `H:` mapped to `\\PDC\HR` at logon.

---

## 16. Block Media and Executable Files via FSRM

### Install FSRM

**Server Manager → Add Roles and Features → File and Storage Services → File Server Resource Manager**

### Create File Screen on Public and HR Shares

1. Open **File Server Resource Manager → File Screening Management → File Screens**
2. Right-click → **Create File Screen**
3. File screen path: `E:\Public`
4. Select **Define custom file screen properties → Active screening**
5. File groups to block:
   - ✅ Audio and Video Files
   - ✅ Executable Files
6. Click **Create**
7. Repeat for `E:\HR` folder

### Blocked File Extensions (examples)

| File Group | Extensions Blocked |
|---|---|
| Audio and Video Files | `.mp3`, `.mp4`, `.avi`, `.mkv`, `.wav`, `.flac` |
| Executable Files | `.exe`, `.bat`, `.cmd`, `.msi`, `.com`, `.scr` |

> **Active screening** immediately denies saving blocked file types. Users attempting to upload media or executables will receive an **Access Denied** error.

---

## 🔑 Quick Reference — GPO Summary

| GPO Name | Linked To | Config Type |
|---|---|---|
| IT-LocalAdmin | Domain root | Computer Preferences |
| Remove-ThisPC-Properties | Domain root | User Config |
| Disable-CMD-ControlPanel-HR | HR OU | User Config |
| Disable-CMD-ControlPanel-Sales | Sales OU | User Config |
| Disable-ExternalStorage-HR | HR OU | Computer Config |
| HR-Desktop-Shortcut | HR OU | User Preferences |
| Map-Public-Drive | Domain root | User Preferences |
| Map-HR-Drive | HR OU | User Preferences |
| Default Domain Policy | Domain root | Password + Lockout |

---

## 📊 Lab Configuration Summary

| Setting | Value |
|---|---|
| Domain | rev.local |
| Server | PDC — 192.168.1.2 |
| Client | HRPC01 |
| DHCP Scope | 192.168.1.40 – 192.168.1.230 /24 |
| DHCP Exclusion | 192.168.1.80 – 192.168.1.85 |
| DHCP Reservation | 192.168.1.200 → HRPC01 |
| DNS Load Balance | www.rev.local → 192.168.1.8 + 192.168.1.9 |
| Public Share | \\PDC\Public — 2 GB quota, all users R/W/D |
| HR Share | \\PDC\HR — HR-Group create/edit, no delete |
| HR Drive Letter | H: |
| Password Max Age | 60 days |
| Password Min Length | 6 chars + complexity |
| Password History | Last 3 |
| Lockout Threshold | 5 attempts |
| Lockout Duration | 30 minutes |

---

## 🛠️ Requirements

- Windows Server 2016 / 2019 (PDC)
- Windows 10 client (HRPC01)
- Both on subnet `192.168.1.0/24`
- File Server Resource Manager (FSRM) role
- DHCP Server role
- Administrator / Domain Admin credentials

---

## 👨‍💻 Author

> Revision lab solution prepared for Windows Server administration coursework.  
> Domain: `rev.local` — Server: `PDC`
