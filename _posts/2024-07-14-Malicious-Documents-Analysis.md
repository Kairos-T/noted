---
layout: post
title: '[MATT Wk 13] Malicious Documents Analysis'
date: 2024-07-14 23:28 +0800
author: kairos
categories: [ Malware Analysis Tools and Techniques, MATT Notes ]
img_path: "/assets/img/matt/malicious-documents-analysis"
---

# Malicious Documents

Malicious documents are a common vector for malware delivery.

- Can be in forms of Word, Excel, PDF, etc. that contain malicious macros or scripts
- Often used in targeted attacks (spear phishing)

## Office Documents

Two ways of executing code in Office documents:

1. **VBA Macros**
	- Executes on document open or when user clicks `OK` on a prompt
2. **Embedded Shellcode**
	- Executes when the object is loaded
	- Does not require user interaction

### Analysis

Tools like `OfficeMalScanner` can be used to extract macros from Office documents

**OLE (Office 97-2003)**
1. `OfficeMalScanner <file> info` - Checks if macros can be extracted (saves as `.Macros` file), and displays OLE
   structure and other metadata.  
2. `strings2 -nh <file>.Macros` - Extracts strings from the macros

**OOXML (Office 2007+)**
1. `OfficeMalScanner <file> inflate` - Decompresses MS Office documents and extracts `vbaProject.bin` (contains macros)
2. `cd C:\Users\MATT2020\AppData\Local\Temp\DecompressedMsOfficeDocument\word` - Change directory to where `vbaProject.bin` is extracted
3. `OfficeMalScanner vbaProject.bin info` - Extracts macros from `vbaProject.bin`
4. `cd VBAPROJECT.BIN-Macros` - Change directory to where the extracted macros are
5. `strings2 -nh doc` - Extracts strings from the macros
6. [OPTIONAL] Use SSview to examine earlier versions of the VBA macro.

## PDF Documents

PDFs can contain JavaScript/Shellcode that can be used to exploit vulnerabilities in the PDF reader.

### Analysis

1. `pdfid <file>` - Checks for malicious indicators in the PDF
	- Malicious indicators:
    ![Malicious Indicators](indicators.png)
2. `pdf-parser --search /JavaScript <file>` - Locates JavaScript object
	- Output may look like (without my comments ofc):
   
   ``` bash
   obj 31 0                 # Indicates onject number within the PDF file, i.e. object 31, generation 0
     Type:                  # Type of object
     Referencing: 32 0 R    # Object 31 references another object
   
     <<                     # Denotes dictionary object in PDF
        /S /JavaScript      # Key-Value pair - /S action is to execute JavaScript
        /JS 32 0 R          # Key-Value pair - /JS points to object where JavaScript is stored
     >>
   ```
3. `pdf-parser --object 32 --filter --raw <file>` - Extracts JavaScript from object (32 here since it was referenced in the previous step)
4. Examine the JS file extracted and deobfuscate if necessary