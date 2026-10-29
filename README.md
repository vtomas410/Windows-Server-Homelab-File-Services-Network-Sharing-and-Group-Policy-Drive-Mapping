# **Windows Server Homelab: File Services, Network Sharing, and Group Policy Drive Mapping**

This project covers creating shared folders, managing user permissions, mapping network drives on client computers, automating drive mapping through Group Policy, and managing storage with File Server Resource Manager (FSRM).The goal was to simulate common IT support and system administration tasks performed in a business environment.

## **Objectives**

- Configure shared folders on Windows Server.
- Understand the difference between NTFS and share permissions.
- Provide authorized domain users access to network resources.
- Map network drives on Windows client machines.
- Automate drive mapping using Group Policy Objects (GPOs).
- Configure storage quotas and file screening with FSRM.
- Troubleshoot network share and Group Policy issues.

**Prerequisites:** A functioning Active Directory domain, a domain-joined client machine, administrative access to the server, and network connectivity between the client and server.

---

# **Understanding File Sharing and Permissions**

File sharing allows multiple authorized users to access files stored centrally on a server. In a business environment, this makes it easier to collaborate, manage data, and control access to organizational resources.

NTFS Permissions vs. Share Permissions

- NTFS permissions: control access to files and folders on an NTFS-formatted volume. They can apply to individual files, folders, and subfolders.
- Share permissions: control access when a folder is accessed through a network share.

Common permission levels include:

- **Read:** View and open files.
- **Write:** Create or modify files, subject to other permissions.
- **Modify:** Read, create, modify, and delete files.
- **Full Control:** Includes permission management and other administrative actions.

When accessing a folder through a network share, the effective access is determined by both share permissions and NTFS permissions. In general, the more restrictive applicable permission limits the user's access.

---

# Activity 1: Set up a File Sharing

Configure File Sharing

- Create Shared folders with appropriate permissions.
- Set NFTS and share permissions to allow domain users access

Step 1 : Open File explorer → This PC → Local disk (c:) → Create folder → name it “Shared”

<img width="1056" height="781" alt="1 Create new folder" src="https://github.com/user-attachments/assets/2f6921f5-cd2d-4f83-ac1a-9845fb2d0358" />


Step 2: Right click Shared folder → click properties → Open the Sharing tab → Select Advanced Sharing

<img width="551" height="614" alt="2 Advance Sharing" src="https://github.com/user-attachments/assets/fc031555-4950-44a0-a96b-cfb8360e39f6" />


Step 3: Enable Share this folder

<img width="405" height="573" alt="3 Share this folder" src="https://github.com/user-attachments/assets/1c28fa0f-c195-440b-9eaa-868126d90a98" />


Step 4: Specify a share name, such as Shared.

Step 5: Open the share permissions and configure the intended users or groups.

<img width="907" height="698" alt="4 Share permissions with users,accounts, computers, and etc" src="https://github.com/user-attachments/assets/d80d872a-aa8f-4cc4-8cbe-e68abfb7ac12" />


<img width="1050" height="769" alt="5 Select Domain Users" src="https://github.com/user-attachments/assets/4682e339-7fbe-4819-956f-37c795d83908" />

<img width="931" height="662" alt="6 Select Read and apply to all domain users" src="https://github.com/user-attachments/assets/b7be5d86-7357-4d4d-b4c4-d9a68c4fa629" />


Step 6:  Open the Security tab and configure the corresponding NTFS permissions.

Step 7: Apply the changes.

---

# Activity 2: Set up Client Machines (Mapping sharing)

Access Share Recourses

- Log in to the client machines using domain user accounts
- Map network drives to access shared folders

In this activity we are going to create a user to see if the folder is accessible.

Step 1: Go to windows client → Open file explorer → Right click and click Map network drives.

<img width="1249" height="883" alt="7 Log into the Client User" src="https://github.com/user-attachments/assets/c6137ec4-6d87-4092-a3b4-ccdc75e04c9f" />


<img width="973" height="687" alt="8 Folder - PC - Right Click - Map network Drive" src="https://github.com/user-attachments/assets/a73864b4-47c5-44bf-93e0-a84f3d85fcda" />


Step 2: Enter the UNC path in the address bar:`\\SERVERNAME\Shared`

Easiest way to find this, open command prompt and type hostname to get the name of server. 

<img width="1018" height="374" alt="9 Research Hostname from server" src="https://github.com/user-attachments/assets/b3893c28-d5a3-4154-b261-d270a7e9576d" />


<img width="868" height="620" alt="10 Enter Server Mapping" src="https://github.com/user-attachments/assets/19b3b1b6-e933-4849-9756-e4f566f859eb" />


Step 3: Press Enter → Verify that the shared folder opens.

<img width="518" height="308" alt="11 Verify Network Locations Mapping" src="https://github.com/user-attachments/assets/78cbf247-ff45-4e2b-a44e-5c18ed33d884" />


Step 4: Test whether the user can perform only the actions permitted by the assigned permissions.

**Trouble shooting:** I had trouble connecting folder. It didn’t connect at first because I needed to have both the Server and Client machine on at the same time. I didn’t have enough RAM capacity. The best way to resolve this was by buying more RAM to install into my home lab desktop.

---

# Activity 3: GPO Configuration (Network Sharing)

A mapped network drive makes a shared folder accessible through a drive letter in File Explorer. This is more persistent and efficient.

- Configure Group Policy Objects (GPOs) to automatically map network drives for users.

Step 1: Sign in to the domain controller or an administrative workstation.

Step 2: Open Group Policy Management.

Step 3: Locate the domain and the Organizational Unit (OU) containing the target users.

Step 4: Create a new Group Policy Object named: Map Network Drives

<img width="882" height="564" alt="12 Creating New GPO - Mapped Drives" src="https://github.com/user-attachments/assets/4184e093-9842-48ee-8d54-bd1e057f6f34" />


Step 5: Edit the new GPO.

Step 6: Navigate to: User Configuration → Preference → Windows Settings → Drive Maps

<img width="834" height="632" alt="13 User configuration -New Drive Properties" src="https://github.com/user-attachments/assets/17bf4c13-b231-46b9-a5db-83a6930af9ab" />


Step 7: Right-click Drive Maps and select New > Mapped Drive

Step 8: Configure the mapping:

- Action: Update
- Location: \\servername\shared
- Drive Letter: S:
- Label: Shared Drive
- Reconnect**:** Enabled, if appropriate

<img width="419" height="475" alt="14 Mapping Location - Click OK" src="https://github.com/user-attachments/assets/7aa2d8cd-0ea4-44e4-b218-4d17e1ce567b" />


Step 9: Save the configuration.

Step 10: Link the GPO to the OU containing the **User** accounts.

<img width="408" height="156" alt="15 Apply to Users" src="https://github.com/user-attachments/assets/3dfb1da7-3140-44a4-9525-ce32412c0ccb" />


# Why User Configuration?

Drive mapping is usually a user-specific setting because it determines which network resources a user sees when signing in. Therefore, the Drive Maps preference is configured under User Configuration.

The GPO must be linked to an OU within the applicable domain structure, and the target user must be within its scope.

# Verify and Test GPO

Step 1:  On the client machine, open Command Prompt and run: 

“Cmd gpupdate /force” 

<img width="467" height="170" alt="16 Gpupdate force" src="https://github.com/user-attachments/assets/53cf87c9-43fe-47a3-8d87-5b05c28a2fa9" />


Step 2: Sign out and back in. Some user policy changes require a new sign-in before the result becomes visible.

Step 3: Successful Test.

<img width="371" height="664" alt="17 Verified after restart " src="https://github.com/user-attachments/assets/f6e902c3-3741-4b43-b972-47e49a2aeda8" />


Check the user's applied Group Policy Objects and confirm that  Map Network Drives is listed.

Open File Explorer and verify that the S: drive appears and opens successfully.

The network drive is mapped automatically for users within the GPO's scope, without requiring them to configure the drive manually.

---

# Activity 4: Implement quotas and file screening using File Server Resource Manager (FSRM).

Configure FRSM to create quota template and file screen template to effectively manage file storage in your organization.

**What is FSRM?**

File Server Resource Manager.

A suite of tools provided by Microsoft to help you manage and classify data stored on file servers.

Usually there is max to most companies storage file saving.

---

Step 1:  Open Server Manager

Step 2:  Select Manage → Add Roles and Features.

Step 3:  Proceed through the installation wizard.

Step 4: On server roles → File and Storage Services → Locate File Server Resource Manager → Select the feature and complete the installation.

<img width="846" height="597" alt="18 Select File Server Resource Manager" src="https://github.com/user-attachments/assets/b7a4f2d5-eb6b-4104-9fd9-f54e25d0b610" />


<img width="807" height="587" alt="19 Installing The Feature" src="https://github.com/user-attachments/assets/1102870e-5737-49bb-aade-12831e79fbfb" />


---

# Configuring Storage Quotas

Quotas allow administrators to limit how much data can be stored in a folder or on a volume.

For example, a test quota could restrict a lab folder to 10 GB. This is an example configuration, not a requirement for every environment.

**Configuration Steps:**

Step 1. Open File Server Resource Manager.
Step 2. Navigate to Quota Management > Quotas.
Step 3. Select Create Quota.

<img width="1056" height="914" alt="20 Creating Quota" src="https://github.com/user-attachments/assets/523d8b9d-fc7f-4590-8872-4ecdbadc3071" />


Step 4. Specify the folder to which the quota applies.
Step 5. Choose a quota template or configure a custom limit.

<img width="490" height="608" alt="21 setting quota limit" src="https://github.com/user-attachments/assets/b827ad04-92a3-431b-a207-3b25454510aa" />


Step 6. Set the maximum storage capacity.
Step 7. Configure threshold notifications if required.

<img width="535" height="653" alt="22 setting usage reaches " src="https://github.com/user-attachments/assets/d2067594-16e7-45b7-ba45-201b9e7baab9" />


Step 8. Save the quota.
Step 9. Verify that the quota appears in the management console.

**Quota Thresholds**

Administrators can configure thresholds such as 80% of the quota to identify growing storage usage before the limit is reached.

---

# Configuring File Screening

File screening allows administrators to control the types of files that users can store in designated folders.

For example, an organization might restrict video files or executable files in a general document share to reduce unnecessary storage consumption or enforce security policies.

Configuration Steps:

Step 1. Open File Server Resource Manager.
Step 2. Navigate to File Screening Management > File Screens.
Step 3. Select Create File Screen.

<img width="939" height="703" alt="23 File Screening " src="https://github.com/user-attachments/assets/c2922d5c-6267-4056-81ae-f1e96097a5e3" />


Step 4. Specify the target folder.
Step 5. Choose a file-screening template or create a custom configuration.
Step 6. Configure whether the screen is active or passive.

<img width="1259" height="934" alt="24 Created a file screen " src="https://github.com/user-attachments/assets/280c7ea6-2acc-412d-acc4-0e31c13701d8" />


Step 7. Select the file groups to monitor or restrict.
Step 8. Apply the configuration.
Step 9. Test the result using a file type included in the selected file group.

**Active vs. Passive Screening**

- Active screening: Blocks files that match the configured file groups.
- Passive screening: Allows the file operation but logs or reports matching files.

**Verify and Test.**

Attempt to save a test file whose extension belongs to a restricted file group.

For an active screen, the file operation should be blocked if the file matches the configured rules. For passive screening, the file may be saved while the event is logged.

Test only in the lab and confirm that the chosen file group includes the extension being tested.
