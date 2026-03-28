# Oski — CyberDefenders
## Category
Threat Intel

## Scenario
The accountant at the company received an email titled "Urgent New Order" 
from a client late in the afternoon. When he attempted to access the attached 
invoice, he discovered it contained false order information. Subsequently, 
the SIEM solution generated an alert regarding downloading a potentially 
malicious file. Upon initial investigation, it was found that the PPT file 
might be responsible for this download. Could you please conduct a detailed 
examination of this file?

![hash](screenshots/1.jpeg)

## Tools Used
- VirusTotal (site)
- Any.run (site)

## Walkthrough

### Q1 — What was the time of malware creation?
**What I did:** I visited VirusTotal and submitted the given hash. 
To find more details about the file, I navigated to the Details tab.

![virustotal](screenshots/2.jpeg)

**What I found:** In the Details tab, I found the History section 
which contained the creation date.

**Answer:** 2022-09-28 17:40

![creation](screenshots/3.jpeg)


### Q2 — Which C2 server does the malware in the PPT file communicate with?
**What I did:** Since C2 communication is a network behavior, 
I checked the Behavior tab in VirusTotal.

**What I found:** In that tab, I identified the server 
the malware was communicating with.

![post](screenshots/5.jpeg)

**Answer:** POST http://171.22.28.221/5c06c05b7b34e8e6.php

**Note:** Since the malware runs on the local machine and needs 
to send data to the attacker, the most logical method is POST, 
as it allows the malware to send collected data to the C2 server.


### Q3 — What is the first library that the malware requests post-infection?
**What I did:** I stayed in the same Behavior tab, since downloading 
files to the machine is also a malware behavior.

**What I found:** In that tab, more specifically in the Files Dropped 
section, I found the first file downloaded by the malware.

![file](screenshots/6.jpeg)

**Answer:** sqlite3.dll


### Q4 — What RC4 key is used by the malware to decrypt its 
base64-encoded string?
**What I did:** The link provided by CyberDefenders redirected me 
to the Any.run report of the malware.

**What I found:** In the Stealc section of the report, 
I found the decryption key.

![key](screenshots/7.jpeg)

**Answer:** 5329514621441247975720749009


### Q5 — What is the main MITRE technique the malware uses 
to steal the user's password?
**What I did:** The link redirected me to the main page of the 
Any.run report. To see the MITRE ATT&CK techniques, I clicked 
on the ATT&CK button located under the indicators and trackers section.

**What I found:** I found all the techniques used during this attack. 
One of them was "Credentials from Password Stores".

![technique](screenshots/8.jpeg)

**Answer:** T1555


### Q6 — Which directory does the malware target for the deletion 
of all DLL files?
**What I did:** On the main page of the report, I found a small 
process tree. The command responsible for deleting the malware's 
traces was executed by cmd.exe, which is a child process of VPN.exe.

**What I found:** I found a command executed by cmd.exe 
that targeted a specific directory for deletion.

![cmd](screenshots/9.jpeg)

**Answer:** C:\ProgramData


### Q7 — How many seconds does it take for the malware to self-delete 
after exfiltrating the user's data?
**What I did:** To find the delay before the malware deletes itself, 
I followed the process tree and clicked on the timeout process, 
then on "more info".

**What I found:** I found a timeline view showing the exact duration.

![timeline](screenshots/10.jpeg)

**Answer:** 5


## Incident Timeline
- t=0 ms : the malicious file is executed after a successful phishing attempt
- t=31 ms : the malware checks which languages are supported on the machine
- t=343 ms : the malware reads the machine name
- t=359 ms : the malware reads proxy server information from internet settings
- t=437 ms : the malware reads the machine GUID from the registry
- t=750 ms : the malware reads environment variables
- t=750 ms : the malware reads CPU information
- t=750 ms : the malware reads the Windows version of the local machine
- t=750 ms : the malware searches for installed software Adobe Flash Player 32 ActiveX
- t=1703 ms : creation of the file 
  <C:\Users\admin\AppData\Local\Google\Chrome\User Data\Local State>
- t=1781 ms : creation of the file 
  <C:\ProgramData\GCBGCAFIIECBFIDHIJKFBAKEGD>
- t=3593 ms : the file <C:\ProgramData\GDHIIDAFIDGCFHJJDGDA> 
  is filled with stolen credentials
- t=4289 ms : VPN.exe sends a POST request to 
  <http://171.22.28.221/5c06c05b7b34e8e6.php>
- t=4290 ms : Stealc C2 communication is detected
- t=4309 ms : external connection established by VPN.exe — 
  IpDst:171.22.28.221, IpSrc:192.168.100.121, PortDst:80, PortSrc:49175
- t=4327 ms : VPN.exe sends a GET request to download 
  <http://171.22.28.221/9e226a84ec50246d/sqlite3.dll>
- t=10359 ms : Stealc is detected in RAM — because it runs entirely 
  in memory, it cannot be detected by antivirus tools that only scan files
- t=26468 ms : creation of the file 
  <C:\Users\admin\AppData\Roaming\Moonchild Productions\Pale Moon\profiles.ini>
- t=27723 ms : download of <C:\ProgramData\mozglue.dll> — 
  this DLL is used to decrypt passwords saved by Firefox
- t=29031 ms : cmd.exe executes the cleanup command:
  "C:\Windows\system32\cmd.exe" /c timeout /t 5 
  & del /f /q "C:\Users\admin\AppData\Local\Temp\VPN.exe" 
  & del "C:\ProgramData\*.dll" & exit


## What I Learned
- VPN.exe contained the Stealc malware, which works by collecting 
  credentials from the machine and then deleting itself to remove any traces
- Stealc was developed by a Russian-speaking threat actor and sold 
  as a subscription-based service, with the only goal of stealing 
  personal information from victims
- Some malware runs entirely in RAM to avoid detection by antivirus software, 
  since most antivirus tools scan files on disk
- Firefox encrypts saved passwords using its NSS crypto library. 
  Stealc drops mozglue.dll — a required dependency — to load 
  those decryption functions and extract passwords in plaintext
