# Antivirus Evasion
In an attempt to compromise a target machine, attackers often disable or otherwise bypass antivirus software installed on these systems

### What is Antivirus Software
Antivirus (AV) is type of application designed to prevent, detect, and remove malicious software

### Signature-Based Detection
An antivirus signature is a continuous sequence of bytes within malware that uniquely identifies it. Signature-based antivirus detection is mostly 
considered a blacklist technology. In other words, the filesystem is scanned for known malware signatures and if any are detected, the offending files are quarantined.

### Heuristic and Behavioral-Based Detection
is a detection method that relies on various rules and algorithms to determine whether or not an action is considered malicious. 
This is often achieved by stepping through the instruction set of a binary file or by attempting to decompile and then analyze the source code

## Bypassing Antivirus Detection

### On-Disk Evasion
First look at various techniques used to obfuscate files stored on a physical disk.

#### Packer
One of the earliest ways of avoiding detection involved the use of packers. Given the high cost of disk space and slow network speeds
during the early days of the Internet, packers were originally designed to simply reduce the size of an executable

#### Obfuscators
Obfuscators reorganize and mutate code in a way that makes it more difficult to reverse-engineer. This includes replacing instructions with semantically 
equivalent ones, inserting irrelevant instructions or "dead code", splitting or reordering functions

#### Crypters
software cryptographically alters executable code, adding a decrypting stub that restores the original code upon execution. 
This decryption happens in-memory, leaving only the encrypted code on-disk.

### In-Memory Evasion
also known as PE Injection is a popular technique used to bypass antivirus products. Rather than obfuscating a malicious binary, creating new sections, 
or changing existing permissions, this technique instead focuses on the manipulation of volatile memory

#### Remote Process Memory Injection
This technique attempts to inject the payload into another valid PE that is not malicious. The most common method of doing this is by leveraging a set of Windows APls. 
First, we would use the OpenProcess function to obtain a valid HANDLE to a target process that we have permissions to access. 
After obtaining the HANDLE, we would allocate memory in the context of that process by calling a Windows API such as Virtua/AllocEx.
Once the memory has been allocated in the remote process, we would copy the malicious payload to the newly allocated memory using WriteProcessMemory. 
After the payload has been successfully copied, it is usually executed in memory in a separate thread using the CreateRemote Thread API

#### Reflective DLL Injection
Unlike regular DLL injection, which implies loading a malicious DLL from disk using the LoadLibrary API, this technique attempts to load a DLL stored by the attacker 
in the process memory.

#### Process Hollowing
attackers first launch a nonmalicious process in a suspended state. Once launched, the image of the process is removed from memory and replaced with a malicious executable image. Finally, the process is then resumed and
malicious code is executed instead of the legitimate process

#### lnline hooking
This technique involves modifying memory and introducing a hook (instructions that redirect the code execution) into a function to point the execution flow to our
malicious code. Upon executing our malicious code, the flow will return back to the modified function and resume execution, appearing as if only the original code had executed
