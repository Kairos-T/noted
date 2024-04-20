---
title: "[MATT Wk 1] Basic Static Analysis"
author: kairos
categories: [ Malware Analysis Tools and Techniques, MATT Notes ]
img_path: /assets/img/matt/basic-static-analysis
---

## Windows Malware Functions Overview

### Hungarian Notation

| Type/Prefix       | Description                                                                                                                                                                         |
|:------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| WORD (w)          | 16-bit unsigned (positive integer) value                                                                                                                                            |
| DWORD (dw)        | Double-word; 32-bit unsigned value                                                                                                                                                  |
| Handles (H)       | A reference to an object, for resource allocation. <br>E.g.: Opening file, OS returns handle instead of memory/other resources. <br><br> Examples include: HModule, HInstance, HKey |
| Long Pointer (LP) | A direct memory address that points to a specific location in memory; A pointer to another type <br>E.g. LPByte is a pointer to a byte                                              |
| Callback          | A function that will be called by the Windows API <br>E.g. InternetSetStatusCallback passes a pointer to a function that is called when the system has an update of Internet status |

### File API (Host-based Indicators)

Some common Windows API functions used by malware to interact with files:

| Functions                       | Description/Inferences                                                                                                                                                                                                                                                                                      |
|:--------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| CreateFile                      | Creates/opens files, pipes, streams, and I/O devices. <br/>Could indicate file creation, modification, or data exfiltration.                                                                                                                                                                                |
| ReadFile/WriteFile              | Reads/writes to files as a stream. <br/>Suggests data theft, file modification, or payload dropping, etc.                                                                                                                                                                                                   |
| CreateFileMapping/MapViewOfFile | Maps files to memory for direct manipulation. <br/>Implies techniques like in-memory file modifications. <br/><br/>MapViewOfFile: Returns pointer to the base addr of the mapping to access the file. Allows for direct access to the file in memory <br> CreateFileMapping: Loads file from disk to memory |
| - (Pointer to Base Address)     | Allows direct read/write access and navigation within mapped files. <br/>Indicates potential code injection / direct executable modifications in memory.                                                                                                                                                    |

### Windows Registry (Host-based Indicators)

Windows Registry
: The registry is a **hierarchical database** that stores configuration settings and options on Windows operating systems.

Malware often uses the Registry to store **configurations**, **persistence mechanisms**, and other information.
> The Windows Registry Editor can be found by searching for "Regedit" in the Windows search function

![Windows Registry](windows-registry.png)

**Registry Organisation**
The registry is organised into:
- **Root Key/Hive**: Top-level sections of the registry. (HKEY is used to denote a root key)
- **Subkeys**: Subsections of the registry
- **Keys**: Folder-like objects that can contain other folders or key-value pairs
- **Value Entry**: Ordered pairs of name-data that are stored in keys
- **Value Data**: The data stored in a registry entry

| Root Key                   | Description/Inferences                                                                                                                                                                                               |
|:---------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| HKEY_CLASSES_ROOT          | Contains file associations and related data (Tells the system how to handle file types based on their extensions). <br/> Might be hijacking file associations, shell extensions, or program behavior modifications.  |
| HKEY_CURRENT_USER (HKCU)   | Stores user-specific settings. <br/>Suggests user preference/autorun modifications or storing user data.                                                                                                             |
| HKEY_LOCAL_MACHINE (HKLM)  | Stores global local machine settings. <br/>Indicates persistence, system modifications, or privilege escalation attempts.<br/> E.g.:`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Shell Folders`          |
| HKEY_USERS                 | Contains user profile data. <br/>Could target specific user profiles, modify default settings, or persist across user sessions.                                                                                      |
| HKEY_CURRENT_CONFIG        | Stores hardware profile settings. <br/>Might be gathering system information or modifying hardware configurations.                                                                                                   |

**Common Registry Functions**
Some common registry functions used by malware (_* indicates that the function is given in the slides. The rest are extra and good to know!_):

| Functions          | Description/Inferences                                                                                                                                                       |
|:-------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| RegOpenKeyEx*      | Opens a key in the registry for querying/editing. <br/>Indicates registry key access, modification, or enumeration.                                                          |
| RegSetValueEx*     | Adds new value to the registry and sets its data <br/>Implies registry value modification, persistence, or configuration changes.                                            |
| RegGetValue*       | Retrieves the type and data for the specified value entry (opened registry key). <br/>Suggests reading registry values for configuration, persistence, or data exfiltration. |
| RegCreateKeyEx     | Creates a new key or opens an existing key. <br/>Could indicate new registry key creation, persistence, or configuration changes.                                            |
| RegDeleteKey       | Deletes a subkey and its values. <br/>Might be used to remove traces, configurations, or persistence mechanisms.                                                             |

> A way to detect registry modifications is taking a snapshot before and after executing a program using tools like RegShot. Compare the snapshots to identify changes. This will be talked about more in dynamic analysis.
{: .prompt-info }

### Networking API (Network-based Indicators)

Socket
: An endpoint for communication between two machines over a network. Programs uses sockets to send and receive data.

**Common Socket API Functions**

| Function  | Description/Inferences                                                                                         |
|:----------|:---------------------------------------------------------------------------------------------------------------|
| socket    | Creates a socket. <br/> Might be establishing network communication capabilities.                              |
| bind      | Attaches socket to a port.<br/>Could be setting up a listening port for receiving commands/data.               |
| listen    | Prepares socket for incoming connections.<br/>Might be waiting for connections from Command & Control server.  |
| accept    | Opens and accepts incoming connection.<br/>Could be accepting connections from attackers/infected systems.     |
| connect   | Opens connection to remote socket.<br/> Might be connecting to Command & Control server.                       |
| recv      | Receives data from remote socket.<br/>Could be receiving data/commands from remote server.                     |
| send      | Sends data to remote socket.<br/>Might be sending stolen data or system details to remote server.              |

Sniffing
: The act of capturing network traffic. Malware can use sniffing to steal sensitive information or hide its communication.

**Common Sniffing Functions**

| Function                   | Description/Inferences                                                                                                                                                                                                                                    |
|:---------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| WSASocket() / socket()     | Creates a raw socket (allows application to interact directly with network packets at the IP level instead of TCP/UDP). <br/>Might be used for sniffing when combined with other suspicious functions.                                                    |
| bind()                     | Binds socket to a specific port. <br/>Could be setting up a listening port for sniffing or network communication.                                                                                                                                         |
| WSAloctl() / ioctlsocket() | Puts NIC into promiscuous mode <br/>-> NIC captures, and sends to the host's CPU, all frames/packets passing through the network, whether or not it is addressed to that specific NIC<br/>Indicates intent to sniff network traffic.                      |


### WinINET API (Network-based Indicators)

WinINET API
: A Windows API that implements high-level protocols (HTTP/FTP, etc.).

Downloaders
: Malware that downloads additional payloads from the internet. They often use WinINET functions to download the files.

> Malware might need to download additional payloads due to resource limitations (E.g. phishing emails with small payloads), or to evade detection (Initial malware sample has a smaller footprint, appearing less suspicious, evading static analysis/AV detection).
{: .prompt-info }

**Common WinINET (/Downloader) Functions**

| Function                   | Description/Inferences                                                                                           |
|:---------------------------|:-----------------------------------------------------------------------------------------------------------------|
| InternetOpen               | Initialises a connection to the internet. <br/>Might be setting up a connection to download additional payloads. |
| InternetOpenUrl            | Opens a URL on the internet.                                                                                     |
| InternetReadFile           | Reads data from a URL.                                                                                           |
| URLDownloadToFile          | Downloads a file from a URL. <br/>Indicates downloading files from the internet.                                 |
| ShellExecute() / WinExec() | Executes a program. <br/>Could be executing downloaded payloads.                                                 |

### Process Manipulation (Host-based Indicators)

Process Manipulation
: - Malware often manipulates processes to **evade detection** (from firewall/user) by creating new child processes (more info below).
  - `CreateProcess` is a common function used to create new processes. It takes in the parameter `STARTUPINFO`, which includes a handle to standard input, output and error messages. 
  - A malware can create a new process with the I/O handles bound to a socket through the attacker's machine, effectively providing a remote shell.

Parent-Child Process
: - Malware can create a new process that is a child of the malware process to execute malicious activities.
: - An example from [this site](https://study-harder.qinguan.me/np/ict/matt/2-windows-malware/#downloaders):
      - A malware (disguised as Minecraft) is downloaded onto the victim's computer
      - While the "application" (also known as the parent process) runs, background processes (also known as child process) are also created
      - These background processes can be malicious and could be deleting the victim data 

> Many security solutions monitor the behaviour of the parent process / existing processes only. By creating a child process, the malware can evade detection.
{: .prompt-info }

### Key Loggers (Host-based Indicators)

Key Loggers
: - Malware that records keystrokes. 
  - Keyloggers can be used to spy on users or steal sensitive information like passwords.

**Methods of Key Logging**
1. **Hooking**
    - Hook: A mechanism that intercepts events, like keystrokes.
    - Hook Procedure: A function that intercepts a particular type of event. It can act on each event it receives, and modify or discard it 
    - API:
   
    | Function         | Description/Inferences                                                                                                                                                                                                  |
    |:-----------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | SetWindowsHookEx | When called with `WH_KEYBOARD`, it installs a hook procedure to relay the event to a (malicious) function each time a key is pressed.<br>When called with `WH_MOUSE`, it relays mouse messages (left-click/right-click) |
   
2. **Polling**
    - Loop and Poll: Continuously checking the state of each key on the keyboard within a long-running loop.
    - API:
   
    | Function         | Description/Inferences                                                  |
    |:-----------------|:------------------------------------------------------------------------|
    | GetAsyncKeyState | Retrieves the state of a specific key. <br>Used for polling keystrokes. |
