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

### Default Administrators
* Domain Admins and Enterprise Admins are "super" groups

### Server Operators
* Members can modify services, access SMB shares, and backup files

### Backup Operators
* Members are allowed to log onto DCs locally and should be considered Domain Admins
* Can make shadow copies of the SAM/NTDS database, read the registry remotely, and access the file system on the DC via SMB

### Print Operators
* Members can log on to DCs locally and "trick" Windows into loading a malicious driver

### Account Operators
* Members can modify non-protected accounts and groups in the domain

### Remote Desktop Users
* Members are not given any useful permissions by default but are often granted additional rights such as Allow Login Through Remote Desktop Services
* Can move laterally using RDP protocol

### Remote Management Users
* Members can log on to DCs with PSRemoting
* This group is sometimes added to the local remote management group on non-DCs

### Group Policy Creator Owners
* Members can create new GPOs 
* Would need to be delegated additional permissions to link GPOs to a container such as a domain or OU

### Schema Admins
* Members can modify the Active Directory schema structure
* Can backdoor any to-be-created Group/GPO by adding a compromised account to the default object ACL

### DNS Admins
* Members can load a DLL on a DC, but do not have the necessary permissions to restart the DNS server
* Can load a malicious DLL and wait for a reboot as a persistence mechanism