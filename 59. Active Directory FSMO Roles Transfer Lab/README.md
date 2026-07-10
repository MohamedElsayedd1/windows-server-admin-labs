# Active Directory FSMO Roles Transfer Lab

## Overview

This lab demonstrates how to transfer and seize **FSMO (Flexible Single Master Operations)** roles in an Active Directory environment. The scenario involves migrating all five FSMO roles from a Primary Domain Controller (`PDC.test.local`) to an Additional Domain Controller (`ADC.test.local`), handling both graceful transfers (when the source DC is online) and forced seizures (when it is offline).

### Environment

| Server | Role |
|--------|------|
| `PDC.test.local` | Source DC — original FSMO role holder |
| `ADC.test.local` | Target DC — destination for all FSMO roles |

### FSMO Roles Covered

| Role | Scope | Tool Used |
|------|-------|-----------|
| RID Master | Domain | Active Directory Users and Computers |
| PDC Emulator | Domain | Active Directory Users and Computers |
| Infrastructure Master | Domain | Active Directory Users and Computers |
| Domain Naming Master | Forest | Active Directory Domains and Trusts |
| Schema Master | Forest | Active Directory Schema (MMC Snap-in) |

---

## Prerequisites

- Two Domain Controllers joined to the same domain
- Administrator credentials with Domain Admin (and Schema Admin for Schema Master)
- Both DCs reachable for graceful transfer; only the target DC needed for seizure
- Remote Server Administration Tools (RSAT) installed

---

## Part 1 — Graceful Transfer via GUI

These steps are performed while **both DCs are online**.

---

### Task 1 — Transfer RID Master Role to ADC

**Steps:**
1. Open **Active Directory Users and Computers** (`dsa.msc`)
2. Right-click the domain root → **Operations Masters**
3. Select the **RID** tab
4. Verify the current Operations Master is `PDC.test.local`
5. Click **Change…** to transfer the role to `ADC.test.local`
6. Confirm the prompt — a success dialog will appear

**Screenshot:**

![Task 1 – Transfer RID Role](1777374967492_task1-transfer-rid-to-adc.png)

> ✅ **Expected Result:** Dialog reads *"The operations master role was successfully transferred."* The Operations master field now shows `ADC.test.local`.

---

### Task 2 — Transfer PDC Emulator Role to ADC

**Steps:**
1. In **Active Directory Users and Computers**, right-click the domain root → **Operations Masters**
2. Select the **PDC** tab
3. Click **Change…** to transfer the PDC Emulator role to `ADC.test.local`
4. Confirm the success dialog

**Screenshot:**

![Task 2 – Transfer PDC Role](1777374967493_task2-transfer-pdc-to-adc.png)

> ✅ **Expected Result:** PDC Emulator role moved to `ADC.test.local`.

---

### Task 3 — Transfer Infrastructure Master Role to ADC

**Steps:**
1. In **Active Directory Users and Computers**, right-click the domain root → **Operations Masters**
2. Select the **Infrastructure** tab
3. Click **Change…** to transfer the Infrastructure Master role
4. Confirm the success dialog

**Screenshot:**

![Task 3 – Transfer Infrastructure Role](1777374967494_task3-transfer-infastructure-to-adc.png)

> ✅ **Expected Result:** Infrastructure Master role moved to `ADC.test.local`.

---

### Task 4 — Change Active Directory Focus to ADC

Before transferring the Domain Naming Master role, you must point the management console to `ADC.test.local`.

**Steps:**
1. Open **Active Directory Domains and Trusts** (`domain.msc`)
2. Right-click **Active Directory Domains and Trusts** in the left pane → **Change Active Directory Domain Controller…**
3. In the *Change Directory Server* dialog, select `ADC.test.local` from the list
4. Click **OK**

**Screenshot:**

![Task 4 – Change Active Directory Focus](1777374967494_task4-change-active-dir.png)

> ℹ️ Both `ADC.test.local` and `PDC.test.local` are visible as Global Catalog (GC) servers. The current focus is changed to ADC before proceeding.

---

### Task 5 — Transfer Domain Naming Master Role to ADC

**Steps:**
1. In **Active Directory Domains and Trusts**, right-click the root node → **Operations Master…**
2. In the *Operations Master* dialog, click **Change** to transfer the Domain Naming Master role to `ADC.test.local`
3. Confirm the success dialog

**Screenshot:**

![Task 5 – Transfer Domain Naming Master](1777374967494_task5-transfer-domain-name-role.png)

> ✅ **Expected Result:** Dialog reads *"The operations master was successfully transferred."*

---

### Task 6 — Register the Schema Management DLL

The Active Directory Schema snap-in is not registered by default and must be enabled before use.

**Steps:**
1. Press **Win + R** to open the Run dialog
2. Type: `regsvr32 schmmgmt.dll`
3. Click **OK** — Windows will register the DLL with administrative privileges
4. A confirmation dialog confirms successful registration

**Screenshot:**

![Task 6 – Register Schema DLL](1777374967495_task6-enable-schema-master-role.png)

> ⚠️ This step is required only once per machine. Without it, the Active Directory Schema snap-in will not appear in the MMC console.

---

### Task 7 — Add Active Directory Schema Snap-in to MMC

**Steps:**
1. Press **Win + R**, type `mmc`, press Enter
2. In the MMC console, go to **File → Add/Remove Snap-ins…**
3. In the *Available snap-ins* list, select **Active Directory Schema**
4. Click **Add >** — it will appear in the *Selected snap-ins* panel
5. Click **OK**

**Screenshot:**

![Task 7 – Add AD Schema Snap-in](1777374967495_task7-add-ad-schema.png)

> ℹ️ The description confirms: *"View and edit the Active Directory Schema."*

---

### Task 8 — Transfer Schema Master Role to ADC

**Steps:**
1. In the MMC console, expand **Active Directory Schema [ADC.test.local]**
2. Right-click **Active Directory Schema** → **Operations Master…**
3. In the *Change Schema Master* dialog:
   - Current schema master (online): `PDC.test.local`
   - Target: `ADC.test.local`
4. Click **Change** to initiate the transfer

**Screenshot:**

![Task 8 – Transfer Schema Master](1777374967496_task8-transfer-schema.png)

> ✅ **Expected Result:** Schema Master role transferred from `PDC.test.local` to `ADC.test.local`.

---

## Part 2 — Transfer via PowerShell / CMD

---

### Task 9 — Transfer All FSMO Roles via PowerShell (Single Command)

You can transfer all five roles with a single PowerShell command.

**Command:**
```powershell
Move-ADDirectoryServerOperationMasterRole -Identity "pdc16" -OperationMasterRole 0,1,2,3,4
```

**Role Number Reference:**

| Number | Role |
|--------|------|
| 0 | PDCEmulator |
| 1 | RIDMaster |
| 2 | InfrastructureMaster |
| 3 | SchemaMaster |
| 4 | DomainNamingMaster |

**Screenshot:**

![Task 9 – PowerShell Transfer](1777374967497_task9-transfer-by-cmd.png)

> ℹ️ The cmdlet prompts confirmation for each role. Answer **Y** (or **A** for Yes to All) for each. All five roles are moved to `PDC16.company.local` in this example.

---

### Task 10 — Transfer Error: Source DC Offline

If you attempt a GUI transfer when the source DC is offline, you will encounter the following error.

**Error Message:**
> *"The current operations master is offline. The role cannot be transferred."*

**Screenshot:**

![Task 10 – Transfer Error (DC Offline)](1777374967497_task10-transfer-error-because-down.png)

> ⚠️ When the source DC is offline, a graceful transfer is impossible. You must perform a **seizure** instead (see Task 15).

---

### Task 11 — Verify FSMO Role Holders via CMD

After transfer, verify all roles are assigned correctly.

**Command:**
```cmd
netdom query fsmo
```

**Screenshot:**

![Task 11 – Verify FSMO with netdom](1777374967497_task11-verify.png)

**Expected Output:**
```
Schema master          PDC16.company.local
Domain naming master   PDC16.company.local
PDC                    PDC16.company.local
RID pool manager       PDC16.company.local
Infrastructure master  PDC16.company.local
The command completed successfully.
```

> ✅ All five FSMO roles are confirmed on the target server.

---

### Task 12 — Transfer Roles Back to ADC via PowerShell + Verify

This task demonstrates transferring all five roles back to `ADC.company.local` and immediately verifying the result.

**Command:**
```powershell
Move-ADDirectoryServerOperationMasterRole -Identity "ADC" -OperationMasterRole 0,1,2,3,4
netdom query fsmo
```

**Screenshot:**

![Task 12 – Transfer Roles Back + Verify](1777374967498_task12-transfer-roles-by-powershell.png)

**Expected Output After Transfer:**
```
Schema master          ADC.company.local
Domain naming master   ADC.company.local
PDC                    ADC.company.local
RID pool manager       ADC.company.local
Infrastructure master  ADC.company.local
The command completed successfully.
```

---

## Part 3 — Decommissioning the Old PDC

---

### Task 13 — Force Remove AD DS from PDC (Demotion)

When the PDC is being decommissioned, use the AD DS Configuration Wizard with **Force Removal** enabled (useful if normal demotion fails or the PDC cannot communicate with other DCs).

**Steps:**
1. Open **Server Manager → Manage → Remove Roles and Features**
2. Select **Active Directory Domain Services** for removal
3. The AD DS Configuration Wizard launches automatically
4. On the **Credentials** page:
   - Check **Force the removal of this domain controller**
   - Note the warning: *"Unless this is the last domain controller in the domain, you must perform metadata cleanup manually after removal."*
5. Click **Next >**

**Screenshot:**

![Task 13 – Force Remove PDC AD DS](1777374967498_task13-force-remove-pdc-ad.png)

> ⚠️ Force removal bypasses normal replication. Always transfer FSMO roles **before** demoting. If the DC was forcibly removed, run `ntdsutil` metadata cleanup on a surviving DC.

---

### Task 14 — Demotion Progress

During the demotion, the wizard shows progress and warnings.

**Screenshot:**

![Task 14 – Demotion Progress](1777374967498_task14-demotion.png)

**Key Warning:**
> *"DNS Server service has been detected on this server. Any existing Active Directory integrated zones will be deleted during the removal of Active Directory Domain Services (AD DS) on this server. After the forced removal of AD DS, you should delete any existing DNS delegations pointing to this server."*

> ⚠️ After demotion completes, the server restarts automatically. Manually remove DNS delegations and metadata records pointing to the old PDC from the surviving DC.

---

## Part 4 — FSMO Seizure (Source DC Offline)

---

### Task 15 — Seize FSMO Roles Using ntdsutil

When the original FSMO holder is **permanently offline or unavailable**, roles must be **seized** (forcibly taken) rather than transferred.

**Steps:**
```cmd
ntdsutil
  roles
  connections
    connect to server pdc16.company.local
    quit
  seize schema master
  seize naming master
  seize rid master
  seize pdc
  seize infrastructure master
  quit
quit
```

**Screenshot:**

![Task 15 – Seize FSMO via ntdsutil](1777374967499_task15-seize.png)

**Understanding the Output:**

| Message | Meaning |
|---------|---------|
| `Attempting safe transfer of schema FSMO before seizure.` | ntdsutil tries a graceful transfer first |
| `FSMO transferred successfully – seizure not required.` | Schema Master was transferred normally (source was reachable) |
| `ldap_modify_sW error 0x51(81 (Server Down).` | Source DC unreachable for this role |
| `Transfer of [role] FSMO failed, proceeding with seizure...` | Graceful transfer failed; forced seizure initiated |
| `DsListRolesW error 0x6(The handle is invalid.)` | Expected after seizure — role list handle is reset |

> ⚠️ **Important:** Only seize FSMO roles if the original holder will **never** be brought back online. If the original DC comes back online after a seizure, it must be forcibly demoted immediately to avoid a split-brain conflict.

---

## Summary: All Transfer Methods

| Method | When to Use | Tool |
|--------|-------------|------|
| GUI Transfer (ADUC) | RID, PDC, Infrastructure — source DC online | `dsa.msc` |
| GUI Transfer (Domains and Trusts) | Domain Naming Master — source online | `domain.msc` |
| GUI Transfer (Schema MMC) | Schema Master — source online | `mmc` + `schmmgmt.dll` |
| PowerShell Transfer | All 5 roles at once — source online | `Move-ADDirectoryServerOperationMasterRole` |
| ntdsutil Seizure | Source DC offline/unreachable | `ntdsutil` |

---

## Verification Commands

```powershell
# Verify all FSMO roles (CMD)
netdom query fsmo

# Verify using PowerShell
Get-ADDomain | Select-Object PDCEmulator, RIDMaster, InfrastructureMaster
Get-ADForest | Select-Object SchemaMaster, DomainNamingMaster

# Verify using netdom (PowerShell)
netdom query fsmo /domain:test.local
```

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| *"The current operations master is offline. The role cannot be transferred."* | Source DC is down | Use `ntdsutil` seizure |
| Schema snap-in missing from MMC | DLL not registered | Run `regsvr32 schmmgmt.dll` |
| `ldap_modify_sW error 0x51` | Server unreachable during seizure | Expected — seizure proceeds automatically |
| PDC Emulator seizure fails with `0x20af` | Source DC completely unavailable | ntdsutil proceeds with forced seizure |
| DNS zones deleted after demotion | Force removal deletes AD-integrated DNS | Recreate zones or restore from backup on surviving DC |
| Metadata cleanup warning after force removal | Replication metadata not cleaned | Run `ntdsutil → metadata cleanup` on a surviving DC |

---

## References

- [Microsoft Docs: Transfer FSMO roles](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/transfer-or-seize-fsmo-roles-in-ad-ds)
- [Microsoft Docs: Seize FSMO roles](https://learn.microsoft.com/en-us/troubleshoot/windows-server/active-directory/seize-fsmo-roles)
- [Move-ADDirectoryServerOperationMasterRole](https://learn.microsoft.com/en-us/powershell/module/activedirectory/move-addirectoryserveroperationmasterrole)
- [netdom query fsmo](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc835086(v=ws.11))
