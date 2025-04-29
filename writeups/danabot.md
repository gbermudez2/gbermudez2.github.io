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

3. Now that we know the file used for Initial Access, I can get the file hash and do some further analysis.
   - Wireshark has a neat option to export HTTP objects. While the exact filename isn't there, this login.php file matches the line number with the TCP stream I examined.

     ![image](https://github.com/user-attachments/assets/5fbb9882-023a-45a4-9ed7-b0db827f215e)

   - 

