# PsExec Hunt Lab

## Category
Network Forensics

## Scenario
 An alert from the Intrusion Detection System (IDS) flagged suspicious lateral movement activity involving PsExec. This indicates potential unauthorized access and movement across the network. As a SOC Analyst, your task is to investigate the provided PCAP file to trace the attacker’s activities. Identify their entry point, the machines targeted, the extent of the breach, and any critical indicators that reveal their tactics and objectives within the compromised environment.

## Tools Used
 - Wireshark

## Walkthrough

### Q1 — To effectively trace the attacker's activities within our network, can you identify the IP address of the machine from which the attacker initially gained access?

![pcap](screenshots/1.jpeg) ![pcap](screenshots/2.jpeg)




**What I did:** I selected the .pcap file in Wireshark for precise analysis. Furthermore, I accessed the conversations section where all traffic activities are summarized by IP, MAC, and size.

**What I found:** I found two communications containing the largest sizes. They shared a common IP; therefore, to verify if this IP belonged to the attacker, I used a filter **ip.addr eq [IP]** to check if this IP was responsible for the first TCP SYN packet in the conversation.

**Answer:** 10.0.0.130
![pcap](screenshots/3.jpeg)


### Q2 — To fully understand the extent of the breach, can you determine the machine's hostname to which the attacker first pivoted?

![pcap](screenshots/4.jpeg)

**What I did:** It should be noted that we have the attacker's IP, which is an internal IP. Consequently, I used this indicator in the traffic filtering.

**What I found:** The filtered traffic contained several packets highlighting a connection attempt using the SMB protocol. One of these packets contained some identification information.

**Answer:** Sales-PC

### Q3 — Knowing the username of the account the attacker used for authentication will give us insights into the extent of the breach. What is the username utilized by the attacker for authentication?

**What I did:** Using the info section and source IP section in Wireshark, I was able to determine which packet contained the identification information used by the attacker.

**What I found:** By examining the packet, specifically the **smb2 header** section, I found the hostname as well as the authentication identifier.

**Answer:** ssales

### Q4 — After figuring out how the attacker moved within our network, we need to know what they did on the target machine. What's the name of the service executable the attacker set up on the target?

![pcap](screenshots/5.jpeg)

**What I did:** To track the attacker's activities, I filtered the traffic using the IP of the machine pivoted by the attacker.

**What I found:** From the info section, while examining the smb2 section in the packet, I found the name of the created executable file.

**Answer:** psexesvc.exe

### Q5 — We need to know how the attacker installed the service on the compromised machine to understand the attacker's lateral movement tactics. This can help identify other affected systems. Which network share was used by PsExec to install the service on the target machine?

**What I did:** First, to find the network share used, we must know what a network share is. Regarding SMB, network shares are like trees pointing to a part of the machine's live or physical memory. 

**What I found:** In the same last authentication packet, specifically the tree section, I found the network used to access and install PsExec.

**Answer:** ADMIN$ (points to the hard drive where the C:\Windows\ tree is located)

### Q6 — We must identify the network share used to communicate between the two machines. Which network share did PsExec use for communication?

![pcap](screenshots/6.jpeg)


**What I did:** The most common network shares in systems are **ADMIN$, IPC$, C$, D$**. I used the timeline of the filtered traffic along with the **info** section.

**What I found:** I found several SMB packets. Furthermore, I tried to understand the chronology of the SMB packets, where the first ones were for SMB communication verification, followed by the packets responsible for authentication. From there, I reached a packet where the attacker tried to establish a logical communication with the victim machine to manage processes in the RAM on the machine.

**Answer:** IPC$

### Q7 — Now that we have a clearer picture of the attacker's activities on the compromised machine, it's important to identify any further lateral movement. What is the hostname of the second machine the attacker targeted to pivot within our network?

![pcap](screenshots/8.jpeg)

**What I did:** To find the second pivoting attempt, I filtered the traffic by including the attacker's IP while excluding the IP of the first machine.

**What I found:** I found SMB packets showing an establishment of communication between two machines, one of which was used by the attacker. Consequently, I examined the packets where I found the second machine.

**Answer:** Marketing-PC

## Activity Timeline
 - t=283,377 | The attacker initializes a TCP connection with a SYN packet using the HR machine.
 
 - t=283,393 | Establishment of the highest common version of SMB, including the digital signature and the encryption algorithm used by SMB.
 
 - t=283,408 | NTLMSSP request and response with identification information to be integrated into the password generation.
 
 - t=283,409 | Successful authentication attempt with username **ssales**.
 
 - t=283,411 | Establishment of a logical SMB2 communication with the target machine using the IPC$ tree.
 
 - t=283,413 | Access to the ADMIN$ tree on the hard drive.
 
 - t=283,416 | Creation of the file **PSexesvc.exe** on the ADMIN$ tree to benefit from privileges.
 
 - t=283,417 | Transmission of the code to the file **Psexesvc.exe**.
 
 - t=283,419 | Information request regarding the file **Psexecsvc.exe**.
 
 - t=534,442 | Pivot attempt to a new machine **Marketing-PC** with user **jdoe**, which subsequently failed.
 
 - t=536,505 | Second successful authentication attempt on the same machine with user **IEuser**.
 
 - t=536,507 | Access to the ADMIN$ tree on the **Marketing-PC** machine.
 
 - t=536,511 | Creation of the file **psexesvc.exe** on the **Marketing-PC** machine.

## What I Learned
 - I learned that the SMB protocol is a data transmission protocol on a local network. Furthermore, to ensure security when using this protocol, an authentication protocol like NTLM must be used, along with firewall configuration to ensure no external device has access to this protocol.
 
 - From the attacker's perspective, it is useful to use the first NTLM response for hostname collection.
 
 - Restricting access to other machines using SMB will be useful in terms of least privilege so that an attacker's pivoting on a network is limited.
 
 - In network shares, ADMIN$ is widely used for transmitting binary files (.exe), while the IPC$ tree is used for remote control.
