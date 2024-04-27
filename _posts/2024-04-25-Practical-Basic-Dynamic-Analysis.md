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
- [ ] ApateDNS - To redirect DNS queries to localhost (Note: In case you weren't listening in class, you have to change your preferred DNS server addr to 127.0.0.1)
- [ ] Regshot (1st shot) - To monitor registry changes
- [ ] Netcat - To listen for connections (80/443)

![Set up](3-1-Setup.png)

Right off the bat, there were a bunch of stuff going on. I'll go through them one by one.

![Netcat](3-1-NetCat.png)

On the netcat side, we see some data being transferred via port 443. Given that it's HTTPS, we can't see the data since it's encrypted. Nonetheless, this tells us that the malware is supposed to communicate with a server.

![ApateDNS](3-1-ApateDNS.png)

We can see that the malware is trying to connect to `www.ngeeann-malwareparalysis.com`

If we were to look at Process Explorer and look at what processes the malware spawns (ctrl + L to see the lower pane):

![Process Explorer](3-1-Mutant.png)

We can see that the malware does indeed create a Mutex when it is running.

> Malware utilises Mutexes to ensure that only one instance of the malware is running at any given time. This is to prevent multiple instances of the malware from running, which could lead to conflicts and unintended behaviour. 
{: .prompt-info }

### Q4 - Does the malware load any new DLLs when running?

If we were to look at Process Monitor and filter for `Load Image` events, we can see that the malware does load a bunch of DLLs.

![Process Monitor](3-1-Load-Image.png)

Alternatively, we could look at Process Explorer and see the DLLs that the malware loads (Ctrl + D. To switch back to handles view: Ctrl + H). 

![Process Explorer](3-1-Pex-DLL.png)

### Q5 - Does the malware create any file? If so, what is the filename & location of the file?

We could go to Process Monitor and filter for `CreateFile` operation events.

![Process Monitor](3-1-Create-File.png)

Here, we can see that the malware creates several DLLs in the `C:\Windows\System32` directory.

> Malware often creates dlls in the System32 directory for persistence, stealth, or DLL hijacking (placing malicious dll with the same name to load it into legitimate processes instead of the "real" dll). 
{: .prompt-info }

### Q6 - What registry keys are being modified?

To find out what registry keys are being modified, we could filter for `RegSetValue` in Process Monitor.

![Process Monitor](3-1-RegSetValue.png)

There are two keys that are being modified:

`HKLM\SOFTWARE\Microsoft\Cryptography\RNG\Seed` and `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\VideoDriver`

> Malware modifies the cryptographic seed to ensure that the random numbers generated are predictable. This could be used to generate encryption keys, which could be used to encrypt/decrypt data. 
{: .prompt-info }

> Malware modifies the `VideoDriver` key to ensure that it is executed when the system starts up (It is part of the `Run` keys, which allow programs to run automatically on startup).
{: .prompt-info }

Alternatively, we could take our second Regshot and compare it with the first one to see what registry keys have been modified.

![Regshot](3-1-Regshot.png)

It seems like there are more values detected by Regshot :O

### Q7 - What DNS request is being made?

We saw earlier that the malware was trying to connect to `www.ngeeann-malwareparalysis.com`. After running the malware for around 30+ minutes, we can see that the malware has been requesting `www.ngeeann-malwareparalysis.com` every 1m1s past a certain point. 

![ApateDNS](3-1-ApateDNS2.png)

### Q8 - Use netcat to listen to port 443. What data is being transferred?

We also saw this earlier, and I didn't rerun the netcat instance. We can't decipher the data since it's encrypted anyway.

### Conclusion

I completely forgot to take a screenshot of this, but the malware installs the vmx32to64.exe file, in the `C:\Windows\System32` directory. With the registry keys modified, the malware `vmx32to64.exe` will be executed on startup, and possibly connect to the server `www.ngeeann-malwareparalysis.com` for other malicious payloads. To gain further information on what the malware does, we need to analyse the `vmx32to64.exe` file, which is beyond the scope of this lab. 
