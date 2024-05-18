---
layout: post
title: '[EH Wk 3] Module 04: Enumeration'
date: 2024-05-18 13:04 +0800
author: galen
categories: [ Ethical Hacking, EH Notes ]
---

# Enumeration

- Creating active connections with target to perform directed queries to garner information from responses
- Identify points for a system attack
- Generally conducted in an intranet environment (Trusted private network within an organization or facility)

## Techniques for Enumeration

- Username exfiltration through email IDs (Company Emails often have very standardised formats, which can be exploited
  to retrieve usernames for log in portals)
- Exfiltration through default passwords
- Active Directory Brute Force
- Exploiting DNS Zone Transfers
- Extracting User Groups from Windows
- Extracting Usernames through SNMP

## Common service + Associated Ports to Enumerate (More can be found in slides)

- 53: Domain Name System (DNS Zone Transfers)
- 22: Secure Shell (SSH)
- 21: File Transfer Protocol (FTP)
- 139: Netbios Session Service (SMB over NetBIOS)

## Simple Network Management Protocol (SNMP) Enumeration

- Enumerate user accounts and devices on target system utilizing protocol
- Contains Manager and Agents
- 2 Passwords/strings to access and modify SNMP agent from management station:
  1. Read-Only Community String: Public by default  
  2. Read/Write Community string: Private by default
- Default community string allows information exfiltration.

## Lightweight Directory Access Protocol (LDAP) Enumeration

- Protocol for accessing distributed directory services
- Often arranged in hierarchical and logical structure
- Can be queried to extract valid usernames, and addresses

## Network Time Protocol (NTP) Enumeration

- Used to synchronize clocks of networked computers
- Port 123 and UDP used by default
- Can be queried to extract connected hosts, client IP addresses, and possibly internal IPs, if server is in DMZ

## Simple Mail Transfer Protocol (SMTP) Enumeration

- In-built commands

    1.	VRFY: Validates users
    2.	EXPN: Retrieve actual delivery addresses of aliases, and mailing list
    3.	RCPT TO: Defines recipients of email
- Attackers can use these commands to validate users, as they have differentt responses

## Domain Name System (DNS) Enumeration

- Performed through exploiting DNS Zone transfers
- DNS server names, hostnames, usernames, aliases can be obtained

<br>

### **IPsec, VoIP, Unix/Linux, Telnet, FTP, TFTP, SMB, IPV6 and BGP enumeration can be found in slides**


