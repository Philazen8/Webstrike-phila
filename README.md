# Webstrike-phila
A detailed forensic analysis of the WebStrike blue team lab from CyberDefenders, focusing on investigating a web server compromise through network traffic analysis. This repository documents my systematic approach to identifying attacker techniques, analyzing malicious activity, and extracting critical indicators of compromise from PCAP files.

# 🎯 Learning Objectives
Through this investigation, I aimed to build and demonstrate competency in:
Network Traffic Analysis: Using Wireshark to filter and inspect malicious packets.
Attack Chain Reconstruction: Chronologically mapping the stages of a cyber attack.
Indicator of Compromise (IOC) Extraction: Identifying key forensic artifacts (IPs, hashes, filenames).
Professional Documentation: Creating clear, actionable reports for stakeholders.
MITRE ATT&CK Mapping: Aligning attacker techniques to a standardized framework.

# 1. Executive Summary
On 25 January 2026, an analysis of network traffic (WebStrike.pcap) revealed a successful web server compromise. An attacker from Tianjin, China (IP: 117.11.88.124) exploited improper input validation to upload a malicious web shell (image.jpg.php) to the /reviews/uploads/ directory. The attacker subsequently used this shell to establish a reverse shell on port 8080 and exfiltrate the system file /etc/passwd. The server 24.49.63.79 is considered compromised.

# 2. Methodology
The investigation was conducted via static analysis of the provided PCAP file using Wireshark. Focus was placed on HTTP traffic, particularly POST requests, and follow-up TCP streams to reconstruct events.

# 3. Detailed Findings & Attack Narrative

Initial Reconnaissance/Attack Origin: The attack originated from IP 117.11.88.124 (Tianjin, China), using the User-Agent Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0.

Vulnerability Exploitation: The attacker made two POST requests:

An attempt to upload image.php was rejected by the server.
A second attempt to upload image.jpg.php was successful. The double extension (.jpg.php) likely bypassed weak file upload filters.
Establishment of Foothold: The content of image.jpg.php contained a web shell command instructing the server to connect back to the attacker's machine (117.11.88.124) on port 8080 using netcat (nc).

Post-Compromise Activity: Analysis of traffic on port 8080 showed the attacker successfully executed a curl command from the compromised server (24.49.63.79), targeting and exfiltrating the sensitive system file /etc/passwd.

# 4. Indicators of Compromise (IOCs)

Type	Value	Context
Attacker IP	117.11.88.124	Source of attack & C2 server.
Malicious Filename	image.jpg.php	Uploaded web shell.
Target Path	/reviews/uploads/	Location of uploaded web shell.
C2 Port	8080/tcp	Port used for reverse shell.
Exfiltrated File	/etc/passwd	Sensitive file targeted for exfiltration.

# 5. MITRE ATT&CK Mapping

Tactic	Technique (ID)	Description
Initial Access	Exploit Public-Facing Application (T1190)	Exploited the file upload functionality.
Persistence	Web Shell (T1505.003)	Uploaded image.jpg.php for persistent access.
Command & Control	Ingress Tool Transfer (T1105)	Used netcat to establish a reverse shell.
Collection	Data from Local System (T1005)	Targeted and exfiltrated /etc/passwd.

# 6. Conclusion & Recommendations
Conclusion: The web server was fully compromised due to insufficient file upload validation, allowing a web shell to be planted and used for data theft.
Recommendations:

Immediate: Isolate the affected server (24.49.63.79). Remove the web shell file (image.jpg.php from /reviews/uploads/).

Technical: Implement strict file upload validation (checking file type by content, not extension). Review logs for other indicators from IP 117.11.88.124.

Preventative: Apply the principle of least privilege to web server directories. Implement a Web Application Firewall (WAF) ruleset.

