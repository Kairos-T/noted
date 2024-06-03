---
layout: post
title: '[EH Wk 1] Module 01: Introduction to Ethical Hacking'
date: 2024-05-18 13:00 +0800
author: galen
categories: [ Ethical Hacking, EH Notes ]
img_path: "/assets/img/eh/w1"
---

# Introduction to Ethical Hacking

**Ethical Hacking**

- Process of testing and validating security measures of an organisation's information systems and provide remediation
  strategies.
- Includes using tools and techniques used by threat actors to spot vulnerabilities in systems without the bounds of
  legality.

# Classification of Attacks

**Passive Attacks**

- **Does not tamper** with data
- Generally conducted through intercepting information like network traffic
- Includes sniffing and eavesdropping

**Active Attacks**

- Will tamper with data.
- Can **disrupt** communication or services between systems.
- Includes DoS, Man-in-the-middle.

**Close-in Attacks**

- Conducted by attackers that are in close physical proximity to target system.
- Can be **both passive, and active** attack.
- Includes social engineering, eavesdropping, dumpster diving.

**Insider Attacks**

- Conducted by trusted users within a network.
- Use **privileged access** to violate rules to cause a threat to organisation's information or system.
- Includes theft of physical devices, planting keyloggers, backdoors and malware.

**Distribution Attacks**

- Hardware or software attacks that are **tampered with prior** to distribution/installation.
- Could occur at source or transit
- Includes supply chain attacks

# Methodologies / Frameworks / Models

## CEH Hacking Methodology (CHM)

1. **Footprinting**: Gathering information about the target system
2. **Scanning**: Procedures to identify hosts, port, and services
3. **Enumeration**: Actively connecting with target to gather information about users, shares, etc. from responses from
   target
4. **Vulnerability Analysis**: In-depth examination of the ability of a system to withstand attacks
5. **System Hacking**: Gaining access to target system
  - Gaining Access
  - Escalating Privileges
  - Maintaining Access
  - Clearing Logs

## Cyber Kill Chain Methodology

- The sequence of steps an attacker takes to infiltrate a target system
- Component of intelligence-driven defense strategy to detect and prevent malicious intrusion activities
- Important to understand adversary's tactics, techniques and procedures (TTPs) beforehand

![Cyber Kill Chain](ckcm.png)

1. Reconnaissance
2. Weaponization
3. Delivery
4. Exploitation
5. Installation
6. Command and Control (C2)
7. Actions on Objectives

## Tactics, Techniques, and Procedures (TTPs)

1. Tactics:
    - Guidelines on the way(s) an attacker conducts an attack
    - Tactics exist for different stages of an attack (e.g. initial exploitation, privilege escalation, lateral movement,
      etc.)
2. Techniques:
    - Technical methods employed by an attacker
    - Includes initial exploitation, setting up C2 servers, and accessing target infrastructure.
3. Procedures:
    - Organisational approach that threat actors follow

## MITRE ATT&CK Framework

1. Reconnaissance
2. Weaponize
3. Deliver
4. Exploit
5. Control
6. Execute
7. Maintain

## Diamond Model of Intrusion Analysis

![Diamond Model](dmia.webp)
_Ref: [Trellix](https://www.trellix.com/en-us/img/newsroom/stories/archives/ryuk-exploring-the-human-connection-1.jpg)_

- Used to identify clusters of events that are correlated to any system intrusion

1. **Adversary**: Entity **who** was behind an attack
2. **Victim**: Location **where** exploitation occurred or target that was attacked
3. **Capability**: **How** the attack was performed or strategies used, e.g. skills
4. **Infrastructure**: **What** was used to perform the attack, e.g. C2 servers, malware, etc.

# Types of Hackers

1. **Black Hat**: Illegal, unethical hacking
2. **White Hat**: Legal hackers, using skills for defensive purposes
3. **Grey Hat**: Works both offensively and defensively
4. **Cyberterrorists**: Hackers that cause fear through disruptions or damage to systems
5. **Hacktivists**: Hackers that use hacking to promote political agendas
