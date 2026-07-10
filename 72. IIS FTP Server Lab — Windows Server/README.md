# 📁 IIS FTP Server Lab — Windows Server

> **Lab Overview:** This lab covers installing, configuring, and securing an FTP site on Windows Server using IIS (Internet Information Services). You will install the FTP Server role, create an FTP site with anonymous and authenticated access, configure NTFS permissions, set up AD users and groups for FTP authorization, implement user isolation so each user is restricted to their own home directory, and verify access from a client machine.

---

## 📋 Table of Contents

1. [Prerequisites](#prerequisites)
2. [Lab Topology](#lab-topology)
3. [Task 1 — Install FTP Server Role (IIS)](#task-1--install-ftp-server-role-iis)
4. [Task 2 — Add FTP Site (Site Information)](#task-2--add-ftp-site-site-information)
5. [Task 3 — FTP Binding and SSL Settings](#task-3--ftp-binding-and-ssl-settings)
6. [Task 4 — FTP Authentication & Authorization (Anonymous)](#task-4--ftp-authentication--authorization-anonymous)
7. [Task 5 — Disable IE Enhanced Security on Client](#task-5--disable-ie-enhanced-security-on-client)
8. [Task 6 — Access FTP Site from Client](#task-6--access-ftp-site-from-client)
9. [Task 7 — Create AD Users and Group for FTP](#task-7--create-ad-users-and-group-for-ftp)
10. [Task 8 — Disable NTFS Inheritance on FTP Folder](#task-8--disable-ntfs-inheritance-on-ftp-folder)
11. [Task 9 — Set NTFS Permissions for FTP Users Group](#task-9--set-ntfs-permissions-for-ftp-users-group)
12. [Task 10 — Full Permissions for CREATOR OWNER](#task-10--full-permissions-for-creator-owner)
13. [Task 11 — FTP Authentication & Authorization (Basic / AD Group)](#task-11--ftp-authentication--authorization-basic--ad-group)
14. [Task 12 — Authenticate to FTP as AD User](#task-12--authenticate-to-ftp-as-ad-user)
15. [Task 13 — FTP User Home Folders](#task-13--ftp-user-home-folders)
16. [Task 14 — Configure FTP User Isolation](#task-14--configure-ftp-user-isolation)
17. [Troubleshooting](#troubleshooting)
18. [Key Concepts Summary](#key-concepts-summary)

---

## Prerequisites

| Requirement | Details |
|---|---|
| Server OS | Windows Server 2016 / 2019 / 2022 |
| Server Name | PDC16.company.local |
| IIS Role | Web Server (IIS) must be installed |
| FTP Root Path | `C:\inetpub\ftproot` (anonymous) |
| Secured FTP Path | `E:\FTP` (AD-authenticated, isolated) |
| FTP Port | 21 (default) |
| Server IP | 192.168.1.2 |
| Domain | COMPANY |
| AD Users | Khaled Amr (`h.amr`), Mohamed Elsayed |
| AD Group | `FTP Users` (Security Group – Global) |

---

## Lab Topology

```
                    PDC16.company.local
                      192.168.1.2
                           │
                    ┌──────┴───────┐
                    │   IIS / FTP  │
                    │   Port 21    │
                    └──────┬───────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
    FTP Site: FTP1    FTP Site: FTP2   (Isolated)
    C:\inetpub\       E:\FTP\          E:\FTP\
    ftproot\          ├─ LocalUser\    ├─ LocalUser\
    (Anonymous)       │   ├─ Hany Folder       │   ├─ h.amr\
    Read only         │   └─ Mohamed Folder    │   └─ m.elsayed\
                      └─ (Auth: FTP Users AD group)

Client Machine ──────────────────────────────► ftp://192.168.1.2/
  (IE / File Explorer / CMD)
```

---

## Task 1 — Install FTP Server Role (IIS)

**Goal:** Install the **FTP Server** role service under Web Server (IIS) on PDC16 using the Add Roles and Features Wizard.

![Task 1 - Install FTP](images-ftp/task1-install-ftp.png)

### Steps:

1. Open **Server Manager** → click **Manage → Add Roles and Features**.

2. Proceed through the wizard:
   - **Installation Type:** Role-based or feature-based installation
   - **Server Selection:** PDC16.company.local
   - **Server Roles:** Expand **Web Server (IIS) → Web Server → FTP Server**
   - Check both:
     - ✅ **FTP Server**
     - ✅ **FTP Service**

3. On the **Confirmation** page, verify the installation list:
   ```
   Web Server (IIS)
     └── FTP Server
           └── FTP Service
   ```

4. Click **Install** and wait for completion.

> **💡 Note:** The FTP Server role is a sub-role of Web Server (IIS). If IIS is not yet installed, the wizard will automatically add it as a dependency.

### Role Hierarchy:
```
Web Server (IIS)
├── Web Server
│     ├── Common HTTP Features
│     └── ...
└── FTP Server         ← Install this
      └── FTP Service  ← Required sub-component
```

---

## Task 2 — Add FTP Site (Site Information)

**Goal:** Create a new FTP site named **FTP1** using the Add FTP Site Wizard in IIS Manager, pointing to the default FTP root directory.

![Task 2 - Add FTP Site](images-ftp/task2-add-ftp-serv.png)

### Steps:

1. Open **IIS Manager** (`inetmgr`) → expand the server node → right-click **Sites → Add FTP Site**.

2. On the **"Site Information"** page:

   | Field | Value |
   |---|---|
   | **FTP site name** | `FTP1` |
   | **Physical path** | `C:\inetpub\ftproot` |

3. Click **...** (Browse) to navigate to or confirm `C:\inetpub\ftproot`.

4. Click **Next**.

> **💡 Tip:** `C:\inetpub\ftproot` is the default IIS FTP directory and is created automatically when IIS is installed. You can use any folder path — for a secured, isolated FTP site, you will use `E:\FTP` in later tasks.

---

## Task 3 — FTP Binding and SSL Settings

**Goal:** Configure the FTP site to listen on all IP addresses, port 21, with no SSL (for this lab environment).

![Task 3 - FTP Binding and SSL](images-ftp/task3-ftpbinding-and-ssl.png)

### Steps:

1. On the **"Binding and SSL Settings"** page:

   | Setting | Value |
   |---|---|
   | **IP Address** | All Unassigned |
   | **Port** | `21` |
   | **Enable Virtual Host Names** | ☐ Unchecked |
   | **Start FTP site automatically** | ✅ Checked |
   | **SSL** | No SSL ✅ |
   | **SSL Certificate** | Not Selected |

2. Click **Next**.

### SSL Options Explained:

| SSL Option | Description | Use Case |
|---|---|---|
| **No SSL** | FTP traffic sent in plain text | Internal lab/testing only |
| **Allow SSL** | SSL is optional; clients can connect with or without it | Mixed environments |
| **Require SSL** | All connections must use FTPS (FTP over SSL) | Production / internet-facing |

> **⚠️ Security Warning:** In production environments, always use **Require SSL** (FTPS) or switch to **SFTP** (SSH-based). Plain FTP transmits credentials and data in clear text, making it vulnerable to interception.

---

## Task 4 — FTP Authentication & Authorization (Anonymous)

**Goal:** Configure the first FTP site to allow **anonymous access** with **Read-only** permissions — no username or password required.

![Task 4 - FTP Auth and Authorization](images-ftp/task4-ftp-authentication-and-authorization.png)

### Steps:

1. On the **"Authentication and Authorization Information"** page:

   **Authentication:**
   | Option | Value |
   |---|---|
   | Anonymous | ✅ Checked |
   | Basic | ☐ Unchecked |

   **Authorization:**
   | Option | Value |
   |---|---|
   | Allow access to | `Anonymous users` |
   | Permissions | ✅ Read / ☐ Write |

2. Click **Finish**.

### Authentication Types Compared:

| Type | Credentials Required | Password Encryption | Use Case |
|---|---|---|---|
| **Anonymous** | None (user: anonymous) | N/A | Public file downloads |
| **Basic** | Windows username + password | Plain text (use with SSL!) | Authenticated users |
| **AD Client Certificate** | Certificate | Encrypted | High-security environments |

> **💡 Anonymous FTP:** When clients connect anonymously, IIS uses the built-in `IUSR` account to access the file system. The NTFS permissions on `C:\inetpub\ftproot` must allow `IUSR` to read files.

---

## Task 5 — Disable IE Enhanced Security on Client

**Goal:** On the client machine, disable **Internet Explorer Enhanced Security Configuration (IE ESC)** so that IE can access the FTP site without security prompts blocking the connection.

![Task 5 - Disable IE ESC on Client](images-ftp/task5-disable-security-on-client.png)

### Steps:

1. On the **client machine**, open **Server Manager** (if Windows Server) → click the local server name in the left pane.

2. Find **IE Enhanced Security Configuration** → click **On** to open the dialog.

3. In the **"Internet Explorer Enhanced Security Configuration"** dialog:

   | Group | Setting |
   |---|---|
   | **Administrators** | ⦿ Off |
   | **Users** | ⦿ Off |

4. Click **OK**.

> **💡 Why disable IE ESC?** IE ESC blocks most websites and FTP connections by default on Windows Server. Turning it off allows normal browsing and FTP access in Internet Explorer/Edge for lab testing.

> **⚠️ Note:** This is a lab step. In production, keep IE ESC enabled and use proper FTP clients (FileZilla, WinSCP) instead.

---

## Task 6 — Access FTP Site from Client

**Goal:** Verify the FTP site is working by accessing it from a client machine using two methods: **File Explorer** and **Internet Explorer**.

![Task 6 - FTP Access via File Explorer](images-ftp/task6-ftp-access-method1.png)
![Task 6 - FTP Access via Internet Explorer](images-ftp/task6-ftp-access-method2.png)

### Method 1 — File Explorer:

1. Open **File Explorer** on the client.
2. In the address bar, type:
   ```
   ftp://192.168.1.2
   ```
3. Press **Enter** — the FTP root directory appears showing `index.html`.
4. Path shown: `The Internet → 192.168.1.2`

### Method 2 — Internet Explorer:

1. Open **Internet Explorer** on the client.
2. Navigate to:
   ```
   ftp://192.168.1.2/
   ```
3. The browser displays the FTP directory listing:
   ```
   FTP root at 192.168.1.2
   04/23/2026  04:30AM    186  index.html
   ```
4. To open in File Explorer from IE: press **Alt → View → Open FTP Site in File Explorer**.

### FTP Access Methods Compared:

| Client Method | Pros | Cons |
|---|---|---|
| **File Explorer** | Drag-and-drop, familiar UI | Limited to Windows |
| **Internet Explorer / Edge** | Quick browsing | Read-only in browser |
| **Command Prompt (ftp.exe)** | Scriptable | Text-only |
| **FileZilla / WinSCP** | Full-featured, SFTP support | Requires installation |
| **PowerShell** | Automatable | Requires scripting knowledge |

---

## Task 7 — Create AD Users and Group for FTP

**Goal:** In Active Directory, create a security group **FTP Users** and add domain users **Khaled Amr** and **Mohamed Elsayed** to it, so they can be authorized to access the secured FTP site.

![Task 7 - AD Users and Group](images-ftp/task7-ad-users-and-group.png)

### Steps:

1. Open **Active Directory Users and Computers** (`dsa.msc`).

2. Navigate to the appropriate OU (e.g., `company.local/Users`).

3. **Create the security group:**
   - Right-click → **New → Group**
   - Name: `FTP Users`
   - Group scope: **Global**
   - Group type: **Security**
   - Click **OK**

4. **Create users** (or verify they exist):

   | Username | Display Name | SAM Account |
   |---|---|---|
   | Khaled Amr | Khaled Amr | `h.amr` |
   | Mohamed Elsayed | Mohamed Elsayed | `m.elsayed` |

5. **Add users to the FTP Users group:**
   - Double-click `FTP Users` → **Members** tab → **Add**
   - Add both `Khaled Amr` and `Mohamed Elsayed`
   - Click **OK**

### AD Objects Created:

```
company.local
└── Users (OU)
      ├── FTP Users          ← Security Group – Global
      │     ├── Khaled Amr   ← Member
      │     └── Mohamed Elsayed  ← Member
      ├── Khaled Amr         ← User (SAM: h.amr)
      └── Mohamed Elsayed    ← User (SAM: m.elsayed)
```

---

## Task 8 — Disable NTFS Inheritance on FTP Folder

**Goal:** Break NTFS permission inheritance on `E:\FTP` so that you can assign precise, custom permissions for FTP users without inheriting overly broad permissions from the parent drive.

![Task 8 - Disable FTP Inheritance](images-ftp/task8-disable-ftp-inheritance.png)

### Steps:

1. Right-click `E:\FTP` → **Properties → Security tab → Advanced**.

2. In **"Advanced Security Settings for FTP"**, click **Disable inheritance**.

3. When prompted, choose:
   - **"Convert inherited permissions into explicit permissions"** — keeps existing permissions but makes them non-inherited (recommended)
   - *(Alternatively: "Remove all inherited permissions" — starts clean)*

4. Remove all permission entries **except** `Administrators (COMPANY\Administrators)` with **Full control** applied to **"This folder only"**.

5. Result — only one entry remains:

   | Type | Principal | Access | Inherited From | Applies To |
   |---|---|---|---|---|
   | Allow | Administrators (COMPANY\A...) | Full control | None | This folder only |

6. Click **OK** / **Apply**.

> **💡 Why disable inheritance?** The `E:\` volume likely has broad permissions (e.g., `Everyone` or `Users` with read access). Breaking inheritance gives you a clean slate to grant only the minimum permissions needed for FTP users — preventing unauthorized access or accidental writes.

---

## Task 9 — Set NTFS Permissions for FTP Users Group

**Goal:** Grant the **FTP Users** AD group specific NTFS permissions on `E:\FTP` — allowing them to list, read, and create subfolders, but **not** write files or delete content at the root level.

![Task 9 - NTFS Permissions for FTP Users](images-ftp/task9-ntfs-perm-on-ftp.png)

### Steps:

1. In **Advanced Security Settings for FTP**, click **Add → Select a principal**.

2. Type `COMPANY\FTP Users` → click **OK**.

3. Configure the **Permission Entry for FTP**:

   | Setting | Value |
   |---|---|
   | **Principal** | FTP Users (COMPANY\FTP Users) |
   | **Type** | Allow |
   | **Applies to** | This folder only |

4. Set the following **Advanced permissions** (✅ = checked):

   | Permission | Status | Purpose |
   |---|---|---|
   | Traverse folder / execute file | ✅ | Navigate into subfolders |
   | List folder / read data | ✅ | See folder contents |
   | Read attributes | ✅ | View file/folder attributes |
   | Read extended attributes | ✅ | Read extended metadata |
   | Create files / write data | ☐ | Denied at root level |
   | Create folders / append data | ✅ | Create their own home folder |
   | Write attributes | ☐ | Denied |
   | Write extended attributes | ☐ | Denied |
   | Delete subfolders and files | ☐ | Denied |
   | Delete | ☐ | Denied |
   | Read permissions | ✅ | View permission settings |
   | Change permissions | ☐ | Denied |
   | Take ownership | ☐ | Denied |

5. Click **OK**.

> **💡 Design Intent:** Users can browse the root `E:\FTP` folder and create their own home directory (via "Create folders / append data"), but cannot write files directly at the root or interfere with other users' folders. The `CREATOR OWNER` entry (Task 10) will grant each user full control over their own subfolder.

---

## Task 10 — Full Permissions for CREATOR OWNER

**Goal:** Grant **CREATOR OWNER** full control over `E:\FTP` and all subfolders and files, so that when a user creates their home folder, they automatically get full control of it.

![Task 10 - Full Permissions for CREATOR OWNER](images-ftp/task10-full-perm-for-creator-owner.png)

### Steps:

1. In **Advanced Security Settings for FTP**, click **Add → Select a principal**.

2. Type `CREATOR OWNER` → click **OK**.

3. Configure the **Permission Entry for FTP**:

   | Setting | Value |
   |---|---|
   | **Principal** | CREATOR OWNER |
   | **Type** | Allow |
   | **Applies to** | This folder, subfolders and files |

4. Check **all** Advanced permissions (Full control = all checked):
   - ✅ Full control
   - ✅ Traverse folder / execute file
   - ✅ List folder / read data
   - ✅ Read attributes
   - ✅ Read extended attributes
   - ✅ Create files / write data
   - ✅ Create folders / append data
   - ✅ Write attributes
   - ✅ Write extended attributes
   - ✅ Delete subfolders and files
   - ✅ Delete
   - ✅ Read permissions
   - ✅ Change permissions
   - ✅ Take ownership

5. Click **OK**.

### How CREATOR OWNER Works:

```
User h.amr connects to FTP and creates folder "Hany Folder" under E:\FTP\
         │
         ▼
h.amr becomes the CREATOR OWNER of "Hany Folder"
         │
         ▼
CREATOR OWNER ACE applies → h.amr gets Full Control of "Hany Folder"
         │
         ▼
h.amr can read, write, delete, rename inside "Hany Folder" ✅
h.amr CANNOT touch "Mohamed Folder" (owned by m.elsayed) ❌
```

### Final NTFS Permission Summary for `E:\FTP`:

| Principal | Access | Applies To |
|---|---|---|
| Administrators | Full control | This folder only |
| FTP Users | Traverse + List + Read + Create folders | This folder only |
| CREATOR OWNER | Full control | This folder, subfolders and files |

---

## Task 11 — FTP Authentication & Authorization (Basic / AD Group)

**Goal:** Create a second, **secured FTP site** (or reconfigure FTP1) using **Basic Authentication** restricted to the **FTP Users** AD group with both Read and Write permissions.

![Task 11 - FTP Authentication and Authorization](images-ftp/task11-ftp-authentication-and-authorization.png)

### Steps:

1. In IIS Manager, add a new FTP site (or edit the existing one) pointing to `E:\FTP`.

2. On the **"Authentication and Authorization Information"** page:

   **Authentication:**
   | Option | Value |
   |---|---|
   | Anonymous | ☐ Unchecked |
   | Basic | ✅ Checked |

   **Authorization:**
   | Option | Value |
   |---|---|
   | Allow access to | `Specified roles or user groups` |
   | Group name | `FTP Users` |
   | Permissions | ✅ Read + ✅ Write |

3. Click **Finish**.

### Anonymous vs Basic — Side by Side:

| Aspect | Anonymous (Task 4) | Basic Auth (Task 11) |
|---|---|---|
| Credentials needed | None | Domain username + password |
| File system identity | IUSR | The actual user account |
| NTFS permissions apply | IUSR's permissions | User's own permissions |
| Who can access | Everyone | Only FTP Users group members |
| Use case | Public downloads | Private user file storage |

---

## Task 12 — Authenticate to FTP as AD User

**Goal:** Log in to the FTP server from the client machine using domain credentials (`h.amr`) to verify Basic Authentication is working.

![Task 12 - FTP Authentication](images-ftp/task12-ftp-auth.png)

### Steps:

1. On the client, open **File Explorer** → navigate to `ftp://192.168.1.2`.

2. When prompted with the **"Log On As"** dialog:

   | Field | Value |
   |---|---|
   | **FTP server** | 192.168.1.2 |
   | **User name** | `h.amr` |
   | **Password** | (user's domain password) |
   | **Log on anonymously** | ☐ Unchecked |

3. Click **Log on**.

4. After successful authentication, the FTP root (`E:\FTP`) appears showing:
   - `Hany Folder` (if previously created by h.amr)
   - `Mohamed Folder` (if previously created by m.elsayed)

5. User `h.amr` can interact with `Hany Folder` but gets **"Access is denied" (550)** if attempting to delete or modify `Mohamed Folder` — proving that NTFS permissions and CREATOR OWNER are working correctly.

> **💡 Username Format:** For domain accounts, use either `h.amr` (SAM account name) or `COMPANY\h.amr`. Both work with Basic FTP authentication against Active Directory.

---

## Task 13 — FTP User Home Folders

**Goal:** Create per-user home directories under `E:\FTP\LocalUser\` that match each user's username, enabling FTP User Isolation to lock users to their own folder.

![Task 13 - FTP Folders](images-ftp/task13-ftp-folder.png)

### Steps:

1. On the FTP server, create the folder structure manually:

   ```
   E:\FTP\
   └── LocalUser\
         ├── Hany Folder\    ← for user h.amr (or use exact username)
         └── Mohamed Folder\ ← for user m.elsayed
   ```

   Or using PowerShell:
   ```powershell
   New-Item -Path "E:\FTP\LocalUser\h.amr" -ItemType Directory
   New-Item -Path "E:\FTP\LocalUser\m.elsayed" -ItemType Directory
   ```

2. For **domain user isolation**, the folder structure must be:
   ```
   E:\FTP\
   └── <DomainName>\       (e.g., COMPANY)
         ├── h.amr\         ← exact SAM account name
         └── m.elsayed\     ← exact SAM account name
   ```

3. Each user's home folder should have the CREATOR OWNER / user ACE applied (from Task 10).

### User Isolation Folder Naming Rules:

| User Type | Required Folder Path |
|---|---|
| Local user `john` | `E:\FTP\LocalUser\john\` |
| Domain user `COMPANY\h.amr` | `E:\FTP\COMPANY\h.amr\` |
| Anonymous user | `E:\FTP\LocalUser\Public\` |

> **⚠️ Important:** Folder names must match the username **exactly** (case-insensitive on Windows). If the folder doesn't exist, the user gets an error on login when isolation is enabled.

---

## Task 14 — Configure FTP User Isolation

**Goal:** Enable **FTP User Isolation** so that each authenticated user is locked into their own home directory and cannot browse to other users' folders.

![Task 14 - FTP User Isolation](images-ftp/task14-ftp-isolation.png)

### Steps:

1. In **IIS Manager**, select the FTP site → double-click **FTP User Isolation**.

2. Choose the isolation mode:

   | Mode | Description |
   |---|---|
   | FTP root directory | All users start at the FTP root; no isolation |
   | User name directory | Users start in a folder matching their username; no restriction |
   | User name directory (disable global virtual directories) | Users isolated to their named folder; global vdirs disabled |
   | **User name physical directory (enable global virtual directories)** ✅ | Users isolated to physical folder; global virtual directories still accessible |
   | FTP home directory configured in Active Directory | Home dir pulled from AD user attribute |
   | Custom | Advanced scripted isolation |

3. Select **"User name physical directory (enable global virtual directories)"** ✅

4. Click **Apply** in the Actions pane.

### How Isolation Works:

```
Before Isolation:
  h.amr logs in → sees ALL of E:\FTP\ including Mohamed's folder ❌

After Isolation (User name physical directory):
  h.amr logs in → automatically landed in E:\FTP\COMPANY\h.amr\
               → cannot navigate to E:\FTP\COMPANY\m.elsayed\ ✅
               → sees ONLY his own files ✅

  m.elsayed logs in → automatically landed in E:\FTP\COMPANY\m.elsayed\
                    → cannot see h.amr's folder ✅
```

> **💡 Global Virtual Directories:** Enabling global virtual directories while using isolation allows you to create shared virtual directories (e.g., a public `downloads` folder) that all users can access, while still being isolated from each other's home folders.

---

## Troubleshooting

| Problem | Possible Cause | Solution |
|---|---|---|
| FTP site not accessible from client | Firewall blocking port 21 | Add inbound rule: TCP port 21 on server firewall |
| Passive FTP fails | Firewall blocking passive ports (1024–65535) | Open passive port range or configure specific range in IIS |
| "530 User cannot log in" | Basic auth failed or user not in FTP Users group | Check AD group membership; verify password |
| "550 Access is denied" | NTFS permissions too restrictive | Check E:\FTP NTFS ACLs for FTP Users group |
| User can see other users' folders | Isolation not enabled or wrong folder structure | Enable isolation; verify folder naming matches SAM account |
| Anonymous access still works after switching to Basic | Old anonymous auth rule still active | IIS Manager → FTP Authentication → Disable Anonymous |
| "503 Bad sequence of commands" | SSL required but client connecting without SSL | Match SSL setting on client and server |
| User home folder not found | Folder name doesn't match SAM account exactly | Rename folder to match SAM account name exactly |

### Useful Commands

```powershell
# Check FTP service status
Get-Service FTPSVC

# Restart FTP service
Restart-Service FTPSVC

# List all IIS FTP sites
Get-WebSite | Where-Object { $_.serverAutoStart }

# Test FTP connection from command line
ftp 192.168.1.2

# Allow FTP through Windows Firewall
netsh advfirewall firewall add rule name="FTP" protocol=TCP dir=in localport=21 action=allow

# Allow FTP passive mode port range
netsh advfirewall firewall add rule name="FTP Passive" protocol=TCP dir=in localport=1024-65535 action=allow

# Create FTP user home folders for domain users
New-Item -Path "E:\FTP\COMPANY\h.amr" -ItemType Directory -Force
New-Item -Path "E:\FTP\COMPANY\m.elsayed" -ItemType Directory -Force

# Check NTFS permissions on FTP folder
Get-Acl "E:\FTP" | Format-List

# Add AD group to FTP folder with read permission
$acl = Get-Acl "E:\FTP"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("COMPANY\FTP Users","ReadAndExecute","ContainerInherit","InheritOnly","Allow")
$acl.AddAccessRule($rule)
Set-Acl "E:\FTP" $acl
```

---

## Key Concepts Summary

| Term | Definition |
|---|---|
| **FTP** | File Transfer Protocol — TCP port 21 for control, ports 20/passive range for data |
| **FTPS** | FTP over SSL/TLS — encrypts credentials and data in transit |
| **Anonymous FTP** | FTP access without credentials; uses IUSR account for NTFS permissions |
| **Basic Authentication** | FTP login with Windows/domain username and password (plain text unless SSL) |
| **FTP User Isolation** | Restricts each user to their own home directory — cannot browse other users' folders |
| **CREATOR OWNER** | Special Windows identity; ACE automatically applies to the creator of a file/folder |
| **NTFS Inheritance** | Child objects inherit permissions from parent by default; can be disabled |
| **IE ESC** | Internet Explorer Enhanced Security Configuration — extra security layer on Windows Server |
| **Passive FTP** | Client initiates both control and data connections; firewall-friendly |
| **Active FTP** | Server initiates data connection back to client; often blocked by firewalls |
| **LocalUser** | Subfolder under FTP root used for local account isolation mapping |
| **IIS Manager** | GUI tool (`inetmgr`) for managing IIS websites, FTP sites, app pools |

---

## Full Lab Flow Diagram

```
[Task 1]  Install IIS → FTP Server + FTP Service role
          │
[Task 2]  Add FTP Site: FTP1 → C:\inetpub\ftproot
          │
[Task 3]  Binding: All IPs, Port 21, No SSL, Auto-start
          │
[Task 4]  Auth: Anonymous only | Authz: Anonymous users | Perm: Read
          │
[Task 5]  Client: Disable IE Enhanced Security (Admin + Users → Off)
          │
[Task 6]  Client: Access ftp://192.168.1.2 via File Explorer + IE ✅
          │
[Task 7]  AD: Create group "FTP Users" + add Khaled Amr, Mohamed Elsayed
          │
[Task 8]  E:\FTP NTFS: Disable inheritance → keep Admins Full Control only
          │
[Task 9]  E:\FTP NTFS: Add FTP Users → Traverse + List + Read + CreateFolders
          │                             (This folder only)
[Task 10] E:\FTP NTFS: Add CREATOR OWNER → Full Control
          │                                (This folder, subfolders & files)
[Task 11] FTP Site: Auth: Basic | Authz: FTP Users group | Perm: Read+Write
          │
[Task 12] Client: Login as h.amr → sees Hany Folder + Mohamed Folder
          │         h.amr cannot delete Mohamed Folder (550 Access Denied) ✅
[Task 13] Server: Create E:\FTP\COMPANY\h.amr\ and E:\FTP\COMPANY\m.elsayed\
          │
[Task 14] IIS: FTP User Isolation → User name physical directory ✅
          └── h.amr logs in → lands in h.amr\ folder only
          └── m.elsayed logs in → lands in m.elsayed\ folder only ✅
```

---

*Lab Environment: Windows Server 2016/2019 | IIS FTP Server | PDC16.company.local | COMPANY domain*
