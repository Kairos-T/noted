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


## Lab 02

For this lab, we will be using `Lab03-02.dll`. As usual, we should first perform basic static analysis to gather some information about the file.

### Basic Static Analysis + Q1 - What are the imported & exported functions of this DLL?

Looking at Dependency Walker, we can see that the DLL imports a few dlls — `KERNEL32.dll`, `ADWAPI32.dll`, `WS2_32.dll`, `WININET.dll` and `MSVCRT.dll`. 

![Dependency Walker](3-2-Dependency-Walker.png)

We can also see that the DLL exports 5 functions. The most interesting ones would be `Install` and `InstallA`, which are probably going to be our entry points.

Looking more into some interesting imported functions:-

**KERNEL32.dll**
![KERNEL32.dll](3-2-kerneldll.png)

- `CreateProcessA` - Creates a new process and its primary thread. 
- `GetCurrentDirectoryA` - Retrieves the current directory for the current process. This could be used to determine the current working directory of the malware.
- `GetLastError` - Retrieves the calling thread's last-error code value. Interesting?
- `GetModuleFileNameA` - Retrieves the full path and filename of the executable file of the current process. This could be used to determine the location of the malware.
- ...
- `ReadFile` - Reads data from a file, starting at the position indicated by the file pointer. 
- `Sleep` - Suspends the execution of the current thread for a specified interval. This could be used to delay the execution of the malware - a common evasion technique for sandbox environments!

**ADWAPI32.dll**
![ADWAPI32.dll](3-2-adwapi32dll.png)

- `RegCloseKey` / `RegCreateKeyA` / `RegOpenKeyExA` / `RegQueryValueExA` / `RegSetValueExA` - Functions related to registry manipulation.

**WS2_32.dll**
![WS2_32.dll](3-2-ws232dll.png)

- `WSASocketA` - Creates a socket! This could be used to establish a connection to a remote server.

**WININET.dll**
![WININET.dll](3-2-wininetdll.png)

- `HttpOpenRequestA` / `HttpQueryInfoA` / `HttpSendRequestA` - Functions related to HTTP requests. This could be used to communicate with a remote server.
- `InternetConnectA` / `InternetOpenA` - Functions related to internet connections, probably used to establish a connection to a remote server.
- `InternetReadFile` - Reads data from the specified file/URL/directory.

To get a better idea of what the malware does / what we should pay attention to, we could use strings.

![Strings](3-2-Strings.png)

We can see that the malware is probably trying to connect to `ngeeann-malwareparalysis.com`. We should also look out for `svchost.exe` processes, as well as registry keys being modified. Additionally, the malware seems to be using `IPRIP` for some sort of communication with the/a server. 

### Q2 - How do you install the malware? How do you run it after installing it?

Since the malware is a DLL, we can't run it directly. Instead, we have to use the `rundll32` command to run the DLL. 

Before running the DLL, set up the necessary monitoring tools (same as Lab 01).

The way to install the malware is to run the following commands: 

```powershell
rundll32.exe Lab03-02.dll,Install
net start iprip
```

The command line would look something like this:

![Install](3-2-run.png)

### Q3 - What important functions are imported in the DLL?

Hey, we already did this in the basic static analysis :)

### Q4 - Using Process Explorer, find the process under which the malware is running.

Initially, I thought the malware would show up as a separate process on its own, like it did in lab 03-01. Unfortunately, that didn't show up. However, ApateDNS and netcat did show some activity, so I knew the malware was definitely running.

![ApateDNS](3-2-ApateDNS.png)

Earlier on, we saw that the strings of the malware had `svchost.exe` in it. Looking at Process Explorer, I saw that there were multiple instances of `svchost.exe` running. I hovered over each of them to see the command line arguments, and saw an interesting one.

![Process Explorer](3-2-ProcessExplorer.png)

The command line argument is quite similar to what we saw in the malware, and there's suspiciously many services running under this `svchost.exe` process. This is probably the process under which the malware is running.

I also found out you could just search for the DLL name in Process Explorer, and it would show you the process that is running the DLL. 

![Process Explorer Search](3-2-ProcessExplorerSearch.png)

### Q5 - Set a filter in Process Monitor to monitor the operations of the process identified in Q4.

Since we know the process that the malware is running under, we can set a filter for the PID (1196).

### Q6 - What are some host-based indicators?

I got distracted halfway through this lab and left the malware running, so I had way too many logs, and I couldn't filter them without having ProcMon complaining. 

Anyway, I took a Regshot before running the malware. So, I could just take the 2nd shot and compare. There were a few things that caught my eye.

![Regshot](3-2-IPRIP.png)

The malware added the IPRIP service, as expected. It also seems to be adding the dll into the IPRIP parameters, possibly for persistence.

Other than the service, the malware also modified the cryptographic seed, as like in the previous lab.

![Regshot](3-2-seed.png)

### Q7 - What network-based indicators?

Domain: `www.ngeeann-malwareparalysis.com`

Port: `80`

What is the data being transferred? Which protocol?: `HTTP/1.1`

![Netcat](3-2-NetCat.png)

It performs a GET request to /serve.html
