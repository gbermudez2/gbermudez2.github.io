---
layout: page
title: "Openfire Writeup - CyberDefenders"
permalink: /writeups/openfire/
---

# Openfire Writeup
### May 1, 2025
Link: [Openfire Lab](https://cyberdefenders.org/blueteam-ctf-challenges/openfire/)

## References
- [NIST](https://nvd.nist.gov/vuln/detail/cve-2023-32315)
- [VulnCheck](https://vulncheck.com/blog/openfire-cve-2023-32315)

## Summary
- Tools used: Wireshark
- MITRE ATT&CK Tactics: Initial Access, Execution, Persistence, Discovery, Command and Control

## Procedure
1. This scenario involves a .pcap file containing network traffic from malicious activity affecting an Openfire server. Wireshark will be my main tool for discovering any IoCs and evidence of attacker actions.

   ![image](https://github.com/user-attachments/assets/9b1d4df6-0544-45ee-8937-cdcdf5c9c537)

2. First, I need to find out if there are any signs of login activity to the server. Openfire is a messaging application server, so it'll be useful in determining any suspicious users accessing these messages.
   - Usually these logins use the HTTP protocol, so putting the search term "http" in the Filter field brings up all relevant data.
   - To look for a login request, we have to find a POST request, since a user would have to enter in credentials:

     ![image](https://github.com/user-attachments/assets/4c6185b0-d208-4abb-9ea8-4eee7e5061b3)

   - Inside the packet data, we find some curious information, like the username, password, and CSRF token, which are all useful.
   - A user attempting to log in as "admin" raises some red flags, though it's best to investigate further before jumping to conclusions.
  
     ![image](https://github.com/user-attachments/assets/abf6cc62-dca7-47e9-bab9-8229927e3c8e)

3. Next, I went through the logs further to find any indicators of account creation, perhaps by the attacker.
   - I thought at first that the account names would either try to blend in (like "Administrator"), but I found an interesting set of network logs that prove otherwise:
  
     ![image](https://github.com/user-attachments/assets/444443a2-7c67-49b5-a7ac-a092aadea8fd)

   - It seems that after failing to log into the admin account three times, the attacker created 3 new accounts with random strings of numbers and letters.
   - What's curious is that in the account creation parameters, the value _isadmin_ is set to "on", which means the attacker used CSRF to enable admin privileges:
  
     ![image](https://github.com/user-attachments/assets/6cd31bc0-d4b0-4cf4-a3dd-1c7a69c2857e)

   - The attacker later logs in with these credentials, which returns a **302 Found** response from the server, signifying a successful login (compared to the previous failed **200 OK** responses).
  
     ![image](https://github.com/user-attachments/assets/e1476333-9708-4cd4-b209-74f76dc0a78c)

4. After confirming that the attacker was trying to access the system, the next reasonable step is to find out the attacker's objectives.
   - Luckily, we don't have to look far to find out. The attacker seems to have submitted a POST request, uploading a plugin called **openfire-plugin.jar** to the server.
  
     ![image](https://github.com/user-attachments/assets/0176a5ca-c9b1-4c90-9475-07f20de5f748)

   - Looking further down the logs, I spotted a few obvious POST commands that utilized this uploaded **openfire-plugin.jar** file. In the post URL, they had an "action=command" string appended to them.
   - I opened the details for the first one I saw, and spotted the following form item:
  
     ![image](https://github.com/user-attachments/assets/0db23996-06fc-420e-b381-b31809c36b1f)

   - The attacker used "whoami" to likely figure out their privileges.
   - The very next POST request was where the main malicious activity was put into place, with the command "nc" (netcat) being used to form a connection between the server and the attacker, perhaps to perform data exfiltration:
  
     ![image](https://github.com/user-attachments/assets/8fdac023-1a20-4ddd-a498-c6eb058628da)

   - Now that the attacker established a reverse shell from port 8888, we can now examine further activity by looking through TCP traffic on port 8888 to IP 192[.]168[.]18[.]60.
   - Various commands like _uname -a_ and _ifconfig_ are shown in the Packet bytes.
  
     ![image](https://github.com/user-attachments/assets/49083c50-3703-4444-9902-93408ab65ccd)
     ![image](https://github.com/user-attachments/assets/a939ef39-f8ff-4a61-9ac4-af79ecd2cbbb)

   
### After doing some more research, I found an [article](https://vulncheck.com/blog/openfire-cve-2023-32315) referencing this vulnerability as CVE-2023-32315, with a CVE 3.0 score of 7.5 (NIST). The premise of the vulnerability involves directory traversal leading into remote code execution. According to VulnCheck, this vulnerability is still present in the wild.
