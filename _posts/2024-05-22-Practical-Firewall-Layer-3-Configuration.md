---
layout: post
title: '[NS Wk 3] Practical - Firewall Layer 3 Configuration'
date: 2024-05-22 16:27 +0800
author: kairos
categories: [ Network Security, NS Practical ]
img_path: "/assets/img/ns/w3"
---

# Practical - Firewall Layer 3 Configuration

This write-up is simply a summary of the practical exercises done in Wk3 without the fluff.

## Scenario

![Scenario](scenario.png)

Objectives:

- Configure Ethernet Interfaces with Layer 3 Information
- Set up virtual router
- Create security policy
- Configure Source NAT

## Configuring Ethernet Interfaces with Layer 3 Information

1. On the Firewall GUI, navigate to `Network > Interfaces > Ethernet`.
2. For `ethernet1/1` interface (connected to Internet), configure the following:
    - Ethernet Interface:
      - Comment: `outside interface`
      - Interface Type: `Layer3`
    - Config Tab:
      - Virtual Router: `lab-vr`
      - Security Zone: `outside` - allows traffic to flow from the outside to the inside
    - Advanced Tab:
      - Management Profile: `ping` - allows ping to the interface
3. For `ethernet1/2` interface, configure the following:
    - Ethernet Interface:
      - Comment: `inside interface`
      - Interface Type: `Layer3`
    - Config Tab:
      - Virtual Router: `lab-vr`
      - Security Zone: `inside`
    - Advanced Tab:
      - Management Profile: `ping`

## Configuring Virtual Router

1. On the Firewall GUI, navigate to `Network > Virtual Routers`, and click on `lab-vr`.
2. Add a new static route:
    - Name: `default-route`
    - Destination: `0.0.0.0/0` - Default route; specifies that all traffic not defined in routing table should be sent
      through the virtual router to the next hop IP address
    - Next Hop IP Address: `192.168.8.2` - VMware's default gateway IP
3. Commit the changes.
4. To verify, go to client VM (VMnet2) and check for IP address obtained via DHCP (E1/2). ping the
   firewall's `ethernet1/2` interface IP address. This should work as it is on the same interface.
5. Client should not be able to ping the firewall's `ethernet1/1` interface IP address as it is on a different 
   interface.

## Create Security Policy
1. On the Firewall GUI, navigate to `Policies > Security`.
2. Ensure `egress-outside` is enabled.
3. Client should now be able to ping the firewall's `ethernet1/1` interface IP address as now there is a security policy
   allowing traffic from the inside to the outside.
4. Client should not be able to access the Internet as there is no NAT policy configured; traffic cannot flow from host
   to communicate through the host's `ethernet1/1` interface.

> Recap: `egress-outside` policy defines the traffic flow from a source zone/interface to a destination zone/interface.
{: .prompt-tip}

## Configure Source NAT

Refer to [Wk3 Notes here](/posts/Security-Policy-And-NAT-Policy/#source-nat-snat) for configurations.
