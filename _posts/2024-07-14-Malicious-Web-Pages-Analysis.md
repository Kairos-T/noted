---
layout: post
title: '[ MATT Wk 12 ] Malicious Web Pages Analysis'
date: 2024-07-14 16:53 +0800
author: kairos
categories: [ Malware Analysis Tools and Techniques, MATT Notes ]
---

# Malicious Websites

Malware can be delivered through the browser or operate within the browser itself.

- Leverage on HTML, JavaScript, and (nowadays) less commonly, VBScript, Flash and Java.

## Protection / Detection

- Use VPNs / anonymising proxies (e.g. TOR, JonDonym)
- Use `wget`/`curl` to download the page and analyse it offline
- Use VirusTotal to scan the page

## Deobfuscation

Browser scripts are often obfuscated to evade detection and complicate analysis.

### Static Analysis

- Use tools like `SpiderMonkey` to deobfuscate JavaScript (on the REMnux VM)
  ```bash
  js -f /usr/local/etc/def.js -f malware.js > deobfuscated.js 
  ```
- Use `Firebug` to inspect the page DOM and JavaScript

### Dynamic Analysis

- Use `Firebug` to debug JavaScript
	- Add `debugger;` at parts of the script (e.g., start of the script, before function / `eval` calls etc.)
	- Inspect the script in the `Script` tab
		- Look for variable values, cookies, etc.

## Common Functions used in JavaScript Browser Malware

- **`eval`** - Executes a string of JavaScript code
- **`document.write`** - Writes HTML content to the page
- **`document.cookie`** - Accesses / Sets cookies for the page
- **`document.location`** - Accesses the URL of the page
- **`document.location.host`** - Returns hostname and port number of the URL of the page
- **`document.body`** - Accesses / Modifies the body of the page
- **`document.createElement`/`document.appendChild`** - Add elements to browser DOM
