---
layout: post
title: '[MATT W3] Assembly Language Pt. 1'
date: 2024-05-04 17:25 +0800
author: kairos
categories: [ Malware Analysis Tools and Techniques, MATT Notes ]
img_path: "/assets/img/matt/assembly"
---

## Introduction

Assembly language (ASM) is a low-level programming language, a representation of machine language. It is obtained from
the disassembly of machine code.

## x86 Architecture Terminologies

### CPU and its Components

![CPU](cpu.png)

- **CPU**: The brain of the computer, executes instructions.
- **Registers**: Small, high-speed storage locations in the CPU. Stores data and addresses temporarily during execution.
- **ALU (Arithmetic Logic Unit)**: Performs arithmetic (+, -, x, etc.) and logical operations on data (AND, OR, NOT,
  etc.).
- **Control Unit**: Fetches instructions from memory, decodes them, and controls the execution of the instructions by
  coordinating the other components of the CPU.
- **Main Memory (RAM)**: Stores data and instructions for the CPU to access and execute. Provides temporary storage for
  programs and data.
- **I/O Devices**: Allow data and instructions to be transferred between CPU and external devices.

Flow of Execution:

- CPU fetches instructions and data from RAM.
- ALU performs operations on the data with registers
- Results are stored back in RAM / output devices.

### RAM and its Components

![RAM](ram.png)

- **Stack**: Region of memory for storing temporary data (e.g., func calls, local vars, return addr, etc.). LIFO
  structure.
- **Heap**: Region of memory for dynamic memory allocation, managed by the OS/program. Stores data and objects that are
  created and destroyed during execution.
- **Code**: Holds machine code instructions for the program.
- **Data**: Stores global and static variables used by the program.

> The order of these sections may vary depending on the OS, compiler, and program due to memory management strategies.
> {: .prompt-info }






