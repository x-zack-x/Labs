# Yellow Rat — CyberDefenders

## Category
threat intel

## Scenario
During a regular IT security check at GlobalTech Industries, abnormal network traffic was detected from multiple workstations. Upon initial investigation, it was discovered that certain employees' search queries were being redirected to unfamiliar websites. This discovery raised concerns and prompted a more thorough investigation. Your task is to investigate this incident and gather as much information as possible.

## Tools Used
- virus total
- online reports

## Walkthrough

### Q1 — Understanding the adversary helps defend against attacks. What is the name of the malware family that causes abnormal network traffic?


![hash](screenshots/1.jpeg)

**What I did:** First, I accessed the file provided in the scenario. Then, I used the hash to find the malware family. Knowing the family helps to create better protection and detection methods.

**What I found:** VirusTotal tabs did not show the malware family except in the "Community" tab. To get accurate results, I searched in a **REDCANARY** report where I found the same category.

**Answer:** Yellow Cockatoo Rat


![family](screenshots/3.jpeg) ![family](screenshots/4.jpeg)


### Q2 — As part of our incident response, knowing common filenames the malware uses can help scan other workstations for potential infection. What is the common filename associated with the malware discovered on our workstations?


**What I did:** It is important to know the malware filename to find it on different workstations. I used the hash to find more information about the filename.

**What I found:** In the "Details" tab (Names section), I found several names used by this malware during attacks. I chose the first one because it is the most common.

**Answer:** 111bc461-1ca8-43c6-97ed-911e0e69fdf8.dll


![filename](screenshots/2.jpeg)


### Q3 — Determining the compilation timestamp of malware can reveal insights into its development and deployment timeline. What is the compilation timestamp of the malware that infected our network?


**What I did:** The compilation time is critical information. I used VirusTotal to find this information.

**What I found:** I checked the "Details" tab, under the "History" section.

**Answer:** 2020-09-24 18:26


### Q4 — Understanding when the broader cybersecurity community first identified the malware could help determine how long the malware might have been in the environment before detection. When was the malware first submitted to VirusTotal?


**What I did:** For this question, VirusTotal was my support. Finding when the malware was first detected helps to answer many questions.

**What I found:** I found the date in the "Details" tab, under the "History" section.

**Answer:** 2020-10-15 02:47

![history](screenshots/7.jpeg)


### Q5 — To completely eradicate the threat from Industries' systems, we need to identify all components dropped by the malware. What is the name of the .dat file that the malware dropped in the AppData folder?


**What I did:** To find the files created by this malware, I analyzed several reports. Malware sometimes changes behavior in a sandbox. The results match the information from Red Canary.

**What I found:** The most common file ending in `.dat` in these reports was the answer.

**Answer:** solarmarker.dat


![history](screenshots/5.jpeg)


### Q6 — It is crucial to identify the C2 servers with which the malware communicates to block its communication and prevent further data exfiltration. What is the C2 server that the malware is communicating with?


**What I did:** Finding the C2 communication domain is very important. This IoC can be used as a block rule on the firewall and for detection on other machines. In the "Behavior" tab on VirusTotal, I checked the "Network Communication" section (HTTPS requests, DNS, and Memory Patterns).

**What I found:** This information can be tricky. The HTTPS requests were legitimate URLs after verification. The most reliable information was the **"Memory Pattern URLs"** because they are extracted from the malware's RAM. Analysis on VT showed this URL was not legitimate.

**Answer:** https://gogohid.com


![c2](screenshots/6.jpeg)


## Incident Timeline

- t=2020-09-24 18:26:47 UTC | Compilation time by the malware developer (risk of timestomping);
- t=2020-10-15 02:47:37 UTC | First time this file was uploaded to VirusTotal;
- t=2021-01-18 20:15:04 UTC | First time the malware was detected on a machine;
- t=2025-07-05 19:05:08 UTC | Last time this file was submitted to VirusTotal;
- t=2026-04-02 05:17:15 UTC | Last time the malware was analyzed on a file by VirusTotal;

## What I Learned

- I learned that extensions have different roles. For example, `.dat` is a data file used by malware to hide configuration and avoid detection.

- `.dll` files are libraries with specific functions to perform tasks. They are often used for code injection to bypass basic Antivirus (AV) detection.

- The combination of these files is for **persistence**. **Registry keys** are the most important part because they execute malicious commands or processes automatically when the computer starts.
