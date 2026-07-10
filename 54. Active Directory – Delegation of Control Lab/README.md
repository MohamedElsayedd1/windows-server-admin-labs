# 🔑 Active Directory – Delegation of Control Lab

A hands-on lab demonstrating how to use the **Delegation of Control Wizard** in Active Directory to grant a non-admin user granular administrative permissions over a specific Organizational Unit (OU) — without giving them full Domain Admin privileges. The lab also verifies the **boundaries** of delegated permissions: the user can manage their assigned OU but is denied actions outside of it.

---

## 📋 Table of Contents

1. [Background & Concept](#background--concept)
2. [Lab Environment](#lab-environment)
3. [Task 1 – Prerequisites: Understand the Scenario](#task-1--prerequisites-understand-the-scenario)
4. [Task 2 – Run the Delegation of Control Wizard](#task-2--run-the-delegation-of-control-wizard)
5. [Task 3 – Select Tasks to Delegate](#task-3--select-tasks-to-delegate)
6. [Task 4 – Verify Delegated User Can Access and Manage the OU](#task-4--verify-delegated-user-can-access-and-manage-the-ou)
7. [Task 5 – Verify Delegated User Cannot Perform Actions Outside Their OU](#task-5--verify-delegated-user-cannot-perform-actions-outside-their-ou)
8. [Summary](#-summary)
9. [Key Concepts](#-key-concepts)
10. [Delegation vs Domain Admin](#-delegation-vs-domain-admin)
11. [Troubleshooting](#️-troubleshooting)

---

## 📖 Background & Concept

### What is Delegation of Control?

In a typical Active Directory environment, only **Domain Administrators** can create, modify, or delete user accounts, reset passwords, and manage groups. However, granting full Domain Admin rights to helpdesk staff or department managers just so they can manage their own users is a major security risk — it violates the **Principle of Least Privilege**.

**Delegation of Control** solves this problem by allowing Domain Admins to grant **specific, limited permissions** on a **specific OU** to a specific user or group. The delegated user can perform only the allowed actions on the objects within that OU — nothing more, nothing less.

### Why is this important?

| Without Delegation | With Delegation |
|-------------------|-----------------|
| Helpdesk needs Domain Admin rights to reset passwords | Helpdesk gets password-reset-only rights on their OU |
| Department manager can't manage their own users | Manager can create/delete/manage users in their OU |
| Security risk: over-privileged accounts | Least privilege enforced: minimal attack surface |
| Single point of failure: one admin team | Distributed management: each team manages their own scope |

### How it works technically

Delegation works by modifying the **Access Control List (ACL)** of the target OU. The Delegation of Control Wizard writes specific **Access Control Entries (ACEs)** to the OU's `nTSecurityDescriptor` attribute, granting the specified user or group the chosen permissions on objects within that OU only.

---

## 🖥️ Lab Environment

| Component | Details |
|-----------|---------|
| **Domain** | DC.local |
| **Domain Controller** | DC.local |
| **Target OU** | HR |
| **Delegated User** | Ahmed Abdo (ahmed.abdo@DC.local) |
| **OU Contents** | Ahmed Abdo (User), Ahmed Ali (User), HR-Group (Security Group), HR-PC01 (Computer) |
| **Tool** | Active Directory Users and Computers (ADUC) |
| **Outside OU (test)** | Aya Ibrahim — user in a different OU |

---

## Task 1 – Prerequisites: Understand the Scenario

### 📖 Explanation
Before running the Delegation of Control Wizard, it is essential to understand the **scenario** and what we are trying to achieve. In this lab:

- The **HR OU** contains users, a group, and a computer that need to be managed day-to-day
- **Ahmed Abdo** is a regular domain user (not a Domain Admin) who works as the HR department's IT liaison
- The goal is to give Ahmed Abdo the ability to **create/delete/manage user accounts** and **reset passwords** within the HR OU — without making him a Domain Admin
- Ahmed Abdo should **not** be able to manage users or objects in any other OU (e.g., IT, Finance, Sales)

### 🔧 Steps
1. Log on to the Domain Controller as a **Domain Administrator**
2. Open **Active Directory Users and Computers (ADUC)**: `dsa.msc`
3. Expand **DC.local** and locate the **HR** OU
4. Verify the HR OU contains the users and objects you expect to delegate management over
5. Confirm that **Ahmed Abdo** exists as a domain user account (ahmed.abdo@DC.local)
6. Note which tasks you will delegate: user account management + password reset

### ✅ Solution / Expected Result
You can see the HR OU with its contents (users, groups, computers) and the Ahmed Abdo user account exists and is ready to be used as the delegation target.

---

## Task 2 – Run the Delegation of Control Wizard

### 📖 Explanation
The **Delegation of Control Wizard** is the built-in ADUC tool for assigning granular permissions to users or groups over a specific OU. It provides a guided, wizard-driven interface that abstracts the complexity of directly editing ACLs. The wizard:

1. Lets you pick the **target OU** (the scope of delegation)
2. Lets you pick **who** receives the permissions (user or group)
3. Lets you pick **what** permissions are granted (common tasks or custom)

The wizard modifies the OU's **ACL** behind the scenes — you can verify this afterward in the OU's **Properties → Security** tab (requires Advanced Features view in ADUC).

### 🔧 Steps
1. In **ADUC**, right-click the **HR** OU in the left tree pane
2. Select **Delegate Control…** from the context menu
3. The **Delegation of Control Wizard** opens — click **Next**
4. On the **Users or Groups** page, click **Add…**
5. In the **Select Users, Computers, or Groups** dialog:
   - Type `Ahmed Abdo` or `ahmed.abdo` in the search box
   - Click **Check Names** to resolve the account
   - The account resolves to: **Ahmed Abdo (ahmed.abdo@DC.local)**
   - Click **OK**
6. Confirm Ahmed Abdo appears in the **"Selected users and groups"** list
7. Click **Next** to proceed to task selection

### ✅ Solution / Expected Result
The wizard shows **Ahmed Abdo (ahmed.abdo@DC.local)** in the Selected users and groups list. The user is correctly identified and resolved — the wizard is ready to proceed to task selection.

**Screenshot:**

![Task 2 – Delegation of Control Wizard – User Selection](1776920044433_task2-delegation-of-control.png)

> **Best Practice:** In production environments, delegate to a **security group** rather than an individual user account (e.g., delegate to `HR-Admins` group, then add HR IT liaisons to that group). This makes permission management easier — to revoke access, simply remove the user from the group rather than re-running the wizard.

---

## Task 3 – Select Tasks to Delegate

### 📖 Explanation
The **Tasks to Delegate** page is where you define the **scope of permissions** granted to the selected user. The wizard offers two modes:

- **Common Tasks** — pre-built task templates covering the most frequently delegated actions (e.g., reset passwords, manage user accounts, manage groups). These are safe and well-defined.
- **Custom Task** — allows granular, attribute-level permission configuration for advanced scenarios (e.g., allow modification of only the `telephoneNumber` attribute on user objects).

In this lab, we delegate two common tasks:
1. **Create, delete, and manage user accounts** — allows the user to add/remove users and modify their properties within the OU
2. **Reset user passwords and force password change at next logon** — the most common helpdesk delegation task

### 🔧 Steps
1. On the **Tasks to Delegate** page, ensure **"Delegate the following common tasks"** is selected
2. Check the following tasks:
   - ✅ **Create, delete, and manage user accounts**
   - ✅ **Reset user passwords and force password change at next logon**
3. Leave all other tasks unchecked (Read all user information, Create/delete groups, etc.)
4. Click **Next**
5. Review the summary page showing the delegation details
6. Click **Finish** to apply the delegation

### ✅ Solution / Expected Result
Two tasks are selected: user account management and password reset. The wizard completes and writes the corresponding ACEs to the HR OU's security descriptor. Ahmed Abdo now has these permissions scoped exclusively to the HR OU.

**Screenshot:**

![Task 3 – Tasks to Delegate](1776920044433_task3-delegation-tasks.png)

> **What gets written to AD?** The wizard adds ACEs to the HR OU's `nTSecurityDescriptor`. You can view these by: right-clicking the HR OU → Properties → Security tab (requires View → Advanced Features enabled in ADUC). You'll see Ahmed Abdo listed with specific allow permissions.

---

## Task 4 – Verify Delegated User Can Access and Manage the HR OU

### 📖 Explanation
After delegation is configured, it is critical to **verify** it works as expected. The delegated user (Ahmed Abdo) should now be able to:
- Open ADUC and **see the HR OU** and its contents
- **Create, modify, or delete** user accounts within the HR OU
- **Reset passwords** for HR users

This verification step confirms the wizard applied the permissions correctly and that Ahmed Abdo's session reflects the updated ACL. This test should be performed by **logging on as Ahmed Abdo** (not as a Domain Admin).

### 🔧 Steps
1. **Log off** from the Domain Admin session
2. **Log on** to a domain-joined machine as **Ahmed Abdo** (ahmed.abdo@DC.local)
3. Open **Active Directory Users and Computers**: `dsa.msc`
4. Expand **DC.local** → navigate to the **HR OU**
5. Verify you can **see all objects** in the HR OU:
   - Ahmed Abdo (User)
   - Ahmed Ali (User)
   - HR-Group (Security Group)
   - HR-PC01 (Computer)
6. Test the delegated permissions:
   - Right-click a user (e.g., Ahmed Ali) → **Reset Password** — this should succeed ✅
   - Right-click in the HR OU → **New → User** — this should succeed ✅
   - Modify a user's properties — this should succeed ✅

### ✅ Solution / Expected Result
Ahmed Abdo can successfully open ADUC, navigate to the HR OU, and view all objects. The delegated actions (password reset, user creation, user management) work correctly within the HR OU scope.

**Screenshot:**

![Task 4 – Delegated User Accesses and Views the HR OU](1776920044434_task4-accessed-ad.png)

> **Note:** Ahmed Abdo can only **see and manage** the HR OU contents. Other OUs (IT, Finance, Sales) may be visible in the tree but he cannot modify objects within them — the ACL on those OUs does not grant him any permissions.

---

## Task 5 – Verify Delegated User Cannot Perform Actions Outside Their OU

### 📖 Explanation
A core principle of Delegation of Control is **scope enforcement** — the delegated permissions apply **only** to the specific OU they were granted on. This task verifies the **security boundary** of the delegation by attempting an action on an object **outside the HR OU**.

In this test, Ahmed Abdo attempts to delete a user named **Aya Ibrahim** who belongs to a **different OU** (not HR). Since the delegation was applied only to the HR OU's ACL, Ahmed Abdo has no permissions on objects in other OUs. Active Directory Domain Services enforces this boundary and blocks the action.

This is the "negative test" — confirming that the delegation does **not** accidentally grant permissions beyond the intended scope.

### 🔧 Steps
1. While logged in as **Ahmed Abdo**, navigate in ADUC to an OU **other than HR** (e.g., IT, Sales, or Users)
2. Locate the user **Aya Ibrahim** (who exists in a different OU)
3. Right-click **Aya Ibrahim** → select **Delete**
4. Active Directory Domain Services will display an error dialog
5. Read and note the error message

### ✅ Solution / Expected Result
Active Directory blocks the action and displays:

> **"You do not have sufficient privileges to delete Aya Ibrahim, or this object is protected from accidental deletion."**

This confirms the delegation is correctly scoped to the HR OU only. Ahmed Abdo cannot delete, modify, or manage objects outside his delegated scope — the Principle of Least Privilege is enforced.

**Screenshot:**

![Task 5 – Cannot Perform Actions on Objects Outside HR OU](1776920044434_task5-can_t-perform-action-on-another-mu.png)

> **Important:** The error message mentions two possible reasons — insufficient privileges **or** accidental deletion protection. In this context, the primary reason is **insufficient privileges** — Ahmed Abdo's ACL does not grant him Delete permission on Aya Ibrahim's OU. The accidental deletion protection would be a secondary block even if he had partial rights.

---

## 📝 Summary

| # | Task | Action Taken | Who Performs It | Outcome |
|---|------|-------------|-----------------|---------|
| 1 | Understand the scenario | Review HR OU contents and identify delegated user | Domain Admin | Scenario confirmed; Ahmed Abdo exists as a domain user |
| 2 | Run Delegation Wizard | Add Ahmed Abdo as the delegated user for the HR OU | Domain Admin | Ahmed Abdo selected; wizard ready for task selection |
| 3 | Select Delegated Tasks | Check user management + password reset tasks | Domain Admin | Permissions written to HR OU ACL |
| 4 | Verify delegation works | Log on as Ahmed Abdo; open ADUC; view and manage HR OU | Ahmed Abdo | Can view, create, modify users; reset passwords in HR OU ✅ |
| 5 | Verify scope boundary | Attempt to delete Aya Ibrahim (outside HR OU) | Ahmed Abdo | Action blocked: insufficient privileges ✅ |

---

## 🔑 Key Concepts

| Concept | Description |
|---------|-------------|
| **Delegation of Control** | Granting specific AD permissions to a user/group over a specific OU, without making them a Domain Admin |
| **Principle of Least Privilege** | Users should have only the minimum permissions needed to perform their job — no more |
| **Organizational Unit (OU)** | An AD container used to organize objects (users, computers, groups) and apply policies or delegations |
| **ACL (Access Control List)** | A list of permissions attached to an AD object (like an OU), defining who can do what to that object |
| **ACE (Access Control Entry)** | A single permission entry in an ACL — e.g., "Allow Ahmed Abdo to Reset Password on User objects" |
| **Common Tasks** | Pre-built permission templates in the Delegation Wizard for typical helpdesk/management scenarios |
| **Custom Task** | Advanced delegation mode allowing attribute-level permission granularity |
| **Scope of Delegation** | The boundary of delegated permissions — in this lab, strictly the HR OU only |
| **nTSecurityDescriptor** | The AD attribute on every object that stores its ACL, modified by the Delegation Wizard |

---

## ⚖️ Delegation vs Domain Admin

| Capability | Delegated User (Ahmed Abdo) | Domain Admin |
|-----------|---------------------------|--------------|
| Manage users in HR OU | ✅ Yes | ✅ Yes |
| Reset passwords in HR OU | ✅ Yes | ✅ Yes |
| Manage users in IT/Finance/Sales OU | ❌ No | ✅ Yes |
| Create new OUs | ❌ No | ✅ Yes |
| Manage Domain Controllers | ❌ No | ✅ Yes |
| Modify Group Policy | ❌ No | ✅ Yes |
| Promote/demote servers | ❌ No | ✅ Yes |
| Delete objects outside HR OU | ❌ No (blocked) | ✅ Yes |

---

## 🛠️ Troubleshooting

| Problem | Likely Cause | Solution |
|---------|-------------|----------|
| Delegated user can't see the OU in ADUC | Permissions not applied or ADUC not refreshed | Run `gpupdate /force`; refresh ADUC (F5) |
| Password reset fails despite delegation | Task not checked in wizard, or policy blocking | Re-run wizard; verify "Reset user passwords" task was selected |
| Delegated user can perform actions in other OUs | Wizard was run on a parent OU instead of target OU | Check which OU the wizard was run on; remove over-broad ACEs |
| "Access denied" on all actions in HR OU | User not correctly resolved in wizard | Re-run wizard; use "Check Names" to verify user resolution |
| Cannot find "Delegate Control" in right-click menu | Advanced Features not enabled or user lacks admin rights | Enable View → Advanced Features in ADUC |
| Wizard finished but no permissions visible in Security tab | Advanced Features not enabled | Enable View → Advanced Features; check HR OU → Properties → Security |

---

## 🔍 How to Verify Delegation Permissions Manually

To confirm what ACEs were written to the HR OU by the delegation wizard:

1. In ADUC, enable **View → Advanced Features**
2. Right-click the **HR OU** → **Properties**
3. Go to the **Security** tab
4. Click **Advanced**
5. Look for **Ahmed Abdo** in the Permission Entries list
6. Click on his entry → **View** to see the exact permissions granted

This is useful for auditing, troubleshooting, or documenting what has been delegated.

---

*Lab completed on Active Directory Domain Services — Domain: DC.local | Delegated OU: HR | Delegated User: Ahmed Abdo (ahmed.abdo@DC.local)*
