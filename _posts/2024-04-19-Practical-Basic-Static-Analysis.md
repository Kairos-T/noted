---
title: "[MATT Wk 1] Practical - Basic Static Analysis"
author: kairos
categories: [ Malware Analysis Tools and Techniques, MATT Practical ]
img_path: /assets/img/matt/basic-static-analysis
---

These labs are based off Week 1 Basic Static Analysis - Practical. The lab exercises can be
found [here](https://github.com/mikesiko/PracticalMalwareAnalysis-Labs). Note that these are my personal solutions and
may not be 100% accurate.

## Lab 1

For this lab, we will be using the `Lab01-01.dll` and `Lab01-01.exe` files.

### Question 1: When were these files compiled?

To determine the compilation time of the files, use the `PEview` tool.

1. Open `PEview` and load the `Lab01-01.dll` file.
2. Navigate `Lab01-01.dll > IMAGE_NT_HEADERS > IMAGE_FILE_HEADER` to view the Time Date Stamp. This timestamp indicates
   the time the file was compiled.

   ![PEview Lab01-01.dll](1-1-DLLDateTime.png)

3. Repeat the same steps for the `Lab01-01.exe` file.

   ![PEview Lab01-01.exe](1-1-EXEDateTime.png)

### Question 2: Were these files packed?

The most straightforward way to determine if a file is packed is using `PEiD`. `PEiD` is a tool that detects most common
packers and compilers for PE files.

DLL

1. Open `PEiD` and load the `Lab01-01.dll` file.
   ![PEiD Lab01-01.dll](1-1-DLLPEiD.png)
2. Here, we can see:
   - The "Entrypoint" and "File Offset" values match, meaning that the file is not packed/compressed.
   - At the bottom, the compiler is shown as "Microsoft Visual C++ 6.0" instead of a packer (E.g. UPX). Hence, the file
     probably is not packed.
   - The EP (Entry Point) section is listed as `.text`, the standard entry point. A packed/obfuscated one would probably
     point to an unpacking stub to decompress/decrypt the hidden `.text` section first.
3. Hence, we can deduce that this DLL is not packed.

EXE

1. Load the `Lab01-01.exe` file into `PEiD` again.
   ![PEiD Lab01-01.exe](1-1-EXEPEiD.png)
2. Here we can come to the same conclusion as the DLL (same explanations).
3. Hence, we can deduce that this EXE Is not packed.

### Question 3: What could be the functionalities of this malware, the exe and the dll, based on the imports?

Firstly, we can use `Dependency Walker` (note: I'm using a rewrite of depends.exe
from [GitHub](https://github.com/lucasg/Dependencies)) to view the imports and exports of the files.

![Dependency Walker](1-1-DependencyWalker.png)
_DLL on the left, EXE on the right._

From the imports of the DLL:

- `sleep`:  Pauses execution for a specified time period. Malware uses `sleep` to achieve persistence, evade detection (
  E.g.: Stall execution to outlast sandbox's analysis duration, to prevent it from being analysed), or backdoors.
- `CreateMutexA` and `OpenMutexA`: Creates and opens a mutex object, respectively. Mutexes are used to ensure that only
  one instance of the malware runs at a time.
- `WS2_32.dll`: Windows Socket API. This implies network communication capabilities. The malware could be communicating
  with a C2 server or downloading additional payloads.

The EXE is more interesting. From the imports of the EXE:

- `MapViewOfFile` and `CreateFileMappingA`: Maps files to memory for direct manipulation. This implies techniques like
  in-memory file modifications.
- `CreateFileA`: Creates or opens files on the system.
- `FindClose`, `FindNextFileA`, `FindFirstFileA`: Used to recursively search directories for files. This could be used
  for data exfiltration or to locate specific files on the system.
- `CopyFileA`: Copies a file

From the imports only, we can infer that the malware likely searches the file system for files and copies them.

If we were to look at the strings of both files, we could gain more insight into the malware's functionalities.

DLL:

```
!This program cannot be run in DOS mode.
Rich
.text
`.rdata
@.data
.reloc
(TRUNCATED FOR BREVITY)
CloseHandle
Sleep
CreateProcessA
CreateMutexA
OpenMutexA
KERNEL32.dll
WS2_32.dll
strncmp
MSVCRT.dll
free
_initterm
malloc
_adjust_fdiv
exec
sleep
hello
127.26.152.13
```

Here, we can see an IP address, `127.26.152.13`, which could indicate potential C2 communication. There are interesting
strings too, including `exec`, which could mean that the malware might be receiving commands to execute.

EXE:

```
!This program cannot be run in DOS mode.
Richm
.text
`.rdata
@.data
(TRUANCATED FOR BREVITY)
CloseHandle
UnmapViewOfFile
IsBadReadPtr
MapViewOfFile
CreateFileMappingA
CreateFileA
FindClose
FindNextFileA
FindFirstFileA
CopyFileA
KERNEL32.dll
malloc
exit
MSVCRT.dll
_exit
_XcptFilter
__p___initenv
__getmainargs
_initterm
__setusermatherr
_adjust_fdiv
__p__commode
__p__fmode
__set_app_type
_except_handler3
_controlfp
_stricmp
kerne132.dll
kernel32.dll
.exe
C:\*
C:\windows\system32\kerne132.dll
Kernel32.
Lab01-01.dll
C:\Windows\System32\Kernel32.dll
WARNING_THIS_WILL_DESTROY_YOUR_MACHINE
@jjj
@jjj
@jjj
@jjj
```

Here, we mainly see the same imports as before. However, we see something interesting:

- `C:\windows\system32\kerne132.dll`: Instead of `kernel32.dll`, it's `kerne132.dll`, which is an attempt to disguise
  the file as a legitimate system file. This could mean several things, and would require further analysis into the dll
  file to understand its purpose.

In essence, the malware seems to be a backdoor that communicates with a C2 server, to receive commands to execute, and
can copy files from the system.

## Lab 2

For this lab, we will be using the `Lab01-02.exe` file.

### Question 1: Are the files packed? If so, what are the indicators and how could you unpack it?

As before, we can use `PEiD` to determine if the file is packed.

1. Open `PEiD` and load the `Lab01-02.exe` file.

   ![PEiD Lab01-02.exe](1-2-PEiD.png)
2. Here, we can see a few indicators that the file is packed:
  - The "Entrypoint" and "File Offset" values do not match, meaning that the file might be packed.
  - The EP sector is listed as `UPX1`, which is a commonly used packer.

To add on, we can use `PEview` to further confirm that the file is packed.

1. Open `PEview` and load the `Lab01-02.exe` file.
2. Navigate to `Lab01-02.exe > IMAGE_SECTION_HEADERS_UPX0` to view the Virtual Size (the space that will be allocated to
   this segment) and the Raw Size (the space it takes up on the disk).

   ![PEview Lab01-02.exe](1-2-PEview.png)
3. While some discrepancies between the two values are normal, here we see that the virtual size is 4000, and the raw
   size is 0. This probably means that the packer will unpack the file at the allocated section.

There are other ways to determine if the file is packed, like looking at the imports (few imports = packed), or the
strings (obfuscated = packed), but the above already tells us that the file is packed.

So, yes, the file is packed, and we know that it is packed with UPX.

To unpack the file, we can use the UPX unpacker plugin for `PEiD` that is available on the MATT lab machines.
Unfortunately, I don't have it on my sandbox, but it should be found here:

![PEiD Unpacker](1-2-PEiD-Unpacker.png)

Alternatively, you could manually unpack the file using the [UPX tool](https://github.com/upx/upx).

```powershell
C:\Users\kairo\...\upx.exe -o Lab01-02-Unpacked.exe -d Lab01-02.exe
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2024
UPX 4.2.2       Markus Oberhumer, Laszlo Molnar & John Reiser    Jan 3rd 2024

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
     16384 <-      3072   18.75%    win32/pe     Lab01-02-Unpacked.exe

Unpacked 1 file.
```

### Question 2: What are the imports of the malware?

To view the imports of the malware, we can use `Dependency Walker`.

![Dependency Walker](1-2-kernel32.png)

The malware imports from `kernel32.dll`, `advapi32.dll`, `MSVCRT.dll`, and `WININET.dll`.

### Question 3: What important functions are imported in the DLL?

`advapi32.dll`:
![advapi32.dll](1-2-advapi32.png)

- `CreateServiceA`, `StartServiceCtrlDispatcherA` and `OpenSCManagerA`: Creates and starts a service (background
  processes without a user interface), respectively. This could be used to carry out malicious activities.

`WININET.dll`:
![WININET.dll](1-2-wininet.png)

- `InternetOpenUrlA`, `InternetOpenA`: Opens a URL and an internet connection, respectively. This could be used to
  download additional payloads or communicate with a C2 server.

### Question 4: What could be the functionality of the malware based on the import of the malware?

The malware might be a backdoor as well, as it imports functions to create background services, and initialises internet connections. This might be used to communicate with a C2 server, download additional payloads, open a webpage to a site or other malicious activities. We can't be very sure what the malware does without further analysis, but these are some possibilities.

### Question 5: What could be some host-based indicators? 

Apart from the stuff above, we can further look at the strings of the malware to gain more insight into its functionalities.

```
FLARE-VM Sun 04/21/2024 12:37:10.53
C:\Users\kairo\OneDrive\Documents\PMA Labs\PracticalMalwareAnalysis-Labs\Practical Malware Analysis Labs\BinaryCollection\Chapter_1L>strings Lab01-02-Unpacked.exe
!This program cannot be run in DOS mode.
Rich
.text
`.rdata
@.data
(TRUNCATED FOR BREVITY)
KERNEL32.DLL
ADVAPI32.dll
MSVCRT.dll
WININET.dll
SystemTimeToFileTime
GetModuleFileNameA
CreateWaitableTimerA
ExitProcess
OpenMutexA
SetWaitableTimer
WaitForSingleObject
CreateMutexA
CreateThread
CreateServiceA
StartServiceCtrlDispatcherA
OpenSCManagerA
_exit
_XcptFilter
exit
__p___initenv
__getmainargs
_initterm
__setusermatherr
_adjust_fdiv
__p__commode
__p__fmode
__set_app_type
_except_handler3
_controlfp
InternetOpenUrlA
InternetOpenA
MalService
Malservice
HGL345
http://www.malwareanalysisbook.com
Internet Explorer 8.0
```

The main host-based indicator (apart from the ones mentioned) I can see is:
- MalService: Could be the name of the service that the malware creates.

### Question 6: What could be some network-based indicators?

With reference to the strings above, we can see that the malware connects to `http://www.malwareanalysisbook.com`. 
