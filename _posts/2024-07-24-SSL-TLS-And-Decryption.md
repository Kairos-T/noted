---
layout: post
title: '[NS Wk 11] SSL/TLS and Decryption'
date: 2024-07-24 18:07 +0800
author: kairos 
categories: []
img_path: "/assets/img"
---

# SSL/TLS Overview

## Purpose

Secures network communication by:
1. Encrypting data for confidentiality/privacy
2. Hashing data for integrity
3. Authenticating parties with certificates for non-repudiation

Decryption helps to inspect encrypted traffic, securing communication by:
1. Preventing malware introduction by inspecting encrypted traffic
2. Preventing data exfiltration by monitoring data in encrypted traffic

## SSL/TLS Operation

Uses digital certificates to authenticate parties and establish secure communication channels.

Steps:
1. Client requests SSL connection
2. Server sends certificate
3. Client verifies certificate
4. Client generates session key, encrypts with server's public key, sends to server
5. Server decrypts session key with private key
6. Both parties use session key to encrypt/decrypt data

# Palo Alto Firewall

## Types

1. **SSL Forward Proxy (Outbound)**: Decrypts outbound traffic (e.g. to prevent data exfiltration like Facebook posting)
	- Goes through SSL proxy
2. **SSL Inbound Inspection**: Decrypts inbound traffic from external users to internal servers 
    (e.g. to prevent malware introduction)
	- Does not use SSL proxy
3. **SSH Decryption**: Decrypts SSH 
	- Uses SSH proxy 

## Certificate Management

### Public Key Infrastructure (PKI)

- Verifies identity of public key owners using digital certificates
- Components:
	1. Root CA: Trusted entity that issues certificates
    2. Intermediate CA: Issues certificates on behalf of root CA
  3. Device Certificate Store: Stores certificates for devices

### Certificate Chain of Trust
- Hierarchical list of certificates used to authenticate devices, services, or people.
- Device --(trust)--> Intermediate CA --(trust)--> Root CA

### Certificate Creation

1. **Self-Signed Certificate**: Generated a certificate, and public-private key pair
	- Used for internal purposes, not trusted by external entities
    - `Device` > `Certificate Management` > `Certificates` > `Add`
    - Leave `Signed By` field blank to create self-signed certificate
    - Select `Certificate Authority`
2. **CA-Signed Certificate**:
	- Method 1: Request certificate from CA, import certificate and public-private key pair into firewall
    - Method 2: Generate CSR and public/private keys on firewall from internal CA, send Certificate Signing Request
      (CSR) to CA, import CA-signed certificate into firewall
	- Import: `Device` > `Certificate Management` > `Certificates` > `Import`

> CSR: Message sent from applicant to CA to apply for digital certificate
  1. Applicant generates public/private key pair
  2. Applicant signs identity information with private key
  3. Applicant sends signed information and public key to CA
  4. CA returns signed certificate 
{: .prompt-info }

## SSL/TLS Decryption

### SSL Forward Proxy

- Allows firewall to intercept, decrypt, and inspect outbound SSL/TLS traffic
- It acts as an intermediary between clients and external servers, enabling security policies to be enforced on 
  encrypted traffic.

### Forward Trust and Forward Untrust Certificates

Forward Trust Certificate
: Installed on client devices to trust firewall's internal CA
- Used when firewall trusts server certificate
- Allows seamless inspection of SSL traffic

Forward Untrust Certificate
: Informs clients during decryption if site the client is accessing is signed by a CA that is not trusted by the firewall.
- Used to warn clients or block access to untrusted sites











