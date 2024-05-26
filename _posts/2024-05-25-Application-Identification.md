---
layout: post
title: '[NS Wk 4] Application Identification'
date: 2024-05-25 17:52 +0800
author: kairos
categories: [Network Security, NS Notes]
img_path: "/assets/img/ns/w4"
---

# App-ID Overview

App-ID
: Identifies applications in traffic and observed by firewall to understand their behaviour and set policies to control
  them.

## App-ID Process

![App-ID Process](app-id-process.png)

1. **App-ID Signature Matching**
   - Identifies applications by matching traffic to application signatures.
   - Signatures are based on application behaviour not port or protocol.
2. **Protocol Decoder Identification**
   - For protocols like SSL, SSH, check Decryption policy.
3. **Behavioural heuristics**
   - Attempts to identify behavioural patterns consistent with known applications.
   - E.g.: Identifying BitTorrent traffic by identifying P2P traffic.
4. **Unknown Traffic**
   - Traffic not identified by App-ID and is handled by Security policy.

## App-ID and Network Traffic

- Port-Based vs NGFW
  - Traditional firewalls allow ALL traffic on a specific port, e.g. port 53 for DNS
  - NGFW allows only specified application on the default port.
  ![Port-Based vs NGFW](port-based-vs-ngfw.png)
- Zero-Day Malware Protection
  - Traditional IPS may not block unknown threats on allowed ports.
  - App-ID blocks all non-specified traffic, including zero-day malware.
  ![Zero-Day Malware Protection](zeroday.png)

### App-ID and TCP

- In TCP, not all traffic contains application data (e.g.: SYN, ACK, FIN), so firewall must wait for the data to identify
  the application.
- App-ID labels TCP traffic using:
  ![App-ID and TCP](app-id-tcp.png)
  - `not-applicable`: Traffic dropped per policy before application identification.
  - `incomplete`: Three-way handshake did not complete or no data followed.
  - `insufficient-data`: Not enough payload for identification.
  - `unknown-tcp`: Unidentifiable TCP traffic.
  - `unknown-p2p`: Unidentifiable peer-to-peer traffic.

### App-ID and UDP

- In UDP, App-ID only examines a single UDP packet to identify the application.
- App-ID labels UDP traffic using:
  ![App-ID and UDP](app-id-udp.png)
  - `not-applicable`: Traffic dropped per policy before application identification.
  - `unknown-udp`: Unidentifiable UDP traffic.
  - `unknown-p2p`: Unidentifiable peer-to-peer traffic.

##  App-ID Operations

### Application Shifts
- Network traffic can shift from one application to another during a session. E.g.: web-browsing to facebook-chat.
- App-ID continuously monitors and updates the application label as more data is received.

### Application Dependencies
- Some applications depend on other applications to function.
- Security policy must allow dependent applications for proper functioning.

### Handling Unknown Traffic

**Unknown Traffic Identification**
- When App-ID cannot identify traffic, it is labelled as `unknown-tcp`, `unknown-udp` or `unknown-p2p`, if HTTP is not detected.
- Security policy must be configured to handle unknown traffic. 
