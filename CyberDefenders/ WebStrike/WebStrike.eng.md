# Oski — CyberDefenders

## Category
Threat Intel

## Scenario
The accountant at the company received an email titled "Urgent New Order" from a client late in the afternoon. When he attempted to access the attached invoice, he discovered it contained false order information. Subsequently, the SIEM solution generated an alert regarding downloading a potentially malicious file. Upon initial investigation, it was found that the PPT file might be responsible for this download. Could you please conduct a detailed examination of this file?

![hash](screenshots/1.jpeg)

## Tools Used
- virustotal(site)
- Any.run (site)

## Walkthrough

### Q1 — Determining the creation time of the malware can provide insights into its origin. What was the time of malware creation?
**What I did:** I visited the <virustotal> website and entered the provided hash. To find information about the hash, I went to the **Details** tab.

![virustotal](screenshots/2.jpeg)

**What I found:** In the Details tab, I found the **History** section which contained the creation date.

**Answer:** 2022-09-28 17:40

![creation](screenshots/3.jpeg)

### Q2 — Identifying the command and control (C2) server that the malware communicates with can help trace back to the attacker. Which C2 server does the malware in the PPT file communicate with?
**What I did:** Since C2 communication is a network behavior, I checked the **Behavior** tab.

**What I found:** In this tab, I identified the server the malware was communicating with.

![post](screenshots/5.jpeg)

**Answer:** POST http://171.22.28.221/5c06c05b7b34e8e6.php
*NOTE: The malware is on the local machine; for a reverse shell or data exfiltration, the most logical method is POST to establish communication.*

### Q3 — Identifying the initial actions of the malware post-infection can provide insights into its primary objectives. What is the first library that the malware requests post-infection?
**What I did:** To find this information, I stayed in the same tab, as downloading files to the machine is also a malware behavior.

**What I found:** In this tab, specifically in the **Files Dropped** section, I found the downloaded file.

![file](screenshots/6.jpeg)

**Answer:** sqlite3.dll

### Q4 — By examining the provided Any.run report, what RC4 key is used by the malware to decrypt its base64-encoded string?
**What I did:** The link provided by CyberDefenders redirected me to the report on Any.run.

**What I found:** In the **stealc** section, I found the key.

![key](screenshots/7.jpeg)

**Answer:** 5329514621441247975720749009

### Q5 — By examining the MITRE ATT&CK techniques displayed in the Any.run sandbox report, identify the main MITRE technique (not sub-techniques) the malware uses to steal the user’s password.
**What I did:** The link redirects to the report home page. To see the MITRE ATT&CK techniques, I accessed the **MITRE ATT&CK Matrix** by clicking the ATT&CK button under indicators and trackers.

**What I found:** I found all the techniques used during this attack; one of them was "Credentials from Password Stores".

![technique](screenshots/8.jpeg)

**Answer:** T1555

### Q6 — By examining the child processes displayed in the Any.run sandbox report, which directory does the malware target for the deletion of all DLL files?
**What I did:** On the home page, I found a small process tree. The command responsible for deleting the malware's traces was executed by `cmd.exe`, a child process of `VPN.exe`.

**What I found:** I found a command executed by `cmd.exe` that targeted the deletion of a specific directory.

![cmd](screenshots/9.jpeg)

**Answer:** C:\ProgramData

### Q7 — Understanding the malware's behavior post-data exfiltration can give insights into its evasion techniques. By analyzing the child processes, after successfully exfiltrating the user's data, how many seconds does it take for the malware to self-delete?
**What I did:** To determine the delay before the malware deletion, I followed the process tree by clicking on the `timeout` process, then on **More Info**.

**What I found:** I found a representation of the execution timeline.

![timeline](screenshots/10.jpeg)

**Answer:** 5

---

## Incident Timeline
- **t=0 ms:** Execution of the malicious file due to a successful phishing attempt.
- **t=31 ms:** Malware checks the supported language on the machine.
- **t=343 ms:** Malware reads the machine name.
- **t=359 ms:** Malware reads proxy server info in Internet Settings.
- **t=437 ms:** Malware reads the Machine GUID from the registry.
- **t=750 ms:** Malware reads environment variables, CPU info, and local Windows version.
- **t=750 ms:** Search for installed software: Adobe Flash Player 32 ActiveX.
- **t=1703 ms:** Creation of the file `<C:\Users\admin\AppData\Local\Google\Chrome\User Data\Local State>`.
- **t=1781 ms:** Creation of the file `<C:\ProgramData\GCBGCAFIIECBFIDHIJKFBAKEGD>`.
- **t=3593 ms:** Filling the file `<C:\ProgramData\GDHIIDAFIDGCFHJJDGDA>` with stolen credentials.
- **t=4289 ms:** Connection request using a POST method to `<http://171.22.28.221/5c06c05b7b34e8e6.php>` by the process `<VPN.exe>`.
- **t=4290 ms:** Stealc communication detected.
- **t=4309 ms:** External connection established: IpDst:171.22.28.221, PortDst:80.
- **t=4327 ms:** GET request to download `<http://171.22.28.221/9e226a84ec50246d/sqlite3.dll>`.
- **t=10359 ms:** Stealc malware detected in RAM (In-memory execution bypasses file-based antivirus).
- **t=26468 ms:** Creation of the file `<C:\Users\admin\AppData\Roaming\Moonchild Productions\Pale Moon\profiles.ini>`.
- **t=27723 ms:** Download of the file `<C:\ProgramData\mozglue.dll>` (Used to decrypt passwords saved in Firefox).
- **t=29031 ms:** Process `cmd.exe` executes the self-deletion command: `timeout /t 5 & del VPN.exe & del "C:\ProgramData\*.dll"`.

---

## What I Learned
- The `vpn.exe` file contained **Stealc** malware, which focuses on collecting credentials and performing self-deletion to erase traces.
- Stealc was developed by a Russian-speaking actor as a "Malware-as-a-Service" (MaaS) for harvesting personal information.
- Some malwares execute only in **RAM** (In-memory) to avoid detection by traditional file-based antivirus software.
- Firefox encrypts passwords using the `mozglue.dll` library; the malware specifically targets this library to decrypt and steal saved passwords.
