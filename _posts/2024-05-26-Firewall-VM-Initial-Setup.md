---
layout: post
title: '[NS Wk 1] Practical - Firewall VM Initial Setup'
date: 2024-05-26 14:04 +0800
author: kairos
categories: [ Network Security, NS Practical ]
---

# Firewall Configuration Concepts

Commit
: Allows users to review and validate configurations before applying them to the firewall. This is a best practice to
avoid misconfigurations.

Ingress Filtering
: Inspection and filtering of incoming traffic at network entry points. Mitigates risk of attacks like spoofing by
ensuring incoming packets have a valid source IP address belonging to the network where the traffic originated.

Egress Filtering
: Inspection and filtering of outgoing traffic from a network. Prevents data leakage or spoofing attacks by ensuring
source IP address belongs to the internal network or valid IP address range, else it is dropped by the firewall.
