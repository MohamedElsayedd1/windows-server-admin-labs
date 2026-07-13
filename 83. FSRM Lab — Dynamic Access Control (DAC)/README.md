# Dynamic Access Control (DAC) with FSRM — Windows Server Admin Lab

This lab walks through configuring **Dynamic Access Control (DAC)** on Windows Server, integrating **File Server Resource Manager (FSRM)** classification properties, Active Directory **claims**, **central access rules**, and **central access policies** to enforce attribute-based access control on a file share.

## Overview

Dynamic Access Control lets you control access to files and folders based on attributes of the user (department, location, etc.), the device (claims), and the resource (classification properties on the file/folder), instead of relying on NTFS/Share permissions alone. FSRM is used here to manage the resource property (classification) side of DAC, while AD Administrative Center is used to configure claim types, resource properties, and central access rules/policies.

## Prerequisites

- A working Active Directory domain with a Domain Controller
- A member file server joined to the domain
- Group Policy Management access to create/edit GPOs
- Test user accounts and a shared folder to apply the policy to

## Lab Steps

### 1. Share the Folder with Authenticated Users Permission
Create the folder to be protected and share it, granting **Authenticated Users** share-level permission as a baseline.

![Share folder with Authenticated Users permission](task1-share-folder-with-auth-users-perm.png)

### 2. Set NTFS Permissions for Authenticated Users
Configure NTFS permissions on the folder for **Authenticated Users**, since DAC layers on top of (not instead of) standard NTFS permissions.

![Authenticated Users NTFS permission](task2-auth-users-ntfs-perm.png)

### 3. Enable KDC Support for User Claims (Domain Controllers OU)
Enable the **"KDC support for claims, compound authentication, and Kerberos armoring"** Group Policy setting on the Domain Controllers OU. This is required for Kerberos to issue and transmit user/device claims.

![Enable KDC support for user claims on Domain Controllers OU](task3-enable-kdcsupportforuserclaims-on-domain-controllers-ou.png)

### 4. Edit PC (Device) Classification
Set device claim attribute values on the computer object(s) that will be evaluated as part of the access policy.

![Edit PC classification](task4-edit-pc-classification.png)

### 5. Enable DAC via Group Policy (FSRM/DAC Control Panel)
Configure the **"KDC support for claims"** and DAC-related settings, and review the Dynamic Access Control control panel to confirm the feature is enabled domain-wide.

![DAC control panel](task5-DAC-control-panal.png)

### 5b. Edit User Classification
Set user attribute values (e.g., Department, Location) on the test user account(s) in Active Directory — these become the source for user claims.

![Edit user classification](task5-edit-user-classification.png)

### 6. Add a Department Claim Type
In Active Directory Administrative Center, create a new **claim type** based on the user's `Department` attribute.

![Add Department claim type](task6-add-dept-claim-type.png)

### 7. Add a Location Claim Type
Create an additional **claim type** based on the user's `Location`/`Office` attribute.

![Add Location claim type](task7-add-location-claim-type.png)

### 8. Enable and Edit the Department Resource Property
Enable the built-in **Department** resource property and edit its suggested values so it can be used to classify files/folders and referenced in central access rules.

![Enable Department resource property and edit values](task8-enable-dept-resource-property-and-edit-values.png)

### 8b. Enable and Edit the Location Resource Property
Enable the **Location** resource property and edit its suggested values in the same way.

![Enable Location resource property and edit values](task8-enable-location-resource-property-and-edit-values.png)

### 9. Update the FSRM Classification Property
In **File Server Resource Manager**, update/refresh the classification property definitions so the newly enabled resource properties (Department, Location) are available for classifying folders.

![Update FSRM classification property](task9-update-fsrm-class-property.png)

### 10. Add Folder Attributes (Classification)
Apply classification values (Department, Location) directly to the target folder's properties — these are the resource attributes DAC will evaluate against user claims.

![Add folder attributes](task10-add-folder-attributes.png)

### 11. Create a Central Access Rule
Build a **Central Access Rule** that defines the conditional expression — matching user claims (Department/Location) against resource properties on the folder — to grant or deny access.

![Create Central Access Rule](task11-create-access-central-rule.png)

### 12. Create the DAC (Central Access) Policy
Bundle the central access rule into a **Central Access Policy** that can be published and applied via Group Policy.

![Create DAC Policy](task12-create-DAC-Policy.png)

### 13. Review the Central Access Policy for DAC
Verify the central access policy configuration and its associated rule(s) before deployment.

![Central Access Policy for DAC](task13-central-access-policy-DAC.png)

### 14. Apply the Policy on the Folder
Deploy the Central Access Policy to the file server via Group Policy, then apply/associate it to the target folder's security settings (alongside existing NTFS permissions).

![Apply policy on folder](task14-apply-policy-on-folder.png)

### 15. Verify with Effective Access
Use the **Effective Access** tab on the folder's Advanced Security Settings to confirm the test user is granted access as expected, validating that claims + resource properties + central access rule are all working together.

![Effective access accepted](task15-effective-access-acepted.png)

## Result

By combining NTFS/Share permissions with AD claims, FSRM resource classification, and a Central Access Policy, the folder now enforces **attribute-based access control**: a user is only granted access if their claims (e.g., Department = HR, Location = HQ) match the classification applied to the folder — demonstrating a working Dynamic Access Control implementation.

## Notes

- All screenshots referenced above should be placed in the same directory as this README for the images to render correctly on GitHub.
- This lab is part of a broader **100 Windows Server Admin Labs** collection.
