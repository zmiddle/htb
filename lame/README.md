# Lame Walkthrough

Initial nmap scan... Apache web server on port 80 is interesting...
```
# Nmap 7.93 scan initiated Sat Jun  3 13:20:27 2023 as: nmap -Pn -n -sVC -T4 -p- -v -oN initial 10.10.10.68
Nmap scan report for 10.10.10.68
Host is up (0.083s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 6AA5034A553DFA77C3B2C7B4C26CF870
|_http-title: Arrexel's Development Site
|_http-server-header: Apache/2.4.18 (Ubuntu)

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Jun  3 13:20:56 2023 -- 1 IP address (1 host up) scanned in 29.20 seconds
```

[<img height="600" width="600px" src="https://github.com/zmiddle/htb/blob/main/lame/lame_pwned.png" />](https://www.hackthebox.com/achievement/machine/259690/1) 

