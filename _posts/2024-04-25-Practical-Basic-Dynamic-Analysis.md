---
title: "[MATT Wk 2] Windows Malware and Basic Static Analysis"
author: kairos
categories: [ Malware Analysis Tools and Techniques, MATT Practical ]
img_path: /assets/img/matt/basic-dynamic-analysis
mermaid: true
---

These labs are based off the Week 2 Basic Dynamic Analysis - Practical. The labs can be found [here](https://github.com/mikesiko/PracticalMalwareAnalysis-Labs). Note that these are my personal solutions and may not be 100% accurate.

## Lab 01

For this lab, the file `lab03-01.exe` is provided. Before we delve into the dynamic analysis, we should first perform basic static analysis to gather some information about the file.

### Basic Static Analysis + Q1 - What DLLs does the malware import? Could the malware be packed?

If we were to look at the PEiD scan of the file, we see `PEncrypt 3.1 Final -> Junkcode`, instead of a compiler.

![PEiD Scan](3-1-PEiD.png)

A quick Google search shows that it is a packer, and encrypted the original code. Unfortunately, I couldn't find an unpacker, and we would have to manually unpack it using a debugger (like IDA), but it's beyond the scope of this lab. The alternative would be to perform basic dynamic analysis on the file.

![Google Search](3-1-Google.png)

Now, let's look at the DLLs that the malware imports. Looking at dependency walker, we see that it only imports one DLL, `KERNEL32.dll`. Additionally, it only imports ONE function from it, which is `ExitProcess`. This is a strong indicator that the malware is packed.

![Dependency Walker](3-1-Dependency-Walker.png)

### Q2 - What are some strings that may provide useful information for analysis?
To further confirm this, and also to gain more information, we could use strings.

![Strings](3-1-Strings.png)

Here are some strings that may provide useful information for analysis:

| String                                                           | Description                                                                                                                                                                                                                                        |
|:-----------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| CONNECT %s:i HTTP/1.0                                            | Malware connects to a server                                                                                                                                                                                                                       |
| SOFTWARE\Classes\http\shell\open\command                         | This registry key is responsible for defining the default action associated with opening HTTP links. <br/>The malware could modify this key to hijack the default behaviour of opening web links<br/> E.g.: Redirecting user to malicious websites |
| Software\Microsoft\Active Setup\Installed Components             | This registry key is responsible for the installation of software during the setup process. <br/>Malware could add itself to this key to ensure it gets installed during system startup / other specific events.                                   |
| www.practicalmalwareanalysis.com                                 | Malware probably connects to this website                                                                                                                                                                                                          |
| admin                                                            | -                                                                                                                                                                                                                                                  |
| VideoDriver                                                      |                                                                                                                                                                                                                                                    | 
| WinVMX32-                                                        |                                                                                                                                                                                                                                                    |
| vm32to64.exe                                                     |                                                                                                                                                                                                                                                    |
| SOFTWARE\Microsoft\Windows\CurrentVersion\Run                    | This registry key allows programs/scripts to be auto executed when a user logs into the system. <br/>This allows malware to achieve persistence, continuing its malicious activities without user interaction.                                     |
| SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders | This registry key contains subkeys that specify the location of system folders (Desktop, Start Menu, etc). <br/>Malware might modify these subkeys to change the locations of the folders, hiding malicious files.                                 |

For the ones that have a blank description, it's hard to tell what they are used for, so we need the basic dynamic analysis to tell us!

### Set up + Q3 - Does the malware create a Mutex when it is running? 

Since we know some information about the malware, we can proceed to the dynamic analysis. Before we run the malware, we should set up some monitoring tools:

- [ ] Process Monitor - To monitor file system changes
- [ ] Process Explorer - To monitor processes
- [ ] ApateDNS - To redirect DNS queries to localhost
- [ ] Regshot (1st shot) - To monitor registry changes
- [ ] Netcat - To listen for connections (80/443)
