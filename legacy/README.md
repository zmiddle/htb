# Legacy Walkthrough

## Enumeration
Nmap:
'''
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
'''
