# Windows User Privileges
* Privileges in Windows are rights that an account can be granted to perform a variety of operations on the local system such as managing services, loading drivers, shutting down the system
* Different from access rights > system uses access rights to grant or deny access to securable objects
* User and group privileges are stored in a database and granted via an access token when a user logs on to a system
* An account can have different privileges on different systems if the account belongs to an AD domain

The goal of an assessment is often to gain administrative access to a system or multiple systems. Suppose we can log in to a system as a user with a specific set of privileges. In that case, we may be able to leverage this built-in functionality to escalate privileges directly or use the target account's assigned privileges to further our access in pursuit of our ultimate goal.

## Windows Authentication Process
* **Security Principals** is a unique entity that can be authenticated by the system and granted access to resouces
* Fundamental to the security architecture of Windows OS
* Enables the assignment of permissions and privileges to these entities
* Play a role in access control, allowing administrators to specify who can access or modify resources, such as files, directories, or system settings
* Every security principal is identified by a unique SID

![alt text](<../../Images/Windows Auth Process.png>)

* User attempts to access a securable object such as a folder on a file share
* User's access token (user SID, SIDs for any groups they are members of, privilege list, and other access information) is compared against **Access Control Entries (ACEs)** within the objects **Security Descriptor** (which contains security information about a securable object such as access rights)
* Once comparison is complete > a decision is made to either grant or deny access

As part of our enumeration and privilege escalation activities, we attempt to use and abuse access rights and leverage or insert ourselves into this authorization process to further our access towards our goal.

## Rights and Privileges in Windows - Groups
* Windows contains many groups that grant their members powerful rights and privileges
* Many of these groups can be abused to escalate privileges on both standalone Windows host and within an AD domain environment
* May be used to gain Domain Admin, local administrator, or SYSTEM privileges on a Windows workstation, server, or Domain Controller

| Group                     | Description                                                                                                                   |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| Default Administrators    | Domain Admins and Enterprise Admins are "super" groups.                                                                       |
| Server Operators          | Members can modify services, access SMB shares, and backup files.                                                             |
| Backup Operators          | Members are allowed to log onto DCs locally and should be considered Domain Admins. They can make shadow copies of the SAM/NTDS database, read the registry remotely, and access the file system on the DC via SMB. This group is sometimes added to the local Backup Operators group on non-DCs. |
| Print Operators           | Members can log on to DCs locally and "trick" Windows into loading a malicious driver.                                        |
| Hyper-V Administrators    | If there are virtual DCs, any virtualization admins, such as members of Hyper-V Administrators, should be considered Domain Admins.                                                         |
| Account Operators         | Members can modify non-protected accounts and groups in the domain.                                                            |
| Remote Desktop Users      | Members are not given any useful permissions by default but are often granted additional rights such as Allow Login Through Remote Desktop Services and can move laterally using the RDP protocol. |
| Remote Management Users   | Members can log on to DCs with PSRemoting (This group is sometimes added to the local remote management group on non-DCs).  |
| Group Policy Creator Owners | Members can create new GPOs but would need to be delegated additional permissions to link GPOs to a container such as a domain or OU.                                                                 |
| Schema Admins             | Members can modify the Active Directory schema structure and backdoor any to-be-created Group/GPO by adding a compromised account to the default object ACL.                                      |
| DNS Admins                | Members can load a DLL on a DC, but do not have the necessary permissions to restart the DNS server. They can load a malicious DLL and wait for a reboot as a persistence mechanism. Loading a DLL will often result in the service crashing. A more reliable way to exploit this group is to create a WPAD record.        |

## Rights and Privileges in Windows - Privileges
* Depending on group membership, and other factors such as privileges assigned via domain and local Group Policy, users can have various rights assigned to their account

[User Rights Assignment](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/user-rights-assignment)

| Setting Constant      | Setting Name                                        | Standard Assignment                   | Description                                                                                                                                                                                                                                                                                                                                   |
|-----------------------|-----------------------------------------------------|--------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SeNetworkLogonRight   | Access this computer from the network              | Administrators, Authenticated Users | Determines which users can connect to the device from the network. This is required by network protocols such as SMB, NetBIOS, CIFS, and COM+.                                                                                                                                                                                            |
| SeRemoteInteractiveLogonRight | Allow log on through Remote Desktop Services | Administrators, Remote Desktop Users | This policy setting determines which users or groups can access the login screen of a remote device through a Remote Desktop Services connection. A user can establish a Remote Desktop Services connection to a particular server but not be able to log on to the console of that same server.                                                     |
| SeBackupPrivilege     | Back up files and directories                     | Administrators                       | This user right determines which users can bypass file and directory, registry, and other persistent object permissions for the purposes of backing up the system.                                                                                                                                                                        |
| SeSecurityPrivilege   | Manage auditing and security log                   | Administrators                       | This policy setting determines which users can specify object access audit options for individual resources such as files, Active Directory objects, and registry keys. These objects specify their system access control lists (SACL). A user assigned this user right can also view and clear the Security log in Event Viewer.            |
| [SeTakeOwnershipPrivilege](SeTakeOwnershipPrivilege.md) | Take ownership of files or other objects       | Administrators                       | This policy setting determines which users can take ownership of any securable object in the device, including Active Directory objects, NTFS files and folders, printers, registry keys, services, processes, and threads.                                                                                                             |
| [SeDebugPrivilege](SeDebugPrivilege.md)      | Debug programs                                      | Administrators                       | This policy setting determines which users can attach to or open any process, even a process they do not own. Developers who are debugging their applications do not need this user right. Developers who are debugging new system components need this user right. This user right provides access to sensitive and critical operating system components. |
| [SeImpersonatePrivilege](SeImpersonatePrivilege.md) | Impersonate a client after authentication       | Administrators, Local Service, Network Service, Service | This policy setting determines which programs are allowed to impersonate a user or another specified account and act on behalf of the user.                                                                                                                                                                                                |
| SeLoadDriverPrivilege | Load and unload device drivers                     | Administrators                       | This policy setting determines which users can dynamically load and unload device drivers. This user right is not required if a signed driver for the new hardware already exists in the driver.cab file on the device. Device drivers run as highly privileged code.                                                                         |
| SeRestorePrivilege    | Restore files and directories                     | Administrators                       | This security setting determines which users can bypass file, directory, registry, and other persistent object permissions when they restore backed up files and directories. It determines which users can set valid security principals as the owner of an object.                                                                               |

