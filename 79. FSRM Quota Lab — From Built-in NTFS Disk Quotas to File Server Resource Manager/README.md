# Lab: From Built-in NTFS Disk Quotas to File Server Resource Manager (FSRM) Quotas

## Overview

This lab documents two generations of disk quota management on Windows Server: the **legacy, built-in NTFS volume quota** feature, and the modern, far more flexible **File Server Resource Manager (FSRM)** quota system. The lab first demonstrates the old per-volume NTFS method and its limitations, then installs FSRM and rebuilds the same restriction using a reusable **quota template** applied at the folder level — showing why FSRM is the recommended approach on any current Windows Server deployment.

**Lab environment:**
- Server: `PDC16.company.local`
- Volume used for legacy quotas: `New Volume (E:)`
- Folder used for FSRM quotas: `E:\Data`
- Test user: `mohamed@company.local` / warning shown for `Ahmed Abdo (ahmed.abdo@company.local)`

**Goal:** Understand both quota mechanisms, see their outputs from the end-user's perspective, and migrate from the old per-volume method to a template-based FSRM quota that can be applied automatically across many folders.

---

## Table of Contents

1. [Task 1 – Enable Legacy NTFS Disk Quotas on a Volume](#task-1--enable-legacy-ntfs-disk-quotas-on-a-volume)
2. [Task 2 – Add a Per-User Quota Entry (Old Method)](#task-2--add-a-per-user-quota-entry-old-method)
3. [Task 3 – Confirm the Limited Space Is Visible to the User](#task-3--confirm-the-limited-space-is-visible-to-the-user)
4. [Task 4 – Monitor a Quota Entry Reaching Its Warning Level](#task-4--monitor-a-quota-entry-reaching-its-warning-level)
5. [Task 5 – Install the File Server Resource Manager (FSRM) Role](#task-5--install-the-file-server-resource-manager-fsrm-role)
6. [Task 6 – Create a Reusable Quota Template with a Notification Threshold](#task-6--create-a-reusable-quota-template-with-a-notification-threshold)
7. [Task 7 – Apply the Quota Template to a Folder (Hard Quota, Auto-Apply)](#task-7--apply-the-quota-template-to-a-folder-hard-quota-auto-apply)
8. [Summary / Key Takeaways](#summary--key-takeaways)

---

## Task 1 – Enable Legacy NTFS Disk Quotas on a Volume

On `New Volume (E:)` → **Properties → Quota** tab, enable:

- ☑ **Enable quota management**
- ☑ **Deny disk space to users exceeding quota limit**
- **Default quota limit for new users on this volume:** Limit disk space to `10 MB`, set warning level to `7 GB` *(note: warning level is set higher than the limit itself here — see callout below)*
- ☑ **Log event when a user exceeds their quota limit**
- ☑ **Log event when a user exceeds their warning level**

![Enable NTFS Volume Quotas](task1-ntfs-quota-properties.png)

> ⚠️ **Note:** In this screenshot, the warning level (`7 GB`) is actually set *larger* than the hard limit (`10 MB`) — almost certainly a unit mismatch (GB selected instead of MB). In a real deployment, always double check the unit dropdown for both fields; a warning level that's numerically larger than the hard limit due to a unit mismatch means the warning will never trigger before the hard limit is hit.

**Why:** This is Windows' original, built-in disk quota system — it works entirely at the **volume level**, meaning limits can only be applied per user, per entire volume (not per folder). Enabling **"Deny disk space to users exceeding quota limit"** makes this a genuine **hard quota** — write operations fail once a user's limit is reached, not just a passive log entry.

---

## Task 2 – Add a Per-User Quota Entry (Old Method)

From the **Quota Entries** window, add a new entry for a specific user:

- **User:** `mohamed@company.local`
- **Limit disk space to:** `10 MB`
- **Set warning level to:** `5 MB`

![Add Quota Entry for User](task2-add-quota-entry-per-user.png)

**Why:** While the volume-level default (Task 1) applies to *any new user*, individual **Quota Entries** let an administrator override that default for specific users — useful for granting some users more or less space than the volume-wide default.

---

## Task 3 – Confirm the Limited Space Is Visible to the User

From the client/user's perspective, the mapped drive correctly reflects the imposed limit rather than the volume's real physical capacity:

```
Data (\\PDC16) (Z:)
9.51 MB free of 10.0 MB
```

![Quota Limit Visible in Explorer](task3-quota-limit-visible-in-explorer.png)

**Why:** This confirms the quota isn't just an administrative/logging construct — Windows Explorer itself reports the **quota-limited size** (`10.0 MB`) as if it were the drive's actual total capacity, so users get accurate, native feedback about their available space without needing any special client-side software.

---

## Task 4 – Monitor a Quota Entry Reaching Its Warning Level

Back in **Quota Entries**, a user's entry now shows a **Warning** status once usage crosses the configured warning threshold:

| Status | Name | Logon Name | Amount Used | Quota Limit | Warning Level | Percent Used |
|---|---|---|---|---|---|---|
| ⚠️ Warning | Ahmed Abdo | ahmed.abdo@company.local | 9 MB | 10 MB | 5 MB | 90% |

![Quota Entry Warning Status](task4-quota-entry-warning-status.png)

**Why:** This demonstrates the built-in quota system's **monitoring** side — administrators can see, at a glance, which users are approaching their limit (here, 90% used) before they're actually blocked from writing more data, giving early visibility into users who may need cleanup or a higher allocation.

---

## Task 5 – Install the File Server Resource Manager (FSRM) Role

Using **Add Roles and Features**, install:

- **File and Storage Services → File and iSCSI Services → File Server Resource Manager**
- **Remote Server Administration Tools → Role Administration Tools → File Services Tools → File Server Resource Manager Tools**

![Install FSRM Role](task5-install-fsrm-role.png)

**Why:** The legacy NTFS quota system (Tasks 1–4) is limited to whole-volume scope and lacks flexibility (no reusable templates, no email notifications, no reporting). **FSRM** replaces it with **folder-level** quotas, reusable **templates**, multi-channel notifications (event log, email, command execution, reports), and is the quota mechanism Microsoft recommends going forward.

---

## Task 6 – Create a Reusable Quota Template with a Notification Threshold

In **FSRM → Quota Management → Quota Templates → Create Quota Template**:

- **Template name:** `10 MB Limit`
- **Space limit:** `10 MB`
- **Quota type:** **Hard quota** (Do not allow users to exceed limit)
- **Notification threshold:** Add a threshold at **85%** usage

Under the threshold's **Event Log** tab:
- ☑ **Send warning to event log**
- **Log entry** uses FSRM's built-in variables, e.g.:
  > *User [Source Io Owner] has exceeded the [Quota Threshold]% quota threshold for the quota on [Quota Path] on server [Server]. The quota limit is [Quota Limit MB] MB, and [Quota Used MB] MB currently is in use ([Quota Used Percent]% of limit).*

![Create Quota Template with Threshold](task6-create-quota-template-with-threshold.png)

**Why:** Unlike the old per-volume method's fixed warning behavior, FSRM templates let you define **multiple notification thresholds** (e.g., 85%, 95%, 100%), each with its own combination of actions — event log entries, emails to the user/admin, running a command/script, or generating a report — and, critically, the **template itself is reusable** across any number of folders instead of being tied to one volume's single default.

---

## Task 7 – Apply the Quota Template to a Folder (Hard Quota, Auto-Apply)

In **FSRM → Quota Management → Quotas → Create Quota**:

- **Quota path:** `E:\Data`
- **Auto apply template and create quotas on existing and new subfolders**
- **Derive properties from this quota template (recommended):** `10 Mb Limit`

**Summary of quota properties:**
```
Auto Apply Quota: E:\Data
  Source template: 10 Mb Limit
  Limit: 10.0 MB (Hard)
  Notification: 2
    Warning (85%): Event log
```

![Apply Quota Template to Folder](task7-apply-quota-template-to-path.png)

**Why:** **Auto apply** is what makes FSRM genuinely scalable compared to the old method — instead of manually setting a quota on every existing and future subfolder under `E:\Data`, this single configuration automatically applies the `10 Mb Limit` template to **every current and future subfolder**, with zero additional admin effort as new user/project folders are created underneath it.

---

## Summary / Key Takeaways

| Aspect | Legacy NTFS Quotas | FSRM Quotas |
|---|---|---|
| Scope | Whole volume only | Any folder path, including auto-applied to subfolders |
| Reusability | Per-volume defaults only | Reusable templates across unlimited paths |
| Notifications | Event log only, single warning level | Multiple thresholds; event log, email, command, and report actions |
| Automation | Manual per-user entries | Auto-apply to existing *and future* subfolders |
| Recommended for new deployments | ❌ Legacy / limited | ✅ Yes |

**Key takeaway:** The built-in NTFS quota system (Tasks 1–4) still works and is useful for understanding the fundamentals of how Windows enforces and reports disk quotas to end users — but it's fundamentally limited to whole-volume scope with no reuse or rich notification options. FSRM (Tasks 5–7) solves all of those limitations with folder-level, template-driven, auto-applying quotas — which is why virtually all modern Windows Server file-quota deployments use FSRM instead of the legacy per-volume method.
