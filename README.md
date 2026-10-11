# Windows Server Homelab: File Services, Network Sharing, and Group Policy Drive Mapping

## Project Overview

This project demonstrates how to configure centralized file sharing in a Windows Server environment integrated with Active Directory Domain Services (AD DS). It covers creating shared folders, managing user permissions, mapping network drives on client computers, automating drive mapping through Group Policy, and managing storage with File Server Resource Manager (FSRM).

The goal was to simulate common IT support and system administration tasks performed in a business environment.

## Objectives

- Configure shared folders on Windows Server.
- Understand the difference between NTFS and share permissions.
- Provide authorized domain users access to network resources.
- Map network drives on Windows client machines.
- Automate drive mapping using Group Policy Objects (GPOs).
- Configure storage quotas and file screening with FSRM.
- Troubleshoot network share and Group Policy issues.

## Lab Environment

| Component | Purpose |
|---|---|
| Windows Server | Hosts shared folders and manages file access |
| Active Directory Domain Services | Centralized user and computer authentication |
| Group Policy Management | Automates user configurations |
| Windows 10/11 Pro or Enterprise client | Tests access to shared resources |
| File Server Resource Manager | Controls storage consumption and file types |

**Prerequisites:** A functioning Active Directory domain, a domain-joined client machine, administrative access to the server, and network connectivity between the client and server.

*Note: Replace example server names, domain names, usernames, and folder paths with the actual values used in your lab.*

---

## 1. Understanding File Sharing and Permissions

File sharing allows multiple authorized users to access files stored centrally on a server. In a business environment, this makes it easier to collaborate, manage data, and control access to organizational resources.

### NTFS Permissions vs. Share Permissions

**NTFS permissions** control access to files and folders on an NTFS-formatted volume. They can apply to individual files, folders, and subfolders.

**Share permissions** control access when a folder is accessed through a network share.

Common permission levels include:

- **Read:** View and open files.
- **Write:** Create or modify files, subject to other permissions.
- **Modify:** Read, create, modify, and delete files.
- **Full Control:** Includes permission management and other administrative actions.

When accessing a folder through a network share, the effective access is determined by both share permissions and NTFS permissions. In general, the more restrictive applicable permission limits the user's access.

**Security best practice:** Grant permissions to appropriate Active Directory security groups rather than assigning access individually to many users. Follow the principle of least privilege.

## 2. Creating a Shared Folder

The first task was to create a folder on the Windows Server that domain users could access over the network.

### Steps

1. Sign in to Windows Server using an administrative account.
2. Open File Explorer.
3. Create a folder, for example:

   `C:\Shared`

4. Right-click the folder and select **Properties**.
5. Open the **Sharing** tab.
6. Select **Advanced Sharing**.
7. Enable **Share this folder**.
8. Specify a share name, such as `Shared`.
9. Open the share permissions and configure the intended users or groups.
10. Open the **Security** tab and configure the corresponding NTFS permissions.
11. Apply the changes.

The resulting network path should resemble:

`\\SERVERNAME\Shared`

Replace `SERVERNAME` with the actual server hostname.

### Validation

Confirm that the folder is shared and that the intended domain users have appropriate access. Test with a standard domain account rather than relying exclusively on an administrator account.

## 3. Accessing the Share from a Client Machine

After configuring the share, the next step was to verify that a client computer could access the resource.

### Steps

1. Sign in to the Windows client using a domain account.
2. Open File Explorer.
3. Enter the UNC path in the address bar:

   `\\SERVERNAME\Shared`

4. Press Enter.
5. Verify that the shared folder opens.
6. Test whether the user can perform only the actions permitted by the assigned permissions.

### Validation

A successful connection confirms that the client can reach the share and authenticate with sufficient permissions.

If access fails, verify the server name, network connectivity, DNS resolution, firewall configuration, share permissions, and NTFS permissions.

## 4. Mapping a Network Drive

A mapped network drive makes a shared folder accessible through a drive letter in File Explorer.

For example, the shared folder can be mapped to drive `S:`.

### Manual Mapping Steps

1. Open File Explorer on the client machine.
2. Select **This PC**.
3. Choose **Map network drive**.
4. Select the drive letter `S:`.
5. Enter the folder path:

   `\\SERVERNAME\Shared`

6. Select **Reconnect at sign-in** if the connection should be restored at login.
7. Select **Finish**.
8. Confirm that the drive appears under This PC.

### Validation

Open the mapped drive and test access to the shared files.

Manual mapping is useful for initial testing, but centrally managed environments benefit from Group Policy because administrators can automatically configure drive mappings for users.

## 5. Automating Network Drive Mapping with Group Policy

Group Policy can automatically map network drives when users sign in. This reduces manual configuration and helps ensure consistent access across domain computers.

### Creating the GPO

1. Sign in to the domain controller or an administrative workstation.
2. Open **Group Policy Management**.
3. Locate the domain and the Organizational Unit (OU) containing the target users.
4. Create a new Group Policy Object named:

   `Map Network Drives`

5. Edit the new GPO.
6. Navigate to:

   `User Configuration > Preferences > Windows Settings > Drive Maps`

7. Right-click **Drive Maps** and select **New > Mapped Drive**.
8. Configure the mapping:
   - **Action:** Update
   - **Location:** `\\SERVERNAME\Shared`
   - **Drive Letter:** `S:`
   - **Label:** `Shared Drive`
   - **Reconnect:** Enabled, if appropriate
9. Save the configuration.
10. Link the GPO to the OU containing the intended user accounts.

Use the actual server name, share path, and OU structure from your environment.

### Why User Configuration?

Drive mapping is usually a user-specific setting because it determines which network resources a user sees when signing in. Therefore, the Drive Maps preference is configured under User Configuration.

The GPO must be linked to an OU within the applicable domain structure, and the target user must be within its scope.

### Applying and Testing the GPO

On the client machine, open Command Prompt and run:

```cmd
gpupdate /force
```

Then sign out and back in if necessary. Some user policy changes require a new sign-in before the result becomes visible.

Verify the applied policy with:

```cmd
gpresult /r
```

Check the user's applied Group Policy Objects and confirm that `Map Network Drives` is listed.

Open File Explorer and verify that the `S:` drive appears and opens successfully.

### Expected Result

The network drive is mapped automatically for users within the GPO's scope, without requiring them to configure the drive manually.

## 6. Installing File Server Resource Manager (FSRM)

File Server Resource Manager is a Windows Server feature that helps administrators manage storage on file servers. It supports quota management, file screening, and storage reporting.

These controls help prevent shared folders from consuming excessive storage and help enforce organizational file-storage policies.

### Installation Steps

1. Open Server Manager.
2. Select **Manage > Add Roles and Features**.
3. Proceed through the installation wizard.
4. Expand **File and Storage Services**.
5. Locate **File Server Resource Manager** under the available role services.
6. Select the feature and complete the installation.
7. Open File Server Resource Manager from Server Manager or the administrative tools.

Menu names can vary slightly depending on the Windows Server version.

## 7. Configuring Storage Quotas

Quotas allow administrators to limit how much data can be stored in a folder or on a volume.

For example, a test quota could restrict a lab folder to 10 GB. This is an example configuration, not a requirement for every environment.

### Configuration Steps

1. Open File Server Resource Manager.
2. Navigate to **Quota Management > Quotas**.
3. Select **Create Quota**.
4. Specify the folder to which the quota applies.
5. Choose a quota template or configure a custom limit.
6. Set the maximum storage capacity.
7. Configure threshold notifications if required.
8. Save the quota.
9. Verify that the quota appears in the management console.

### Quota Thresholds

Administrators can configure thresholds such as 80% of the quota to identify growing storage usage before the limit is reached.

Depending on the configuration, thresholds can trigger notifications or generate events.

**Important distinction:** A hard quota can prevent additional data from being stored once the limit is reached, while a soft quota is primarily used for monitoring and reporting. Confirm the selected quota type before testing.

## 8. Configuring File Screening

File screening allows administrators to control the types of files that users can store in designated folders.

For example, an organization might restrict video files or executable files in a general document share to reduce unnecessary storage consumption or enforce security policies.

### Configuration Steps

1. Open File Server Resource Manager.
2. Navigate to **File Screening Management > File Screens**.
3. Select **Create File Screen**.
4. Specify the target folder.
5. Choose a file-screening template or create a custom configuration.
6. Configure whether the screen is active or passive.
7. Select the file groups to monitor or restrict.
8. Apply the configuration.
9. Test the result using a file type included in the selected file group.

### Active vs. Passive Screening

- **Active screening:** Blocks files that match the configured file groups.
- **Passive screening:** Allows the file operation but logs or reports matching files.

### Validation

Attempt to save a test file whose extension belongs to a restricted file group.

For an active screen, the file operation should be blocked if the file matches the configured rules. For passive screening, the file may be saved while the event is logged.

Test only in the lab and confirm that the chosen file group includes the extension being tested.

---

## 9. Troubleshooting

During a file services lab, several configuration issues can prevent mapped drives from appearing or users from accessing shared folders.

### Problem: The shared folder cannot be opened

**Possible causes:**
- Incorrect server hostname or UNC path.
- DNS resolution problems.
- Client and server network connectivity issues.
- Missing share or NTFS permissions.
- Firewall rules preventing SMB traffic.

**Troubleshooting steps:**

```cmd
ping SERVERNAME
nslookup SERVERNAME
```

Then test the UNC path directly in File Explorer. A successful ping alone does not prove that file sharing is working.

### Problem: The mapped drive does not appear

**Possible causes:**
- The GPO is linked to the wrong OU.
- The user is outside the GPO's scope.
- The GPO is disabled or filtered by security permissions.
- The drive mapping contains an incorrect path or drive letter.
- The user policy has not refreshed or the user has not signed in again.

**Troubleshooting steps:**

```cmd
gpupdate /force
gpresult /r
```

Next, verify the GPO link, the user's OU, the Drive Maps preference, and the target share's accessibility.

If necessary, review the Group Policy operational log in Event Viewer for additional details.

### Problem: The user receives an Access Denied error

**Possible causes:**
- Incorrect NTFS permissions.
- Incorrect share permissions.
- The user is not a member of the intended security group.
- Group membership changes have not been reflected in the current logon session.

**Troubleshooting steps:**

1. Review the folder's Sharing permissions.
2. Review the folder's Security permissions.
3. Verify the user's Active Directory group membership.
4. Sign out and back in after group membership changes if necessary.
5. Test again using the intended standard user account.

### Problem: A file is not blocked by FSRM

**Possible causes:**
- The file extension is not included in the configured file group.
- The screen is passive rather than active.
- The screen is applied to a different folder.
- The file was created before the screen was configured.

**Troubleshooting steps:**

1. Confirm the file-screen path.
2. Review the configured file groups.
3. Verify the active or passive setting.
4. Test with a new file that matches a restricted extension.

---

## 10. Skills Demonstrated

This project demonstrates practical experience with:

- Windows Server administration.
- Active Directory and domain-user authentication.
- SMB network file sharing.
- NTFS and share permission management.
- UNC paths and mapped network drives.
- Group Policy Management and Drive Maps preferences.
- GPO scope and policy troubleshooting.
- File Server Resource Manager.
- Storage quota configuration.
- File screening and storage policy enforcement.
- Basic network and access-control troubleshooting.

## 11. Business Relevance

In a real IT environment, file servers provide a centralized location for company documents, departmental resources, and shared operational data.

The techniques demonstrated in this project help IT support technicians and system administrators:

- Provision access to shared resources.
- Apply consistent drive mappings across users.
- Troubleshoot file access and authentication issues.
- Limit storage consumption.
- Enforce file-storage policies.
- Reduce repetitive workstation configuration.

These are relevant responsibilities for help desk, desktop support, Windows system administration, and junior infrastructure roles.

## 12. Future Improvements

Possible extensions to this lab include:

- Creating department-specific shares for HR, Finance, and IT.
- Using Active Directory security groups to control access.
- Applying least-privilege permissions to each department.
- Testing access with multiple standard domain accounts.
- Configuring auditing for file access.
- Creating storage usage reports.
- Documenting a repeatable troubleshooting workflow.
- Capturing screenshots of successful configuration and testing.

## Conclusion

This project demonstrates how to configure and manage shared network resources in a Windows Server domain environment. By combining folder permissions, network drive mapping, Group Policy automation, and FSRM, administrators can provide users with consistent access while maintaining control over shared storage.

The lab provides a foundation for more advanced Windows administration and IT support projects.

## References

- **Original tutorial:** [File Services Homelab: Setting up Network Sharing on Windows Server (Ep. 4)](https://www.youtube.com/watch?v=OBnuOOWdEmc)

---

**Portfolio note:** Add screenshots of your actual server configuration, the shared folder permissions, the applied Group Policy, the mapped drive on the client, and your FSRM settings. Include a short explanation of what each screenshot proves. Document the settings you personally implemented and tested, and identify any sections that remain planned or untested.
