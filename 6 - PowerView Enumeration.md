# Powerview - Windows enumeration

## Tool
[https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

**Running powerview script process**
```
powershell -ep bypass
. .\PowerView.ps1
```

**Get Users informations**
```
Get-NetUser

Get-NetUser | select <specific_field>

Get-NetUser -UserName <username>
```

**Get Group information**
```
Get-NetGroup

Get-NetGroup -AdminCount => Return Adminins groups

Get-NetGroup -GroupName "<group Name>

Get-NetGroup -UserName <username>

Get-NetGroup -Domain <Domain.name> => Only information about this particular domain
```

**Get domian computers information**
```
Get-NetComputer

Get-NetComputer -Ping

Get-NetComputer -fulldata
```

**Getting Domain information**
```
Get-NetDomain

Get-NetDomainSID

Get-NetDomainPolicy

Get-NetDomainController
```

**Check to which computer we have acces as admistrator**
```
Find-LocalAdminAccess
```

**Get a list of local admins**
```
Invoke-EnumerateLocalAdmin
```

**Get list of Shared Folders**
```
Invoke-ShareFinder
```

**Enumerate active sessions**
```
Get-NetLoggedon -ComputeName <computer_name>
```

**Enumerate last logon**
```
Get-LastLoggedon -ComputeName <computer_name>
```

**Enumerate RDP sessions**
```
Get-NetRDPSession -ComputeName <computer_name>
```

**Get Group Policy Information**
```
Get-NetGPO

Get-NetGPO | select displayname, whenchanged
```
