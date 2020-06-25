# System Software ─ Background

## 1.3 The Simplified Instructional Computers (SIC)

### 1.3.1 SIC Machine Architecture

**Memory**

+ bytes: 8 bits

+ word: 3 consecutive bytes (32bits)

+ There are total 32,768 (2<sup>15</sup>) bytes in the computer memory

**Registers**

There are five registers, all of which have special uses.

| Mnemonic | Number | Special use                                                                                                    |
|:--------:|:------:| -------------------------------------------------------------------------------------------------------------- |
| A        | 0      | Accumulator; used for arithmetic operations                                                                    |
| X        | 1      | Index register; used for addressing                                                                            |
| L        | 2      | Linkage register; the Jump to Subroutine(JSUB) instruction stores the return address in this register          |
| PC       | 8      | Program counter; contains <u>the address of the next instruction</u> to be fetched for execution. (類似x86中的ISP) |
| SW       | 9      | Status word; contains a variety of information, including a Condition Code(CC)                                 |

**Data Formats**

+ Integers are stored as 24-bit binary numbers

+ 2's complement representation is used for negative values

+ There is no floating-point hardware on the standard version.

**Instruction formats**

All machine instruction on the standard version of SIC have the following 24-bit format

| 8                              | 1   | 15                                              |
|:------------------------------:|:---:|:-----------------------------------------------:|
| --------------opcode---------- | x   | -------------------address--------------------- |

**Addressing Mode**

There are two addressing modes available. <u>Parentheses are used to indicate the contents of a register or a memory location.</u> 

| Mode    | Indication | Target address calculation |
| ------- | ---------- | -------------------------- |
| Direct  | x = 0      | TA = address               |
| Indexed | x = 1      | TA = address + (X)         |

### 1.3.2 SIC/XE Machine Architecture

**Memory**

The memory structure for SIC/XE is the same as that previously described for SIC. However, the maximum memory available on a SIC/XE system is 1MB(2<sup>20</sup> bytes).

**Registers**

| Mnemonic | Number | Special Use                              |
|:--------:|:------:| ---------------------------------------- |
| B        | 3      | Base register; used for addressing       |
| S        | 4      | General working register; no special use |
| T        | 5      | General working register; no special use |
| F        | 6      | Floating-point accumulator(48 bits)      |

**Data Formats**

SIC/XE provides the same data formats as the standard version. In addition, there is a 48-bit floating-point data type with the following format:

| 1   | 11                        | 36                                                 |
|:---:|:-------------------------:|:--------------------------------------------------:|
| s   | ---------exponent-------- | ---------------------fraction--------------------- |

s: signed = 1; unsigned = 0

$$
f\times 2^{(e-1024)}
$$

**Instruction Formats**

> **Format 1** (1 bytes)
> 
> | 8                                 |     |
> |:---------------------------------:| --- |
> | ----------------op--------------- |     |
> 
> **Format 2** (2 bytes)
> 
> | 8                                 | 4   | 4   |
> |:---------------------------------:|:---:|:---:|
> | ----------------op--------------- | r1  | r2  |
> 
> **Format 3** (3 bytes)
> 
> | 6                       | 6                  | 12                                                 |
> |:-----------------------:|:------------------:|:--------------------------------------------------:|
> | -----------op---------- | n\|i\|x\|b\|p\|e\| | ---------------------- disp ---------------------- |
> 
> **Format 4** (4 bytes)
> 
> | 6                       | 6                  | 20                                              |
> |:-----------------------:|:------------------:|:-----------------------------------------------:|
> | -----------op---------- | n\|i\|x\|b\|p\|e\| | ----------------- address --------------------- |

#### Addressing Modes

n = indirect (notation @)

i = immediate (notation #)

x = index (notation  , )

b = base (only for format 3)

p = PC (only for format 3)

e = 0(format 3), 1(format 4)

---

| Type      | n/i/x/b/p/e  | Notation | Calc              | Operand |
| --------- | ------------ | -------- | ----------------- |:-------:|
| Simple    | 1/1/0/0/0/0  | op c     | disp              | (TA)    |
|           | 1/1/0/0/0/1  | +op m    | addr              | (TA)    |
|           | 1/1/0/0/1/0  | op m     | (PC) + disp       | (TA)    |
|           | 1/1/0/1/0/0  | op m     | (B) + disp        | (TA)    |
|           | 1/1/1/0/0/0  | op c, X  | disp + (X)        | (TA)    |
|           | 1/1/1/0/0/1  | +op m, X | addr + (X)        | (TA)    |
|           | 1/1/1/0/1/0  | op m, X  | disp + (X) + (PC) | (TA)    |
|           | 1/1/1/1/0/0  | op m, X  | disp + (X) + (B)  | (TA)    |
| (向下相容)    | 0/0/0/ - - - | op m     | b/p/e/disp        | (TA)    |
| (向下相容)    | 0/0/1/ - - - | op m, X  | b/p/e/disp + (X)  | (TA)    |
| Indirect  | 1/0/0/0/0/0  | op @c    | disp              | ((TA))  |
|           | 1/0/0/0/0/1  | +op @m   | addr              | ((TA))  |
|           | 1/0/0/0/1/0  | op @m    | (PC) + disp       | ((TA))  |
|           | 1/0/0/1/0/0  | op @m    | (B) + disp        | ((TA))  |
| Immediate | 0/1/0/0/0/0  | op #c    | disp              | (TA)    |
|           | 0/1/0/0/0/1  | +op #m   | addr              | (TA)    |
|           | 0/1/0/0/1/0  | op #m    | disp + (PC)       | (TA)    |
|           | 0/1/0/1/0/0  | op #m    | disp + (B)        | (TA)    |

m: memory / c: constant
