# RedTeam Tactics
## Multi-Host Tactics

## Objective

Emulate an attackers behavior to better understand intrusion techniques, lateral movement, privilege escalation and post-exploitation tactics.  This lab is designed to give me a better understanding of detection and response skills relevant to SOC environments.

---

## Skills Learned

- **Vulnerability Assessment** – Discovered a command injection flaw in a web application.
- **Network Reconnaissance** – Scanned and mapped open ports using Nmap.
- **Penetration Testing** – Used critical thinking to find and expose vulnerabilities.
- **Lateral Movement** – Able to use my foothold in one area of the system to gain access into another.
- **Metasploit** – Figured out how to gain a meterpreter shell using exploits in metasploit.
- **Post-Exploitation** – Found multiple ways to move throughout the system.
- **Password Cracking** – Found a hash and used resources to crack it.

---

## Tools Used

- Nmap  
- Hashdump 
- Command Injection
- Metasploit
- Crackstation  
- Command Line Interface  
- SSH keys
- Meterpreter

---

## Exploitation Steps

### 1. Network Scanning
- From my kali linux machine, I needed to find my IP with this in terminal.
  ```bash
  ip a
- My IP is 172.31.59.189
- Then I mapped the network with active reconnaissance
  ```bash
  nmap -sV 172.31.59.0/20
- 20 was the subnet I was given before hand.
- After I found the IPs, I had to ping each one to see what ports they had open and the service versions of all
  ```bash
  nmap -p 1-5000 <ip>
  
 ![image](https://github.com/user-attachments/assets/ea463430-e459-47e2-8b8a-7c9dd2a7688b)
 ![image](https://github.com/user-attachments/assets/ea273534-5da2-4d4a-ab6e-378aab13f597)
- 172.31.31.49 has RDP and SMB ports open and is a Windows-based system.
- 172.31.50.107 has http port set to 1013 instead of 80.
- 172.31.62.2 has ssh port set to 2222 instead of 21.
### 2. Initial Compramise
- Decided to attack the public facing webserver first since I should not need a user name or password.
- I opened a web browser and went to http://172.31.50.107:1013
- They had a ping lookup on the site I could try command injection and was successful.
- The command I used was <ip> && whoami to find the user
 ![image](https://github.com/user-attachments/assets/b2565c3a-a72c-43d3-9faf-3beb351fb332)
- Knowing the vulnerability, I try different commands
### 3. Pivoting
- I have a feeling that the IP with ssh set to 2222 is vulnerable, so I look for ssh keys to copy
- SSH keys are usually in /home
- I also notice /home has names of user accounts I can attempt to see if they are vulnerable.
 ![image](https://github.com/user-attachments/assets/fd70f38f-b7ca-47bf-b89b-77c16b91a828)
- So because I am trying to get an ssh key for the IP I'm not sure what user name goes with it, I decide to copy all keys I can.(alicekey.pem, ubuntukey.pem, labsuserkey.pem
- The only other user with ssh keys I can find is alice, so I try alice ssh key on the IP that has ssh on port 2222.
  ```bash
  chmod u+x alicekey.pem
  ssh -i <keyname.pem> <username>@<ip> -p <port>

 ![image](https://github.com/user-attachments/assets/9102b495-d687-4b3f-8f1e-ec785b11f30f)
### 4. System Reconaissance
- To break into the last one, I poke around a bit and then notice they have a script sitting in /scripts
  ```bash
  find script
  cd /scripts
  cat windows-maintenance.sh

 ![image](https://github.com/user-attachments/assets/0434d0a5-779f-4dce-a574-65002486670d)

### 5. Password Cracking
- With a username and a hash, I can check https://crackstation.net to see if its a known hash
- Success
  ![image](https://github.com/user-attachments/assets/83d4ec69-452a-4133-990f-e4298661c90a)
### 6. Metasploit
- With a username and password, and knowing the operating system is windows from my nmap scan earlier, I can try to run metasploit to get a reverse shell
- I also know smb is open.
- Launch metasploit from kali not alice
  ```bash
  msfconsole
  search exploit/windows/smb
  use 21
- I can use the exploit exploit/windows/smb/psexec
- My payload will be windows/x64/meterpreter/reverse_tcp to get a meterpreter shell
  ![image](https://github.com/user-attachments/assets/4df0a059-dc17-479b-ac72-f9112e277ecf)
  ![image](https://github.com/user-attachments/assets/b58c239f-f7a8-4784-9d1c-e89d94c2e99b)
### 7. Pass the Hash
  - The final computer on the network can be compromised with hashdump
  - With meterpreter, I can background the first session, then set my options to attack the second computer with the username and hash I just gained access to.
  ![image](https://github.com/user-attachments/assets/1543a5f0-d313-457f-930c-02043c96ac8b)
  ![image](https://github.com/user-attachments/assets/d02e3eb3-4154-42e9-ae73-0dd8946bcfe1)
- This is how the entire network may be compromised from a bad actor.
