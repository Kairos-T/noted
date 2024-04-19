---
title: "[MATT Wk 1] Practical - Basic Static Analysis"
author: kairos
categories: [ Malware Analysis Tools and Techniques, Practical ]
img_path: /assets/img/matt/basic-static-analysis
---

These labs are based off Week 1 Basic Static Analysis - Practical. The lab exercises can be found [here](https://github.com/mikesiko/PracticalMalwareAnalysis-Labs).


## Lab 1 
For this lab, we will be using the `Lab01-01.dll` and `Lab01-01.exe` files.

### Question 1: When were these files compiled?

To determine the compilation time of the files, use the `PEview` tool.

1. Open `PEview` and load the `Lab01-01.dll` file.
2. Navigate `Lab01-01.dll > IMAGE_NT_HEADERS > IMAGE_FILE_HEADER` to view the Time Date Stamp. This timestamp indicates the time the file was compiled.

    ![PEview Lab01-01.dll](1-1-DLLDateTime.png)

3. Repeat the same steps for the `Lab01-01.exe` file.

    ![PEview Lab01-01.exe](1-1-EXEDateTime.png)


### Question 2: Were these files packed?

The most straightforward way to determine if a file is packed is using `PEiD`. `PEiD` is a tool that detects most common packers and compilers for PE files.

DLL
1. Open `PEiD` and load the `Lab01-01.dll` file.
    ![PEiD Lab01-01.dll](1-1-DLLPEiD.png)
2. Here, we can see:    
   - The "Entrypoint" and "File Offset" values match, meaning that the file is not packed/compressed. 
   - At the bottom, the compiler is shown as "Microsoft Visual C++ 6.0" instead of a packer (E.g. UPX). Hence, the file probably is not packed. 
   - The EP (Entry Point) section is listed as `.text`, the standard entry point. A packed/obfuscated one would probably point to an unpacking stub to decompress/decrypt the hidden `.text` section first.
3. Hence, we can deduce that this DLL is not packed.

EXE
1. Load the `Lab01-01.exe` file into `PEiD` again.
    ![PEiD Lab01-01.exe](1-1-EXEPEiD.png)
2. Here we can come to the same conclusion as the DLL (same explanations).
3. Hence, we can deduce that this EXE Is not packed.
