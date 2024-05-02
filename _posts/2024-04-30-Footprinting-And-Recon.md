---
layout: post
title: '[EH Wk 1] Footprinting and Reconnaissance'
date: 2024-04-30 22:35 +0800
author: kairos
categories: [ Ethical Hacking, Notes ]
img_path: "/assets/img/eh/w1"
---

## Introduction

These EH notes are simply a summary of the tools and techniques learnt through the labs.

## Lab 1: Footprinting Through Search Engines

### Task 1: Google Hacking

_Ref: [sundowndev's Gist](https://gist.github.com/sundowndev/283efaddbcf896ab405488330d1bbc06)_

| Filter   | Description                                                                         | Example                 |
|:---------|:------------------------------------------------------------------------------------|:------------------------|
| intext   | Searches for the occurrences of keywords all at once or one at a time.              | `intext:"keyword"`      |
| inurl    | Searches for a URL matching one of the keywords.                                    | `inurl:"keyword"`       |
| allinurl | Searches for a URL matching all the keywords in the query.                          | `allinurl:"keyword"`    |
| intitle  | Searches for occurrences of keywords in title all or one.                           | `intitle:"keyword"`     |
| site     | Specifically searches that particular site and lists all the results for that site. | `site:"www.google.com"` |
| filetype | Searches for a particular filetype mentioned in the query.                          | `filetype:"pdf"`        |
| link     | Searches for external links to pages.                                               | `link:"keyword"`        |
| cache    | Searches for the cached version of a page.                                          | `cache:"keyword"`       |

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

- `whois <domain>`<br>
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

```sh
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

```terminal
tracert <domain>
```

```sh
traceroute <domain>
```

## Lab 9: Footprinting Through Footprinting Tools

Gather additional information on target

### Task 1: Recon-ng:

**Set-up**
```sh
# SET-UP
sudo su; cd
recon-ng
> marketplace install all
> workspaces create <name>
> db insert domains
    domain (TEXT): <domain>
```

**Hosts Recon**
```sh

> modules load recon/domains-hosts/brute_hosts # brute force subdomains
> run
> modules load recon/hosts-hosts/reverse_resolve # reverse resolve IP to hostname
> run
> show hosts
```

Dumping Hosts Recon data
```sh
> modules load reporting/html
> options set CREATOR <name>
> options set CUSTOMER <name>
> run # generated report is in /home/attacker/Desktop/
```

### Task 2: Maltego

Gather maximum information on target using transforms and visualises the information in a graph format.

```sh
sudo maltego # Use Maltego CE (Free)
```

Steps:
1. Default System Browser > Privacy Mode: Normal > Open a blank graph...
2. Entity Palette > Website > Change Domain Name
3. Right-click > Run Transform > To: All 
4. Right-click > Run Transforms > Any Tool Required

### Task 3: OSRFramework

OSRFramework
: A set of tools to perform OSINT tasks like social media harvesting, deep web search, etc.

**domainify**
Checks existence of domain names, e.g. eccouncil.com, eccouncil.org, eccouncil.net, etc.

```sh
sudo su; cd
domainify -n <domain> -t all
```

**searchfy**
Searches user details on different social media platforms.

```sh
searchfy -n <username>
```

Other tools:
- `usufy` - Registered accounts with usernames
- `mailfy` - Email addresses
- `phonefy` - Phone numbers
- `entify` - Entities from provided URLs

## Task 4: FOCA Footprinting

FOCA
: Reveals metadata and hidden information in scanned documents on websites - found from Google, Bing, and DuckDuckGo.

Create a new project > Add target Domain website > Add all search engines + extensions > Search all

## Task 5: BillCipher Footprinting

BillCipher
: An information gathering tool for a website or IP address. Gathers information such as GeoIP (IP addr location), DNS, Whois, etc.

**Run**

```sh
sudo su
cd BillCipher # (/home/attacker)
python3 billcipher.py
```

- Enter target domain/IP address
- Choose type of information to gather

![BillCipher](billcipher.png)

## Task 6: OSINT Framework

(T) - Indicates a link to a tool that must be installed and run locally
(D) - Google Dork
(R) - Requires registration
(M) - Indicates a URL that contains the search term and the URL itself must be edited manually
