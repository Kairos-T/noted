---
layout: post
title: '[EH Wk 5] Module 06: System Hacking'
date: 2024-05-18 13:06 +0800
author:
  - galen
  - kairos
categories: [ Ethical Hacking, EH Notes ]
img_path: "/assets/img/eh/w5"
---

# Pt 1: Vulnerability Exploitation for Initial Access

## Microsoft Authentication

### Security Accounts Manager (SAM) Database

- Stores Windows passwords in a hashed format in SAM, the AD database in Domain Controllers.
- Storage Format: `Username: ID: LM Hash: NTLM Hash`
	- LM Hashes have been disabled since Windows Vista, so the portion will be blank, i.e. `Username: ID: : NTLM Hash`

### NTLM Authentication

![NTLM Authentication](ntlm.png)

- New Technology LAN Manager
- Process:
	1. User logs in with username and password.
	2. Client hashes password
	3. Client sends login request to Domain Controller (DC)
	4. DC sends challenge to client
	5. Client hashes password and encrypts challenge sent by DC, then sends it back to DC for verification.
	6. DC verifies and grants access.

### Kerberos Authentication

![Kerberos Authentication](kerberos.png)

- Microsoft's new default authentication protocol for client-server applications.
- Process:
	1. Client sends request to Authentication Server (AS) into key distribution center (KDC).
	2. KDC's AS sends authentication response to client.
	3. Client requests Ticket Granting Ticket (TGT) from Ticket Granting Server (TGS).
	4. TGS sends TGT to client.
	5. Client requests application server to access service.
	6. Server replies to prove identity.

> The significance of the TGT is that it is encrypted using a secret key derived from the user's password and the Kerberos domain's master key.
{: .prompt-info }

## Password Cracking

- Recover passwords from target systems
- Gain unauthorised access to vulnerable systems

### Types of Password Attacks

1. Non-Electronic Attacks
	- Non-technical methods to obtain passwords
	- Includes shoulder surfing, dumpster diving, social engineering
2. Active Online Attacks
	- Attacker has **direct** interaction with target system
	- Includes brute force, dictionary attacks, hash injections/mask attacks, password guessing/spraying
   
   | Attack Type                  | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
   |:-----------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   | Dictionary Attack            | Loads a list of common usernames/passwords to crack passwords                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
   | Brute Force Attack           | Tries all possible combinations of passwords until password is found                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
   | Rule-based Attack            | Crafts payloads based on known information about the password                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
   | Password Spraying            | Targets multiple accounts simultaneously with a small set of probable passwords                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
   | Mask Attack                  | Uses hashes to crack passwords based on known information                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
   | Trojan/Spyware/Keyloggers    | Collect victim's credentials while victim remains unaware                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
   | Hash Injection/Pass the Hash | 1. Attacker logs into local session and extracts domain admin account hash via dumping SAM DB or network sniffing. <br/>2. Attackers injects compromised hash of admin password into local session.<br/>3. Operating system sees provided hash as valid credentials and authenticates the session using that account.                                                                                                                                                                                                                                                                                                                                        |
   | LLMNR/NBT-NS Poisoning       | LLNMR: Link-Local Multicast Name Resolution<br/>NBT-NS: NetBIOS Name Service<br/>1. User queries for host name, and host name is not found in DNS.<br>2. User performs LLMNR/NBT-NS broadcast to resolve hostname<br>3.Attacker responds to broadcast, sending an error message to user but accepting the NTLMv2 hash.                                                                                                                                                                                                                                                                                                                                       |
   | Internal Monologue Attack    | Takes advantage of the NetNTLMv1 challenge-response (auth) protocol, especially the Security Support Provider Interface (SSPI) which manages auth packages.<br>1. Attacker disables NetNTLMv1 security controls, possibly by injecting malicious code.<br>2. Attacker extracts network logon tokens (representing user's auth credentials) from all active processes on the system. <br>3. Attacker relays SSPI requests using the logon tokens to NTLM auth package to receive NTLM responses as the hijacked user. <br>4. Attacker restores original NetNTLMv1 security controls.<br> Attacker cracks the NTLM hash captured or does pass the hash attack. |
   | Cracking Kerberos Passwords  | 1. AS-REP Roasting (Cracking TGT)<br>- Attacker requests TGT from KDC for an account with pre-authentication disabled (Kerberos' master key will not encrypt in the response).<br>- Attacker captures AS-REP response (containing user's encrypted password data) and cracks it to obtain user password.<br>2. Kerberoasting (Cracking TGS)<br>- Attacker requests TGS for Service Principal Name (SPN) of target service account and crack ticket to obtain password                                                                                                                                                                                        |
   | Pass the Ticket              | Leverages on stolen Kerberos TGTs to impersonate users.<br>1. Attacker injects stolen TGT into another system's authentication process (LSASS).<br>2. Attacker impersonates user associated with TGT and requests service tickets to gain access to resources, without needing the credentials.                                                                                                                                                                                                                                                                                                                                                              |
3. Passive Online Attacks
	- Attacker has **no direct** interaction with target system
	- Packet sniffer tools may be run in LAN to intercept raw network traffic
	- Includes wire sniffing, man-in-the-middle attacks, replay attacks
	- MitM/Replay Attacks: Capture network traffic to crack user accounts at a later time
4. Offline Attacks
	- Password cracking occurs on attacker's system.
	- Includes rainbow table attacks and distributed network attacks (e.g. botnets)
	- Rainbow Table Attack: Precomputed word list and associated hashes; captured hashes are compared to the table to
	  find the password
	- DNA: Recover passwords from hash using processing power of multiple systems across a network. DNA manager is
	  installed on a central system, and allocates portions of key search to machines on the network.

## Password Salting

- A random string of characters that are added to password before hash calculations
- Makes it difficult to reverse hash of pre-computed password: hash pairs

---

## Vulnerability Exploitation

Execution of multiple complex, interrelated steps to breach remote systems

## Buffer Overflow

Buffer
: Adjacent memory locations allocated to program data dynamically during runtime

Buffer Overflow
: - Overflows occur when applications accept more data than buffer can contain
- An overflow causes adjacent memory locations outside of buffer to be overwritten
- Malicious code can be injected into buffer to exploit programs/systems

### Types of Buffer Overflow

1. Stack-Based Buffer Overflow
	- Stack: Location for static memory allocation, including function calls, local variables, and return addresses
	- Process:
		1. Attacker takes control of EIP register (instruction pointer) to replace return address with malicious code
		2. Malicious code is executed when function returns
2. Heap-Based Buffer Overflow
	- Heap: Location for dynamic memory allocation during runtime
	- Process:
		1. Data is written to memory without bounds checking
	- Less reliable than stack-based buffer overflow

### Process of Exploiting Buffer Overflow

1. **Spiking**: Identify application is vulnerable to buffer overflow by sending large data to it to cause it to crash
2. **Fuzzing**: Identify specific number of bytes required to crash target application
3. **Finding Offsets**: Use frameworks to generate large pattern of character strings to determine location of EIP
4. **Overwriting EIP**: Replace EIP with address of malicious code
5. **Identifying Bad Characters**: Identify characters that can cause unexpected behaviour in shellcode (e.g. null byte
   \x00)
6. **Identify Right Modules**: Determine modules that lack memory protection and can be exploited
7. **Generate Shellcode**: Create shellcode to be injected to gain shell access in target vulnerable server
8. **Gain Root Access**

## Return-Oriented Programming (ROP) Attack

- Exploitation technique to execute arbitrary malicious code
- Hijacks program flow through reusing available libraries known as gadgets
- Gadgets are instructions that end with x86 RET instructions
- Utilities available and legal code libraries which are not identified by security protections like code signing

## Exploit/Vulnerability Chaining
- Process of combining multiple vulnerabilities to exploit a system and gain root access

---

# Pt 2: Privilege Escalation

After an attacker gets a non-admin user account, the next step is to gain admin privileges to view critical information, delete files, or install malware. This can be caused by design flaws, bugs, and configuration oversights.

## Types

### Horizontal Privilege Escalation

Expanding access rights by gaining access to other user accounts on the same privilege level to gain more information (e.g., files that only some users can view).

### Vertical Privilege Escalation

Exploiting vulnerabilities/misconfigurations to gain admin/root access for complete control over the system.

---

## Privilege Escalation Methods

### Software

Taking advantage of programming flaws in a program/services/OS/kernel to execute malicious code.

**Tools:** ExploitDB

### Hardware (CPU Chips)

**Spectre and Meltdown Vulnerability:**

- **Spectre:** Allows attackers to read memory locations of processes and kernel memory, exposing passwords and keys due to branch prediction.
- **Meltdown:** Allows attackers to force an unprivileged process to read other memory locations to steal credentials and keys stored in memory.

**Tools:** InSpectre, Spectre & Meltdown Checker

**Defence:** Patch and update OS/firmware/BIOS

### Pivoting and Relaying (Network)

![Pivoting and Relaying](pivotrelay.png)

- **Pivot:** Using a compromised system to gain a remote shell on it to access other vulnerable systems in a network.
- **Relay:** Capturing and forwarding sessions/auth credentials to gain access to another system (Requests are coming from compromised system and sent to attacker).
- **Pivot and Relay:** Pivoting to reach an internal server, and use relaying to authenticate and execute commands on it.

### Misconfigured NFS

May allow attackers to sniff sensitive data and files passing through the intranet and launch further attacks.

## Windows

### DLL Hijacking

![DLL Hijacking](dllhijack.png)

Windows applications search through directories for DLLs, and attackers can add a malicious DLL with the same name in a directory that is searched before the real one’s, executing the malicious DLL.

**Tools:** Robber, PowerSploit

**Defence:** View dependency walker to detect problems like invalid modules, import/export mismatches.

### Named Pipe Impersonation

Windows uses named pipes for inter-process communication. Attackers can create a malicious client process that connects to the named pipe, impersonating the server process, allowing them to perform actions of higher privilege access.

**Tools:** Metasploit

### Misconfigured Services

- **Unquoted Service Paths:** Windows launch services (SYSTEM privileges) through absolute paths, like `“C:\Program Files\Service.exe”`, but if the service’s executable path is not quoted, attackers can create an executable with the same name in a directory Windows searches before the intended directory.
- **Service Object Permissions:** Services have permissions/attributes that may be misconfigured, allowing attackers to modify them, e.g., adding new users to the local Administrator group.
- **Unattended Installs:** `Unattend.xml` contains configuration settings used during the installation process which may be exploited by attackers for privilege escalation.

### Windows Sticky Key

When sticky keys are launched, the `sethc.exe` process is launched with SYSTEM privileges. The attacker can open the command prompt, which inherits the SYSTEM privilege from the process. There are many variations of this, like replacing `cmd.exe` with `sethc.exe`.

### User Account Control (UAC) Bypass

Abuse of various techniques to escalate privileges without triggering UAC notifications, such as memory injection, editing registry keys (aka COM Hijacking) to run certain executables with elevated privileges.

**Defence:** Change UAC settings to “Always Notify”.

### Domain Policy Modification / Domain Controller Exploits

Attackers may modify domain settings by changing the group policy and trust relationships between domains; implanting a fake domain controller to escalate privileges.

- **DCSync Attack:** To replicate AD credentials and hashes of privileged accounts.

## MacOS

### Dylib (Dynamic Library) Hijacking

Similar to DLL Hijacking for Windows.

**Tools/Defence:** Dylib Hijack Scanner

---

## Privilege Escalation Defences

- Restrict interactive logon privileges
- Principle of least privilege
- Multi-factor authentication and authorization
- Encrypt sensitive data
- Regularly patch and update kernel/web servers/services
- Ensure all executables are placed in write-protected directories
