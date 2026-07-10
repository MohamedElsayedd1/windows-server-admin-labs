# 👥 Bulk User Creation in Active Directory Using CSV + PowerShell + AI

> A practical lab guide for importing multiple Active Directory users at once using a CSV file, an AI-generated PowerShell script, and Microsoft Copilot — no manual user-by-user creation needed.

![Windows Server](https://img.shields.io/badge/Windows%20Server-2019-0078D4?style=flat-square&logo=windows&logoColor=white)
![PowerShell](https://img.shields.io/badge/PowerShell-Automation-5391FE?style=flat-square&logo=powershell&logoColor=white)
![AI](https://img.shields.io/badge/AI-Microsoft%20Copilot-00B2FF?style=flat-square&logo=microsoft&logoColor=white)
![Course](https://img.shields.io/badge/Session-10-blueviolet?style=flat-square)
![Level](https://img.shields.io/badge/Level-Intermediate-orange?style=flat-square)

---

## 📖 Overview

This is **Session 10** of the Windows Server 2019 course. Instead of creating user accounts one by one in ADUC, this session demonstrates how to bulk-import dozens of users into Active Directory in **under 30 seconds** using:

- A **CSV file** containing employee data
- A **PowerShell script** to read the CSV and create AD users
- **Microsoft Copilot (or ChatGPT)** to generate both the CSV template and the script automatically

> 📌 **Pre-requisite:** A working Domain Controller with AD DS installed, the target OU already created (e.g., `CRM`), and PowerShell access with Domain Admin privileges.

---

## 🎯 What This Lab Covers

| Topic | Description |
|---|---|
| Why bulk import | The problem with manual one-by-one user creation |
| Using AI to generate scripts | Prompting Microsoft Copilot for CSV + PowerShell output |
| CSV file structure | Required columns and format |
| PowerShell script | How the script reads the CSV and creates AD users |
| Running the script | PowerShell ISE as Administrator |
| Verifying results | Checking ADUC after the import |

---

## 🤖 Step 1 — Use AI to Generate the CSV and Script

Instead of writing the PowerShell script manually, use **Microsoft Copilot** (or ChatGPT) to generate everything instantly.

### Example Prompt (English)

```
Create a CSV file template for bulk importing users into Active Directory.
The CSV should include columns: FirstName, LastName, SamAccountName,
UserPrincipalName, OU, and Password.
Use the domain dc.local and OU named CRM.
Also generate a PowerShell script that reads this CSV
and creates the users in Active Directory.
```

> 💡 You can write the prompt in **Arabic or any language** — AI tools like Copilot and ChatGPT understand and respond correctly regardless of input language. This removes the language barrier for non-English speakers.

### What AI Returns

1. A ready-to-use **CSV template** with sample employee data
2. A complete **PowerShell script** that reads the CSV and creates users

Both can be customized with real employee data before running.

---

## 📄 Step 2 — Prepare the CSV File

### CSV Structure

The CSV file must contain these columns:

| Column | Description | Example |
|---|---|---|
| `FirstName` | Employee first name | Alice |
| `LastName` | Employee last name | Baker |
| `SamAccountName` | Windows logon name (pre-Windows 2000) | abaker |
| `UserPrincipalName` | Full UPN login format | abaker@dc.local |
| `OU` | Distinguished Name path of target OU | `OU=CRM,DC=DC,DC=local` |
| `Password` | Initial password (must meet complexity) | Welcome2024! |

### Sample CSV Content (`users.csv`)

This matches the data shown in the lab screenshot:

```csv
FirstName,LastName,SamAccountName,UserPrincipalName,OU,Password
Alice,Baker,abaker,abaker@dc.local,"OU=CRM,DC=DC,DC=local",Welcome2024!
Charlie,Davis,cdavis,cdavis@dc.local,"OU=CRM,DC=DC,DC=local",Pass9876#
Edward,Fisher,efisher,efisher@dc.local,"OU=CRM,DC=DC,DC=local",Secure001!
Grace,Hill,ghill,ghill@dc.local,"OU=CRM,DC=DC,DC=local",Summer2024@
Isaac,Jones,ijones,ijones@dc.local,"OU=CRM,DC=DC,DC=local",BlueSky456!
```

### Creating the CSV File

```
1. Copy the CSV content from the AI output
2. Open Notepad
3. Paste the content
4. File → Save As → name it "users.csv"
5. Change "Save as type" to "All Files"
6. Ensure filename ends with .csv
7. Click Save
```

### Transferring to the Server

```
Copy users.csv to the AD server
→ Place it at C:\users.csv  (or any accessible path)
```

---

## 💻 Step 3 — The PowerShell Script

### Full Script

```powershell
# Import the Active Directory module
Import-Module ActiveDirectory

# Define the path to your CSV file
$csvFile = "C:\users.csv"

# Import users from CSV
$users = Import-Csv -Path $csvFile

foreach ($user in $users) {
    # Convert the password from the CSV into a secure string
    $securePassword = ConvertTo-SecureString $user.Password -AsPlainText -Force

    # Check if user already exists
    if (!(Get-ADUser -Filter "SamAccountName -eq '$($user.SamAccountName)'")) {
        Write-Host "Creating user: $($user.SamAccountName)" -ForegroundColor Green

        New-ADUser -Name "$($user.FirstName) $($user.LastName)" `
                   -GivenName $user.FirstName `
                   -Surname $user.LastName `
                   -SamAccountName $user.SamAccountName `
                   -UserPrincipalName $user.UserPrincipalName `
                   -Path $user.OU `
                   -AccountPassword $securePassword `
                   -ChangePasswordAtLogon $true `
                   -Enabled $true
    } else {
        Write-Warning "User $($user.SamAccountName) already exists. Skipping..."
    }
}
```

### Script Logic Explained

```
1. Import-Module ActiveDirectory
   └── Loads the AD PowerShell module (required on the DC)

2. Import-Csv
   └── Reads users.csv into a collection of objects

3. foreach loop — for each row in the CSV:
   ├── ConvertTo-SecureString
   │   └── Converts plain-text password to SecureString (required by AD)
   │
   ├── Get-ADUser (check if exists)
   │   └── Skips creation if SamAccountName already exists → avoids duplicates
   │
   └── New-ADUser
       ├── -Name              Full display name
       ├── -GivenName         First name
       ├── -Surname           Last name
       ├── -SamAccountName    Windows logon name
       ├── -UserPrincipalName UPN (email-style login)
       ├── -Path              OU distinguished name from CSV
       ├── -AccountPassword   Secure password
       ├── -ChangePasswordAtLogon $true  ← user must reset on first login
       └── -Enabled $true     Account is active immediately
```

### Key Parameters Reference

| Parameter | Purpose |
|---|---|
| `-Name` | Display name shown in ADUC |
| `-SamAccountName` | Short logon name (domain\username) |
| `-UserPrincipalName` | Full UPN login (username@domain) |
| `-Path` | OU where the account is created |
| `-AccountPassword` | Initial password as SecureString |
| `-ChangePasswordAtLogon $true` | Forces password reset on first login |
| `-Enabled $true` | Account is active (not disabled) |

---

## ▶️ Step 4 — Run the Script

```
1. On the Domain Controller, open PowerShell ISE
   → Search: PowerShell ISE → Right-click → Run as Administrator

2. Open the script file
   File → Open → navigate to the script → Open

3. Update the CSV path if needed:
   $csvFile = "C:\users.csv"   ← change to match your file location

4. Click Run (▶) or press F5

5. Watch the output:
   ✅ Green text = user created successfully
   ⚠️ Yellow warning = user already exists, skipped
```

---

## ✅ Step 5 — Verify in Active Directory

```
Server Manager → Tools → Active Directory Users and Computers
→ Expand DC.local → expand CRM OU
→ Confirm all 5 users appear:
   ├── Alice Baker    (abaker)
   ├── Charlie Davis  (cdavis)
   ├── Edward Fisher  (efisher)
   ├── Grace Hill     (ghill)
   └── Isaac Jones    (ijones)
```

Or verify via PowerShell:

```powershell
Get-ADUser -Filter * -SearchBase "OU=CRM,DC=DC,DC=local" |
    Select-Object Name, SamAccountName, Enabled
```

---

## ⚡ Time Comparison

| Method | Time for 30 users | Error risk |
|---|---|---|
| Manual (ADUC, one by one) | 30–45 minutes | High (typos, missed fields) |
| CSV + PowerShell script | Under 30 seconds | Low (consistent, repeatable) |
| AI-generated CSV + script | Under 2 minutes total | Very low |

---

## 📋 Process Summary

| Step | Action | Tool |
|---|---|---|
| 1 | Write a prompt describing the CSV and script needed | Microsoft Copilot / ChatGPT |
| 2 | Copy AI output → save as `users.csv` | Notepad |
| 3 | Fill in real employee data in the CSV | Notepad / Excel |
| 4 | Transfer CSV to the AD server (`C:\users.csv`) | File copy |
| 5 | Open the PowerShell script in PowerShell ISE as Admin | PowerShell ISE |
| 6 | Update CSV path in script → Run | PowerShell ISE |
| 7 | Verify users created in ADUC or via `Get-ADUser` | ADUC / PowerShell |

---

## ⚠️ Important Notes

- The CSV file must be saved with **comma-separated** values — not semicolons or tabs.
- The **OU distinguished name** in the CSV must exactly match the OU path in Active Directory (e.g., `OU=CRM,DC=DC,DC=local`).
- Passwords must meet the domain's **complexity requirements** (uppercase, lowercase, number, special character, min 7 characters).
- Users are set to **change password at next logon** — communicate initial passwords securely to employees.
- The script **skips duplicate accounts** rather than overwriting — safe to run multiple times.
- Always run PowerShell ISE or PowerShell as **Administrator** on the Domain Controller.

---

## 📚 Terminology Reference

| Term | Definition |
|---|---|
| **CSV** | Comma-Separated Values — a plain text file where each row is a record and columns are separated by commas |
| **PowerShell** | Microsoft's automation framework; used for scripting and managing Windows environments |
| **PowerShell ISE** | Integrated Scripting Environment — GUI editor for writing and running PowerShell scripts |
| **Import-Csv** | PowerShell cmdlet that reads a CSV file and converts each row into an object |
| **New-ADUser** | PowerShell cmdlet that creates a new user account in Active Directory |
| **SamAccountName** | The pre-Windows 2000 logon name; used in `DOMAIN\username` format |
| **UserPrincipalName** | The UPN login name in `username@domain` format |
| **SecureString** | An encrypted string type required by AD cmdlets for passwords |
| **Microsoft Copilot** | AI assistant integrated into Windows and Microsoft 365 for generating code and automating tasks |
| **OU Distinguished Name** | The full LDAP path of an OU, e.g., `OU=CRM,DC=DC,DC=local` |

---

## 🔭 Next Session Preview

- Applying **Group Policy Objects (GPOs)** to control user and computer settings
- How policies propagate from the DC to domain-joined machines
- Using `gpupdate /force` and `gpresult` to verify applied policies

---

## 📄 License

This repository is for educational purposes. Feel free to fork and adapt for your own learning.
