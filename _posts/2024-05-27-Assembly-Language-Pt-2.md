---
layout: post
title: '[MATT Wk 4] Assembly Language Pt. 2'
date: 2024-05-27 23:57 +0800
author: kairos
categories: [ Malware Analysis Tools and Techniques, MATT Notes ]
img_path: "/assets/img/matt/assembly"
---

# Conditionals

Conditionals
: Instructions that perform comparisons; based on the result, the program will execute different instructions.
- Most common conditional instructions: `test` and `cmp`

TEST Instruction
: Performs bitwise `AND` operation between two operands without modifying them, i.e. only the status flags (most
commonly Zero Flag) are set according to the result. The result is then discarded.
- Often used to check for NULL values or the presence of a specific bit in a register.
- If two operands have any bits in common, the Zero Flag is set to `0`,

  ```assembly
  ; Example of ZF = 0 
  0 0 1 0 0 1 0 1  ; <- input value
  0 0 0 0 1 0 0 1  ; <- test value
  0 0 0 0 0 0 0 1  ; <- result: ZF = 0
  ```

- If two operands have no bits in common, the Zero Flag is set to `1`.

  ```assembly
  ; Example of ZF = 1
  0 0 1 0 0 1 0 0  ; <- input value
  0 0 0 0 1 0 0 1  ; <- test value
  0 0 0 0 0 0 0 0  ; <- result: ZF = 1
  ```
- E.g.

  ```assembly
  mov eax, 1      ; load 1 into eax
  mov ebx, 2      ; load 2 into ebx 
  test eax, ebx   ; perform bitwise AND operation between eax and ebx
  jz func         ; if ZF = 1, jump to func
  jmp func        ; else, jump to func
  ```

CMP Instruction
: Performs bitwise `SUB` operation by subtracting the second operand from the first operand, without modifying the
operands. The status flags are set according to the result:

| cmp dst, src | ZF | CF | Example                      |
|:-------------|:---|:---|:-----------------------------|
| dst = src    | 1  | 0  | `mov al, 5` <br> `cmp al, 5` | 
| dst > src    | 0  | 0  | `mov al, 6` <br> `cmp al, 5` | 
| dst < src    | 0  | 1  | `mov al, 4` <br> `cmp al, 5` |

## Jumps

Branching
: Changing the flow of execution based on the results of the branches of a program. Most common way is through `jump`
instruction.

Jump Instructions
: Causes the next instruction executed to be the one specified by the `jmp` instruction.
- There is no `if` statement in assembly language; instead, conditional jumps are used to implement branching.

### Specific Flag-Based Jumps

| Instruction | Description         | Flags Checked |
|:------------|:--------------------|:--------------|
| `JZ loc`    | Jump if Zero        | ZF = 1        |
| `JNZ loc`   | Jump if Not Zero    | ZF = 0        |
| `JC loc`    | Jump if Carry       | CF = 1        |
| `JNC loc`   | Jump if No Carry    | CF = 0        |
| `JO loc`    | Jump if Overflow    | OF = 1        |
| `JNO loc`   | Jump if No Overflow | OF = 0        |
| `JS loc`    | Jump if Sign        | SF = 1        |
| `JNS loc`   | Jump if No Sign     | SF = 0        |
| `JP loc`    | Jump if Parity      | PF = 1        |
| `JNP loc`   | Jump if No Parity   | PF = 0        |

> The jump occurs if the previous instruction sets the specified flag to the expected value.
{: .prompt-info}

### Equality-Based Jumps

| Instruction | Description                                          | Conditions Checked |
|:------------|:-----------------------------------------------------|:-------------------|
| `JE loc`    | Jump if Equal                                        | leftOp = rightOp   |
| `JNE loc`   | Jump if Not Equal                                    | leftOp ≠ rightOp   |
| `JCXZ loc`  | Jump if CX is Zero (or register between `J` and `Z`) | CX = 0             |
| `JECXZ loc` | Jump if ECX is Zero                                  | ECX = 0            |

### Comparison-Based Jumps

#### Unsigned Comparison

| Instruction | Description             | Conditions Checked |
|:------------|:------------------------|:-------------------|
| `JA loc`    | Jump if Above           | leftOp > rightOp   |
| `JNBE  loc` | Jump if Not Below/Equal | leftOp > rightOp   |
| `JAE loc`   | Jump if Above/Equal     | leftOp ≥ rightOp   |
| `JNB loc`   | Jump if Not Below       | leftOp ≥ rightOp   |
| `JB loc`    | Jump if Below           | leftOp < rightOp   |
| `JNAE loc`  | Jump if Not Above/Equal | leftOp < rightOp   |
| `JBE loc`   | Jump if Below/Equal     | leftOp ≤ rightOp   |
| `JNA loc`   | Jump if Not Above       | leftOp ≤ rightOp   |

#### Signed Comparison

| Instruction | Description               | Conditions Checked |
|:------------|:--------------------------|:-------------------|
| `JG loc`    | Jump if Greater           | leftOp > rightOp   |
| `JNLE loc`  | Jump if Not Less/Equal    | leftOp > rightOp   |
| `JGE loc`   | Jump if Greater/Equal     | leftOp ≥ rightOp   |
| `JNL loc`   | Jump if Not Less          | leftOp ≥ rightOp   |
| `JL loc`    | Jump if Less              | leftOp < rightOp   |
| `JNGE loc`  | Jump if Not Greater/Equal | leftOp < rightOp   |
| `JLE loc`   | Jump if Less/Equal        | leftOp ≤ rightOp   |
| `JNG loc`   | Jump if Not Greater       | leftOp ≤ rightOp   |

### Looping using Jumps

Example:

```assembly
      mov eax, 10     ; load 10 into eax, i.e. loop counter
loop: dec eax         ; decrement eax
      cmp eax, 0      ; compare eax with 0
      jnz loop        ; if ZF = 0, jump to loop
```

## Repeat Instructions

- Used for manipulating data buffers, i.e. processing multi-byte data.
  - Most commonly in array of bytes, but can be single or double words.
- Registers must be initialized before using repeat instructions.
- `rep` is the prefix for repeat instructions.

Registers Used
- `ECX` register: Loop variable counter
- `ESI` register: Source index register
- `EDI` register: Destination index register

Instructions

| Instruction                 | Description                      | Conditions Checked  |
|:----------------------------|:---------------------------------|:--------------------|
| `rep <op>`                  | Repeat until `ECX` = 0           | `ECX` = 0           |
| `repe <op>` / `repz <op>`   | Repeat until `ECX` = 0 or ZF = 0 | `ECX` = 0 or ZF = 0 |
| `repne <op>` / `repnz <op>` | Repeat until `ECX` = 0 or ZF = 1 | `ECX` = 0 or ZF = 1 |

Other Repeat Instructions

![Repeat Instructions](repeat-instructions.png)

> Most common data buffer manipulation instructions are `movsx`, `cmpsx`, `stosx`, and `scasx`, where `x`=b, w, or d 
  (byte, word, or double word). <br> A summary: <br>
  `movsb`: Copies a sequence of bytes, with size defined by `ECX`<br>
  `cmpsb`: Compares a sequence of bytes to determine if they are equal<br>
  `stosb`: Stores a byte in the destination buffer<br>
  `scasb`: Scans a byte for a single value.<br>
{: .prompt-info}

### Looping with Repeat Instructions

Example:

```assembly
mov esi, src          ; load source address into esi
mov edi, dest         ; load destination address into edi
mov ecx, byte_count   ; load byte count into ecx
cld                   ; clear direction flag, i.e. increment esi and edi. 
; Alternatively, use `std` to decrement esi and edi.
rep movsb             ; copy byte_count bytes from src to dest
```

## Function Calls

### The Stack

Stack
: A region of memory used to store data temporarily for functions, local variables, and flow control.
- Push-Pop operations to store and retrieve data from the stack.
- LIFO (Last In, First Out) data structure.
- `ESP` register points to the top of the stack.
- `EBP` register is used as a backup for `ESP`; does not change during function calls. Points to the start of frame.

**Push Operation**
- Decrements stack pointer `ESP` by 4 bytes (32-bit push) and copies value into location pointed by `ESP`.
- Stack grows downwards, and the area under ESP is available for use unless overflowed.
- Syntax:
  - `push r/m16` or `push r/m32` - Register of memory; i.e. `push eax` or `push [eax]`
  - `push imm32` - Immediate value; i.e. `push 0x12345678
- ![Stack Push Operation](stack-push.png)

**Pop Operation**
- Copies value at stack[ESP] into register/variable, then increments `ESP` by 2 or 4 bytes, depending on attribute of 
  operand receiving the data.
- Syntax similar to `push` operation.

### Functions

Function
: A block of code within a function that performs a specific task.
- A program would `call` a function to transfer execution flow control to it.
- When function completes execution, control is `ret`urned to the next program instruction after call.

**Function Structure**
1. **Prologue**: A few lines of code at the start of function to prepare stack and registers for use within function.
2. **Function Body**: The main code block that performs the task.
3. **Epilogue**: Restores stack and registers to state before the function was called.

> Prologue and Epilogue can contain BOF protection code
{: .prompt-info}

**Interacting with Stack**
1. Arguments placed on stack with `push` instruction.
2. Function called using `call <memory_location>` (starting address of function).
3. EIP pushed onto stack and set to `<memory_location>`.
4. Space allocated on stack for local variables, and `EBP` is pushed onto stack.
5. Function executes. After completion, stack is restored using epilogue.
6. Function returns using `ret` instruction. Stack adjusted to remove function arguments (unless they'll be used again).

## C vs ASM

- Malware is often written in C.
- Main method: `int main(int argc, char **argv)`.
  - `argc`: Number of arguments passed to program, including program name.
  - `argv`: Array of arguments passed to program containing command-line arguments.
  - `argc` and `argv` determined at runtime.

**Main Method**

![Main Method](main-method.png)

![Prologue and Epilogue](prologue-epilogue.png)

Refer to slides for the other comparison examples :")