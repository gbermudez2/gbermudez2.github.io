---
layout: page
title: "DanaBot Writeup - CyberDefenders"
permalink: /writeups/danabot/
---

# DanaBot Writeup
### April 28, 2025
Link: [Oski Lab](https://cyberdefenders.org/blueteam-ctf-challenges/danabot/)

## Summary
- Tools used: Wireshark, VirusTotal, VirtualBox (Windows 10 VM)
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

   - This file has malicious properties, so **using a VM** would be necessary to do any analysis. Also, the file would be automatically quarantined by Windows Defender, so it's important to allow these files on a safe environment.

     ![image](https://github.com/user-attachments/assets/0c42382d-d398-419e-8b62-de70494dd4d8)

   -  With the file unquarantined, I could get the SHA256 hash of the malware file using PowerShell.
  
     ![image](https://github.com/user-attachments/assets/a086a52f-3505-4aa7-a494-720ee09905fe)

4. Inputting the retrieved hash on VirusTotal can give us valuable insights. Immediately I notice that 25 vendors marked the file as malicious, and the malware contains the tags "obfuscated", "long-sleeps", and "spreader".
   - For infostealing campaigns, this makes sense, considering that long-term persistence and detection evasion across multiple endpoints is necessary for data exfiltration.
  
     ![image](https://github.com/user-attachments/assets/5e2ecd17-f8c0-49da-930c-bd14757dae74)

   - According to further analysis information on [any.run](any.run), this malware uses the **wscript.exe** process to execute the malware.
  
     ![image](https://github.com/user-attachments/assets/e81b97c5-5b6f-4a82-b723-e5eb0138c0c9)

5. Going back to the Wireshark analysis, there seems to be other files within the .pcap that may be worth analyzing. I decided to investigate on the resources.dll file later in the file.
   - .dll files being in network traffic definitely raises some eyebrows, so perhaps it could give more information.
  
     ![image](https://github.com/user-attachments/assets/440c6ac0-a2ec-41b7-9841-891c239da1a0)

   - Unlike the login.php file that Windows Defender detected earlier, this one clearly detects it as DanaBot!
  
     ![image](https://github.com/user-attachments/assets/67c6e370-8bc8-4be8-8410-dfa705a560de)

6. I got the file hash of this second malicious file using PowerShell once again (this time, the MD5 hash):

     ![image](https://github.com/user-attachments/assets/cf3410bc-6ca2-44cb-ad57-59a729c5f308)
  
   - I put the hash into VirusTotal out of curiosity, and it gave all the signs pointing towards malicious activity.
  
     ![image](https://github.com/user-attachments/assets/79dc9ad2-8ab4-47b7-bb8c-3eeef5bf0258)

### Success!
