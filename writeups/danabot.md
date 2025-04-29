---
layout: page
title: "DanaBot Writeup - CyberDefenders"
permalink: /writeups/danabot/
---

# DanaBot Writeup
### April 28, 2025
Link: [Oski Lab](https://cyberdefenders.org/blueteam-ctf-challenges/danabot/)

## Summary
- Tools used: Wireshark, VirusTotal
- MITRE ATT&CK Tactics: Execution, Command and Control

## Procedure
1. The scenario involves a .pcap file containing network traffic from the DanaBot malware. Wireshark will be a useful tool for analyzing the traffic here!

   ![image](https://github.com/user-attachments/assets/7b10c331-72bf-41de-acf8-5dee67599cc1)

2. First, I need to find out the attacker's IP used for Initial Access.
   - To narrow down any IPs looking to retrieve info, I performed a "http.request.method==GET" search.

     ![image](https://github.com/user-attachments/assets/75933b7e-36f4-4368-aed2-0021cf37594b)

   - This narrows down the possible IPs to merely four.
   - My next idea was to search the network traffic of each IP listed above to spot any signs of activity. It's possible that all of these IPs are malicious, but since I'm looking for the Initial Access IP, the one I'm looking for will be the earliest one.
   - To start off, I searched for the first IP with "ip.addr==62.173.142.148".

     ![image](https://github.com/user-attachments/assets/0a02596e-3a49-4609-9dbd-92eb7afb10de)

   - That's weird... TCP PDU seems oddly suspicious. I decided to follow the TCP stream to get more insights, and lo-and-behold:
  
     ![image](https://github.com/user-attachments/assets/78d03dec-89a6-4224-905e-069a8f883a33)

   - This seems like a jumbled mess. I did some research and found an [article by cyfirma](https://www.cyfirma.com/research/danabot-stealer-a-multistage-maas-malware-re-emerges-with-reduced-detectability/) detailing the vulnerability, which mentions the use of a JavaScript file filled with "meaningless variable declarations to avoid static detections".
   - This is further solidified by the fact that the TCP stream states that the filename is "_allegato_708.js_". It seems I didn't have to look far to find the culprit!
  
     ![image](https://github.com/user-attachments/assets/7b09e38f-269a-4a09-9f37-998c2ec30895)


4. We want to find out the **Creation Time** of this malware. The **Details** tab has all we need for this information.
   - Located in the History section, we can see the **Creation Time**.
   - I found it interesting to note that while the malware was made in 2022, its first sighting was almost exactly a year later.

     ![image](https://github.com/user-attachments/assets/9560ff2b-da4e-4b32-a8f9-f07f7e9a0bbe)

5. The lab notes that the malware involved Command and Control, so we can find out the source of the attacks based on any identified IPs.
   - According to the **Contacted URLs** section under the **Relations** tab, there seems to be 2 notable URLs, notably with a domain that shares a common IP.
   - With C2, attackers may set up web servers to beacon and exfiltrate data from their victims.

     ![image](https://github.com/user-attachments/assets/9bb6acfa-d16d-4773-b4aa-6ce381c89fad)

6. If we go to the **Behavior** section, we can look through the file interactions that the malware has, under **Activity Summary**.
   - Many files are modified when this malware is executed, but notably, there is a file called **sqlite3.dll** under the Files Dropped section.
   - According to [HYAS' StealC Report](https://www.hyas.com/blog/caught-in-the-act-stealc-the-cyber-thief-in-c), the malware requests this SQL library to assist with credential exfiltration and database access.

     ![image](https://github.com/user-attachments/assets/a540158f-b38d-4e59-8993-3f813022727e)
     
7. The lab mentions that the StealC malware utilizes **RC4** for "decrypting a base64 string". Upon research, this method of encryption is used to most likely evade detection from anti-malware or static code scanning.
   - Admittedly, I was not familiar with RC4 before, so I did some research on it, and it is a stream-cipher algorithm created by the makers of the RSA algorithm.
   - Analyzing the malware may give insights into the RC4 key that the malware authors used to encrypt their code.
   - any.run has a library of user reports regarding malware. I made a search using the hash, and clicked on the oldest entry to view the report.

     ![image](https://github.com/user-attachments/assets/bd193017-7843-4824-9ed1-ce79fc03e3d9)

   - I scrolled down a bit to view any relevant information, and I found some under **Malware Configuration**. There's a convenient section labeled "Keys" that contained the RC4 key. Using this to decrypt the attacker's code would be extremely useful for static code analysis.
  
     ![image](https://github.com/user-attachments/assets/ff74b795-add4-4e2a-b133-e07bf43d28da)

8. StealC, as I mentioned earlier, is a known infostealer. Knowing the TTPs of the attackers can reveal vital information into how these attacks operate.
   - Luckily, the full any.run report contains information regarding these tactics and techniques in a convenient ATT&CK matrix.
  
     ![image](https://github.com/user-attachments/assets/9d14955b-bfa6-4e2d-af81-72ba89cff231)

   - StealC steals credentials like passwords from their victims. This is labeled as "Credentials from Password Stores" under **Credential Access**, aka T1555.

     ![image](https://github.com/user-attachments/assets/ca3de6c8-63fb-4108-a0d3-c22e34d1f6a2)

9. In addition to code obfuscation, StealC and similar malware may induce tactics to evade further detection by analysts.
   - I remembered from my earlier analysis that VirusTotal lists the files that were deleted. However, these didn't correlate to any specific directory or file pattern for Defense Evasion; these files were deleted _on execution_.
   - I decided to go back to the any.run analysis. Within the MITRE ATT&CK matrix, there's a technique labeled "Indicator Removal" under the tactic **Defense Evasion**.
   - Within the command line details, there's a command called "del C:\ProgramData\*.dll" that self-deletes after payload execution. Judging by the .dll, it appears to remove any traces of the malware libraries in the ProgramData folder.
   - Looking earlier into the command, the "timeout /t 5" also indicates that the malware waits 5 seconds after exfiltration before self-deleting.

   ![image](https://github.com/user-attachments/assets/747222da-04f2-4ea5-bd6e-da5dcbeb0548)

### Most advanced malware have the capability to evade detection and cyber defenses; it's important to be vigilant and look for other indicators beyond physical evidence, such as behavior and heuristics.
