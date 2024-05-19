---
layout: post
title: '[MATT Prac 1] Revision'
date: 2024-05-18 21:34 +0800
author: bowen
categories: [ Malware Analysis Tools and Techniques, MATT Revision Notes ]
img_path: "/assets/img/matt"
---

## Basic Static Analysis

### PEiD

- Packer detection (goofy ahh string = packed)
- Contains a plugin to unpack UPX

### PE View

- Metadata of the binary
- Points of interest:
  - `IMAGE_NT_HEADERS > IMAGE_FILE_HEADER` – Contains the binary's compile time

### Dependency Walker

- Lists all the functions called by a binary
- A binary containing only 1 or 2 exports suggests that it is packed
  - No normal binary is able to operate with 1-2 exported functions
- Take note of file I/O and network related functions

### Strings

- Reveals hardcoded values of the binary
- File paths, web endpoints and other juicy info can be found here

---

## Basic Dynamic Analysis

### Running DLLs

- Syntax: `rundll32.exe <dll>,<entrypoint>`, e.g. `rundll32.exe malware.dll,Install`

### Starting Services

- Syntax: `net start <service>`, e.g. `net start iprip`

### Process Explorer

- Lists all the processes that are running
- Can view in-memory strings, handles and loaded DLLs

### Process Monitor (Procmon)

- List of events that occurred
- Filter allow you to see app-specific events
- Keep a lookout for file I/O and registry edits

### Network Monitor (Netcat)

- Captures network traffic for specific ports
- `nc -l -p <port>` - Usually 80/443

### Registry Monitor (Regshot)

- Monitors registry changes
- Take a snapshot before and after running the binary
