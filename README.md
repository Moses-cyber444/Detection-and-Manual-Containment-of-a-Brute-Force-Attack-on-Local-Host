# Detection-and-Manual-Containment-of-a-Brute-Force-Attack-on-Local-Host
### **Key tools Used:** 
* **Kali Linux**
* **Wireshark** 
* **Wazuh**
* **Splunk**

The attack was simulated on a local host environment to analyse how security events are generated and captured by SIEM tools in real-time.
I simulated a brute force attack on my local host by making use of the Nmap command in order to find vulnerabilities on the target. After running the scan, it was observed that the Secure Shell (SSH) port was open. I recognised this is usually a primary point for attackers because SSH is a critical service used for remote administration and if its left poorly secured, it allows an adversary to gain full command-line control of the system. 
<img width="1366" height="413" alt="Screenshot (26)" src="https://github.com/user-attachments/assets/5b0470d6-9ea3-4aa4-94ab-114c0f52ae13" />
After the Nmap scan, I modified the SSH configuration to ensure the service was active and reachable on the local host and ran the brute force attack command to try thousands of password combinations while investigating the traffic in the network using Wireshark.
<img width="1366" height="768" alt="Screenshot (27)" src="https://github.com/user-attachments/assets/f6bce077-248d-4b2e-8db5-161af1159f83" />
<img width="1366" height="768" alt="Screenshot_2026-05-06_07_45_10" src="https://github.com/user-attachments/assets/82a64460-b4e5-4930-b288-9cdbc37383c9" />
In the image above, it is seen that there are red packets which means something went wrong with the connection. There is a flood of RST, ACK (Reset and Acknowledgement) packets. When an attacker tries to log into a server with a wrong password, the server sends a Reset packet and seeing alot of these red packets in a few seconds from the same IP address means someone is using an automated tool like Hydra to guess the password. I filtered the SSH packets and followed the TCP stream of the packet.
<img width="1366" height="768" alt="Screenshot (28)" src="https://github.com/user-attachments/assets/2f869405-2486-4940-9299-84b488c05d9f" />
The communication between the attacker and the target is encrypted becuase SSH is a secure port. This confirms that while an attacker is trying to guess cridentials, the session itself is protected by SSH's encryption layer, preventing any man-in-the-middle sniffing of passwords.
After the attack simulation, I moved to the Wazuh dashboard to verify the detection. I applied a filter for a RULE ID: 5760 (SSH authentication failed) to isolate relevant events. 
<img width="1366" height="768" alt="Screenshot (29)" src="https://github.com/user-attachments/assets/5b54f4d7-7a4e-4655-8f0b-956e9407947a" />
The histogram indicates that there is a massive spike in failed login attempts within a very short timeframe, confirming an automated brute-force attack from the source IP 127.0.0.1.
I also analysed the MITRE att&ck mapping within the wazuh alert. The event was categorised under a tactic for credential access and lateral movement while the technique the attacker used was Password guessng (T1110.001). This allowed me to understand the broader goals of the attacker beyond just the single failed login.
<img width="1366" height="768" alt="Screenshot_2026-05-06_11_44_28" src="https://github.com/user-attachments/assets/9ad23be1-7da0-4ad4-a3e4-0418f95a4db4" />
Splunk was used to aggregate the logs for deeper analysis. I made use of its Search Processing Language (SPL), to create a structured table of authentication failures. This allowed me to quickly identify the targeted username (root) and the source port being used  by the attacker, which is very vital for log analysis.
<img width="1366" height="768" alt="Screenshot_2026-05-06_13_15_31" src="https://github.com/user-attachments/assets/d328bcd0-8c0b-4ea9-8d73-3af35513966c" />
Once the brute force attack was confirmed through log analysis and SIEM (Wazuh and Splunk) alerts, I utilised iptables firewall to contain the threat. By specifically, targeting the SSH port and source IP, I neatralised the attack, preventing any further unauthorised acces attempts while maintaining system availability for other services.
<img width="1366" height="625" alt="Screenshot_2026-05-06_13_29_27" src="https://github.com/user-attachments/assets/acb6955b-8a2f-435a-81f1-575215d3caa1" />
Although I made use of iptables, in an enterprise environment using Endpoint Detection Response (EDR) tools like CrowdStrike Falcon or SentinelOne, the process is usually streamlined to Network Isolation. With a single click, an analyst can logically disconnect a compromised host from the network while maintaining a management connection for investigation.
# Incident Summary
On May 6, 2026, a high-severity brute-force attack was detected targeting the SSH service (Port 22) on the local host. The attack was identified by a massive spike in failed authentication logs within the SIEM tools and was cofirmed through traffic inspection in Wireshark.
# Key Findings:
* **Attacker's IP**: 127.0.0.1 (Local Loopback)
* **Targeted Username**: root
* **Attack Method**: Automated Password guessing (MITRE T1110.001)
* **Evidence**: Correlation between TCP Reset (RST) packets in Wireshark and Rule 5760 alerts in  Wazuh and Splunk.
# Remediation and Containment:
The threat was successfully neutralised by applying iptables DROP rule to block all traffic from source IP to the SSH port. This immediate action stopped the attack and prevented potential credential compromise.
# Conclusion
This lab successfully demonstrated the initial detection, deep packet analysis, log aggregation and final threat containment. It highlights the importance of multi-layered monitoring and the ability to pivot between different security tools to validate and respond to real-world threats.
