---
layout: page
title: "Oski Writeup - CyberDefenders"
permalink: /writeups/oski/
---

# Oski Writeup

## Summary
- Tools used: Oracle VirtualBox (Windows 10 VM), VirusTotal
- MITRE ATT&CK Tactics: Initial Access, Execution, Defense Evasion, Credential Access, Command and Control, Exfiltration

## Procedure
1. Since this is a lab with potential malware, it's best practice to open it in a Virtual Machine. I opened my Windows 10 VM environment before extracting and opening the zip.
   - Inside, there's a text file with an MD5 hash, which will be important for analysis. It's a good thing that there's no actual malware in the file, but it's better safe than sorry.
     ![image](https://github.com/user-attachments/assets/eef1c0c9-340d-4d32-9ab6-ea2847803636)

2. Using the MD5 hash as suggested by the .txt file, we'll paste it into VirusTotal for some insights.
   - This is very clearly a malicious file, with 62 vendors flagging it as a trojan.
   - Looking through the Crowdsourced IDS rules, we can see that it's labeled as the **StealC C2** malware, a known infostealer.
     ![image](https://github.com/user-attachments/assets/c86257ba-0e8a-42dc-abf2-fb99454b2d95)

3. We want to find out the **Creation Time** of this malware. The **Details** tab has all we need for this information.
   - Located in the History section, we can see the **Creation Time**.
   - I found it interesting to note that while the malware was made in 2022, its first sighting was almost exactly a year later.
     ![image](https://github.com/user-attachments/assets/9560ff2b-da4e-4b32-a8f9-f07f7e9a0bbe)

4. The lab notes that the malware involved Command and Control, so we can find out the source of the attacks based on any identified IPs.
   - According to the **Contacted URLs** section under the **Relations** tab, there seems to be 2 notable URLs, notably with a domain that shares a common IP.
   - With C2, attackers may set up web servers to beacon and exfiltrate data from their victims.
     ![image](https://github.com/user-attachments/assets/9bb6acfa-d16d-4773-b4aa-6ce381c89fad)

5. If we go to the **Behavior** section, we can look through the file interactions that the malware has, under **Activity Summary**.
   - Many files are modified when this malware is executed, but notably, there is a file called **sqlite3.dll** under the Files Dropped section.
   - According to [HYAS' StealC Report](https://www.hyas.com/blog/caught-in-the-act-stealc-the-cyber-thief-in-c), the malware requests this SQL library to assist with credential exfiltration and database access.
     ![image](https://github.com/user-attachments/assets/a540158f-b38d-4e59-8993-3f813022727e)
     
