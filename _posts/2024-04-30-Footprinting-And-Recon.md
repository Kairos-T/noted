---
layout: post
title: '[EH Wk 1] Footprinting and Reconnaissance'
date: 2024-04-30 22:35 +0800
author: kairos
categories: [Ethical Hacking, Notes]
img_path: "/assets/img/eh"
---

## Introduction

These EH notes are simply a summary of the tools and techniques learnt through the labs.

## Lab 1: Footprinting Through Search Engines

### Task 1: Google Hacking

_Ref: [sundowndev's Gist](https://gist.github.com/sundowndev/283efaddbcf896ab405488330d1bbc06)_

| Filter   | Description                                                                                       | Example                                               |
|:---------|:--------------------------------------------------------------------------------------------------|:------------------------------------------------------|
| intext   | Searches for the occurrences of keywords all at once or one at a time.                            | `intext:"keyword"`                                    |
| inurl    | Searches for a URL matching one of the keywords.                                                  | `inurl:"keyword"`                                     |
| allinurl | Searches for a URL matching all the keywords in the query.                                        | `allinurl:"keyword"`                                  |
| intitle  | Searches for occurrences of keywords in title all or one.                                         | `intitle:"keyword"`                                   |
| site     | Specifically searches that particular site and lists all the results for that site.               | `site:"www.google.com"`                               |
| filetype | Searches for a particular filetype mentioned in the query.                                        | `filetype:"pdf"`                                      |
| link     | Searches for external links to pages.                                                             | `link:"keyword"`                                      |
| cache    | Searches for the cached version of a page.                                                        | `cache:"keyword"`                                     |

### Task 2: Image/Video Search Engines

- [Google Lens - Reverse Image Search](https://lens.google.com/)
- [YouTube Metadata Viewer](https://mattw.io/youtube-metadata/)

### Task 3: FTP Search Engines
Search targets (e.g.: Microsoft) on:
- [Napalm FTP Indexer](http://www.searchftps.net/)
- [FreewareWeb FTP File Search](http://www.freewareweb.com/ftpsearch.shtml)

### Task 4: IoT Search Engines
Gather information such as manufacterer, location, IP address, hostname, open ports, etc.
- [Shodan](https://www.shodan.io/)
- [Censys](https://censys.io/)

## Lab 6: Whois Footprinting
Gather information on domain, IP address, owner, its registrar (business that registers domain names), etc.

### Task 1: Whois Lookup using DomainTools / Other Tools

Online Tools:
- [DomainTools](https://whois.domaintools.com/)
- [SmartWhois](https://www.tamos.com/products/smartwhois/)<br>
- (+): May provide additional information such as website snapshots, DNS records, etc.<br>
- (-): Information may be outdated or incomplete since it may not be updated in real-time.

Command Line:
-  `whois <domain>`<br>
- (+): Relatively more up-to-date information.<br>
- (-): May not provide as much information as online tools.

## Lab 7: DNS Footprinting

Gather information on DNS domain names, computer names, IP addresses, mail servers, service records, etc.

### Task 1: DNS Information Gathering using NSLookup / Online Tools

```shell
nslookup
> set type=a
> <domain>
> set type=cname #(alias for another domain)
> <domain>

> set type=a
> <primary name server> # an attacker can target this!
```

Online Tools:
- [NSLookup](http://www.kloth.net/services/nslookup.php)
- [DNSdumpster](https://dnsdumpster.com/)
- [DNS Records](https://network-tools.com)

### Task 2: Reverse DNS Lookup usingDNS Recon / Online Tools

```Parrot Security (Shell)
cd ~/dnsrecon
chmod +x ./dnsrecon.py
./dnsrecon.py -r x.x.x.0-x.x.x.255
```

Online Tools:
- [Network Tools](https://www.yougetsignal.com/tools/web-sites-on-web-server/)

### Task 3: Subdomain and DNS Records Enumeration

- [SecurityTrails](https://securitytrails.com/) - Requires sign-up
- [DNSChecker](https://dnschecker.org/)
- [DNSdumpster](https://dnsdumpster.com/)

## Lab 8: Network Footprinting

Gather information on network range, traceroute, TTL, CIDR, etc.

### Task 1: Locate Network Range

- [ARIN](https://search.arin.net/rdap/) - Enter IP addr

### Task 2: Traceroute/Tracert

```Command Prompt
tracert <domain>
```

```shell
traceroute <domain>
````

[//]: # (## Lab 9: Footprinting Through Footprinting Tools)

[//]: # ()
[//]: # (### Task 1: Recon-ng)
