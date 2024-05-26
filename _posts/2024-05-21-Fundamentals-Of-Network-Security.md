---
layout: post
title: '[NS Wk 1] Fundamentals of Network Security'
date: 2024-05-21 22:23 +0800
author: kairos
categories: [ Network Security, NS Notes ]
img_path: "/assets/img/ns/w1"
---

# Secure Network Designs

## Weak Network Design/Architecture

- **Single point of failure**: One device failing can bring down the entire network
- **Complex dependencies**: Difficult to troubleshoot and maintain
- **Prioritising availability over confidentiality and integrit**y: E.g.: Open networks with no security measures
- **Lack of documentation and change control**: Difficult to maintain and troubleshoot
- **Over-dependence on perimeter security**: Leads to weak internal security

## Network Appliances

![Network Appliances](appliances.png)
_Layers of appliances in a network design_

## Concepts in Network Security

### Network Topology and Zones

Network Topology
: Physical and logical layout of a network

Zones
: Isolated network for hosts with similar security requirements
- Traffic between zones is controlled by security policies
- Zone types:
- Intranet (private, internal network)
- Extranet (private network for trusted partners, e.g.: VPN for business partners)
- Internet (public network)
- Enterprise architecture zones: Access blocks representing host groups with similar security requirements

### Demilitarised Zone (DMZ)

- Network segment that separates the internal network from an external network
- Purpose: Isolate Internet-facing hosts; No direct communication with internal network from DMZ is allowed

Best Practices:

- Use proxies to rebuild packets for forwarding to internal network
- Utilise Bastion Hosts: Secure hosts that provide services to external users
- Use different DMZ types for different functions (e.g.: Web DMZ, Mail DMZ)

> The DMZ is sometimes referred to as a "perimeter network".
{: .prompt-tip }

DMZ Topologies:

- **Screened Subnet**: Redundancy in firewalls
  - Two firewalls on both sides of DMZ
  - Edge/screening firewall: Restricts external traffic to DMZ
  - Internal/choke firewall: Restricts traffic from DMZ to internal network - Choke point; narrow gateway faciliating
    better access control and monitoring
    ![Screened Subnet](screenedsubnet.png)
    _Ref: [TechTarget](https://cdn.ttgtmedia.com/rms/onlineimages/sdn-dmz_network_architecture_mobile.png)_
- **Triple-Homed Firewall**
  - Single appliance with three interfaces: Public, DMZ and LAN
  - Routing and filtering rules manage traffic between interfaces
    ![Triple-Homed Firewall](triplehomed.png)
- **Dual-Homed Gateway**: Smaller networks with budget/technical constraints to implement DMZ
  - Single firewall with two interfaces that sits between untrusted and trusted networks, i.e. Internet and LAN
    ![Dual-Homed Gateway](dualhomed.png)

### Proxies

Proxy
: Acts as an intermediary between clients and servers
- Main types:
- Forward Proxy
- Opens connections with **external** servers for internal clients
- Application-specific: HTTP, FTP, etc.
- Supports authentication
- Reverse Proxy
- Opens connections with **internal** servers for external clients
- Load balancing, caching, SSL termination
- Protects internal servers from direct exposure to external clients

# Cyber Attack Lifecycle

- **Stages**:
  - Reconnaissance: Attackers gather information about targets
  - Weaponization: Attackers create malicious payloads
  - Delivery: Attackers deliver payloads to targets
  - Exploitation: Attackers exploit vulnerabilities to gain access
  - Installation: Attackers install malware on the target system
  - Command & Control: Attackers establish a remote control channel
  - Act on Objective: Attackers achieve their goals (e.g., data theft, disruption)

# Firewall Implementations

**Packet Filtering Firewall**

- Enforces network access control lists (ACLs)
- Inspects headers of individual packets (source/destination IP, protocol ID/type, port numbers)
- Stateless: No memory of previous packets; Analyses each packet independently

**Stateful Firewall**

- Maintains state table to track connections
- Transport layer (Layer 4): Track TCP handshakes and connections
- Application layer (Layer 7): Validate protocols and match threat signatures

**iptables**

- Linux kernel firewall
- Command-line tool to manage packet filtering rules

## Firewall Types

Firewall Appliances

- **Routed Mode**: Layer 3 firewall; Routes traffic between interfaces
- **Bridged/Transparent Mode**: Layer 2 firewall; Acts as a bridge between interfaces
- **Router/Firewall**: Combines routing and firewall capabilities

Application-Based Firewalls

- **Host-Based Firewalls**: Protect individual hosts
- **Application Firewalls**: Inspect application-layer traffic
- **Network Operating System (NOS) Firewalls**: Software-based running under network server OS. Acts as a gateway/proxy

## Access Control Lists (ACLs)

- Configured on the principle of least access
- Only allow minimum traffic required for the operation of valid network services and no more

# Next-Generation Firewall Architecture

## PAN Single-Pass Architecture

- Single-pass software architecture; combines multiple security functions into a single stream-based architecture acting
  **per packet**
- Traffic classification (App-ID), user/group mapping, content scanning (threats, URL, confidential data)
- Parallel processing hardware; separate data and control planes

### Control and Data Planes

![Control and Data Planes](controlanddata.png)

Control Plane
: Manages the firewall's configuration, logging and reporting on a separate processor, RAM and hard drive

Data Plane
: Handles signature matching, security processing (App-ID, User-ID, URL matching, etc), and Network Processing 
(NAT, QoS)

## Zero Trust Architecture

- Never trust, always verify
- Inspect perimeter traffic and internal traffic - inbound and outbound
