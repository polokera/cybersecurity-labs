
# Parcel Check - Software Reverse Engineering

## Overview

This lab involved analysing a stripped x86-64 Linux executable
called `parcel-check.bin`.

The objective was to understand how the program processes
parcel-related input and determines whether the input is valid.

The analysis was performed using static analysis techniques
and the Ghidra reverse-engineering tool.

## Objectives

- Identify the executable format and architecture.
- Inspect strings and imported functions.
- Build a function and cross-reference map.
- Identify input-handling and validation logic.
- Cross-check decompiled code against disassembly.
- Reconstruct the program's validation flow.

## Environment

| Component | Details |
|---|---|
| Operating system | kali linux |
| Architecture | x86-64 |
| Analysis tool | Ghidra |
| Additional tools | file, sha256sum, readelf, strings |

## 1. Initial File Inspection

### 1.1 Verify File Integrity

Command:

```bash
sha256sum parcel-check.bin
```

Purpose:

Calculate the SHA-256 hash of the executable and compare
it with the hash provided by the lecturer.

Expected result:

The calculated hash matches the expected hash.

### 1.2 Identify the File

Command:

```bash
file parcel-check.bin
```

Purpose:

Identify the file format, architecture and executable type.

Findings:

- File format: ELF
- Architecture: x86-64
- Executable type: PIE
- Linking: static

### 1.3 Inspect the ELF Header

Command:

```bash
readelf -h parcel-check.bin
```

Purpose:

Inspect the ELF header to confirm the executable's
architecture and other structural information.

Findings:

- little endian
- Entry point address: 0x11b0

### 1.4 Extract Printable Strings

Command:

```bash
strings -n 4 parcel-check.bin
```

Purpose:

Identify printable strings that may reveal program
functionality, input prompts or result messages.

Findings:

- parcel accepeted
- parcel denied
- strlen

Interpretation:

- suggests that theres a logical checks within the code.
- strlen suggests that a length check is oone of the checks 

**all this is yet to be confirmed**


## 2. Ghidra Analysis and Investigation notes

### 2.1 Identify relevant strings

**Action:** Window -> Defined strings -> Search for relevant strings
**Observation:** Two strings were of interest.
- parcel accepted
- parcel denied
**Reasoning:** These strings are probably in the program's validation function.

### 2.2 Follow string refeerence
**Action:** Selected each string and checked on the address to view where in the program they are refernced.
**Observation:** Both strings were referenced by the function FUN_001010e0.
**Reasoning:** The function references both the acceptance and rejection messages. This sugests that it is responsible for validating the parcel code.

### 2.3 Examine the function

**Action:** Click on the address next to the name of the function FUN-....
**oBSERVATION:** It opens the function in assembly in code browser with decompiled C on the right.

#### 2.3.1 the steps taken by the function

1. checks number of arguments
2. input length 
3. checks the starting 4 characters
4. checks for digits
5. checks if the last character is a specific character


## 3. Validation Flow

```text
Parcel code
  |
  v
Check code length = 8 and code starts with PCL-
  |
  +---- not equal to 8 ---> return 2 usage error
  |
  v
 check if the characters from index 4 to 6 are digits
  |
  + --- if any are not digits ---> return 1 deny parcel
  |
  v
  Check if last character is a letter obtained from adding the previous 3 digits then the answer module 26 then use that to get an alphabet letter
  |
  + --- if its not equal to expected letter ---> parcel denied
  |
  v
  Parcel Accepted 
```

## 4. Key Findings

the parcel code is in the format PCL-NNNX
where N is any digit 
and X is a letter calculated from the 3 digits

## 5. Challenges and Lessons Learned

### Challenges

- Reading C code

### Lessons Learned

- Ghidra usage and how it shows both disassembled and decompiled code. How the variables are not descriptive but random.
- Check for interesting strings using strings -n 4 or window --> Defined strings 

## Disclaimer

This repository contains my own notes and analysis of an
authorised educational reverse-engineering exercise.

The private course binary and restricted course resources
are not included.
