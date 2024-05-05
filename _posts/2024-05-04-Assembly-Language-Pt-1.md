---
layout: post
title: '[MATT Wk 3] Assembly Language Pt. 1'
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
  {: .prompt-info }

## Assembly Instructions

### Syntax

Instruction Format:

|            | Mnemonic                                             | Destination Operand                                             | Source Operand                                               |
|:-----------|:-----------------------------------------------------|:----------------------------------------------------------------|:-------------------------------------------------------------|
| Definition | Operation to be performed, e.g., `mov`, `add`, `sub` | Destination register/memory location where the result is stored | Source register/memory location where the value is read from |
| Example    | `mov`                                                | `eax`                                                           | `0x42`                                                       |

Each instruction corresponds to `opcodes`, which are the machine code representation of the instruction.

| Instruction | Opcode        |
|:------------|:--------------|
| `mov ecx`   | `B9`          |
| `0x42`      | `42 00 00 00` |

> x86 architecture uses little-endian format (LSB stored first). However, during network communication, big-endian (MSB stored first) is used. 
> <br>Conversion Examples:<br> `0x42` -> `42 00 00 00` (little-endian) -> `00 00 00 42` (big-endian)
> <br>`0x0100007F` (little-endian) -> `0x0100007F` (big-endian)
{: .prompt-info }

**Operand Types**

- **Immediate Operands**: Constant value, e.g., `0x42`. 

  E.g.:

  ```asm
  mov     eax, 0x42       ; Move 0x42 into eax
  add     eax, 0x10       ; Add 0x10 to eax
  sub     eax, 0x5        ; Subtract 0x5 from eax
  ```

- **Register Operands**: Data stored in a CPU register (e.g., `%eax` denotes the `eax` register). More info in the next section.
- **Opcode Operands**: Machine code representation of the instruction (e.g., `B9`)
- **Memory Address Operands**: Specific location in memory denoted by register, value or equation between brackets (E.g.: `[eax]`).

### Registers 

Register
: Small, high-speed data storage locations in the CPU for temporary storage and manipulation of data. 

![Registers](registers.png)

**Types of Registers:**
- **General Register**: Used by CPU to perform arithmetic, logical, etc. operations during execution
- **Segment Register**: Track memory sections (e.g., code, data, stack)
- **Status Flags**: Makes decisions based on the result of operations (e.g., zero flag, carry flag)
- **Instruction Pointer (IP)**: Stores address of the next instruction to be executed

**General Purpose Registers:**

Some instructions use specific registers by definition
- EDX: Division 
- EAX: Multiplication, Holding return value for function call
- ESP, EBP: Function call/return
- ESI, EDI and ECX: Used in repeat instructions
- ESI, EDI: Store memory addresses

Referencing Registers

![Register Reference](referencing.png)

> There is no direct way to reference the upper 16 bits as it is unnecessary. ([Ref](https://stackoverflow.com/questions/28429609/why-arent-the-higher-16-bits-in-eax-accessible-by-name-like-ax-ah-and-al))
{: .prompt-info }
