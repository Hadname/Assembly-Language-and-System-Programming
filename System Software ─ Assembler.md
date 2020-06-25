# System Software ─ Assembler

## 2.1 Basic Assembler Functions

**Assembler directives**

| Mnemonics | Description                                                                                                      |
|:---------:| ---------------------------------------------------------------------------------------------------------------- |
| START     | Specify name and starting address for the program.                                                               |
| END       | Indicate the end of the source program and (optionally) specify the first executable instruction in the program. |
| BYTE      | Generate character or hexadecimal constant, occupying as many bytes as needed to represent the constant.         |
| WORD      | Generate one-word integer constant.                                                                              |
| RESB      | **Res**erve the indicated number of **b**ytes for a data area.                                                   |
| RESW      | **Res**erve the indicated number of **w**ords for a data area.                                                   |

### 2.1.1 A Simple SIC Assembler

The translation of source program to object code requires us to accomplish the following functions (not necessarily in the order given):

1. Convert mnemonic operation codes to their machine language equivalent

   e.g., translate TTL to 14

2. Convert symbolic operands to their equivalent machine addresses

   e.g., translate RETADR to 1033

3. Build the machine instructions in the proper format.

4. Convert the data constants specified in the source program into their internal machine representations

   e.g., translate EOF to 454F46

5. Write the object program and the assembly listing

In addition to translating the instructions of the source program, the assembler must process statements called **assembler directives** (or pseudo-instructions). These statements are not translated into machine instruction (although they may have an effect on the object program) . Instead, they provide instructions to the assembler itself.

Finally, the assembler must write the generated object code onto some output device. This object program will later be loaded into memory for execution. The simple object program format we use contains three types of records: Header, Text, and End.

Header records:

The header record contains the program name, starting address ,and length.

> Col. 1         H
>
> Col. 2-7      Program name
>
> Col. 8-13    Starting address of object program (hex)
>
> Col. 14-19  Length of object program in bytes (hex)

Text records:

The text record contains the translated instruction and data of the program, together with an indication of the addresses where these are to be loaded.

> Col.1          T
>
> Col. 2-7     Starting address for object code in this record (hex)
>
> Col. 8-9     Length of object code in this record in bytes (hex)
>
> Col. 10-89 Object code, represented in hexadecimal (2 columns per byte of object code)

End records:

The end of the object program and specifies the address in the program where execution is to begin. (This is taken from the operand of the program's END statement. If no operand is specified, the address of the first executable instruction is used.)

> Col. 1    E
>
> Col. 2-7 Address of first executable instruction in object program (hex)

### 2.1.2 Assembler Algorithm and Data Structures

Our simple assembler uses two major internal data structures: the **Operation Code Table(OPTAB)** and the **Symbol Table (SYMTAB)**. We also need a  **Location Counter (LOCCTR)**

> **OPTAB** (static)
>
> It is used to look up mnemonic operation codes and translate them to their machine language equivalents.
>
> + Pass 1: used to loop up and validate operation codes in the source
>   program
>
> + Pass 2: used to translate the operation codes to machine language
>
> **SYMTAB** (dynamic)
>
> It is used to store values (address) assigned to labels.
>
> + Pass 1: labels are entered into SYMTAB with their address (from
>   LOCCTR) as they are encountered in the source program
>
> + Pass 2: symbols used as operands are looked up in SYMTAB to
>   obtain the address to be inserted in the assembled instruction
>
> **LOCCTR**
>
> It is a variable that is used to help in the assignment of addresses. LOCCTR is initialized to the beginning address specified in the START statement. After each source statement is processed, the length of the assembled instruction or data area to be generated is added to LOCCTR. Thus whenever we reach a label in the source program, the current value of LOCCTR gives the address to be associated with that label.

## 2.2 Machine-Dependent Assembler Features

In our assembler language

+ <u>Indirect addressing</u> is indicated by adding the <u>prefix @</u> to the operand(see line70).

+ <u>Immediate operands</u> are denoted with the <u>prefix \#</u>

+ <u>Instructions that refer to memory </u>are normally assembled using either the <u>program-counter relative (PC)</u> or the <u>base relative mode (B)</u>.

If the displacements required for both program-counter relative and base relative addressing are too large to fit into a 3-byte instruction, then the 4-byte extended format (Format 4) must be used.

### 2.2.1 Instruction formats and addressing modes

![figure2.6](src/Figure2-6.jpg)

Translation of reg-to-reg instructions presents no new problems. The assembler must simply <u>convert the mnemonic operation code to machine language</u> and <u>change each register mnemonic to its numeric</u> equivalent. This translation is done during Pass2, at the same point at which the other types of instruction are assembled. The conversion of register mnemonics to numbers can be done with a separate table; however, it is often convenient to <u>use the symbol table for this purpose</u>. To do this, SYMTAB would be preloaded with the register names(A, X, etc.) and their values(0, 1, etc.)

Most of the reg-to-mem instructions are assembled using either program-counter relative or base relative addressing. The assembler must, in either case, calculate a displacement to be assembled as part of the object instruction. This is computed so that the correct target address results when the displacement is added to the contents of the program counter(PC) or the base register(B). Of course, the resulting displacement must be small enough to fit in the 12-bit filed in the instruction. This means that the displacement must be between 0 and 4095 (for base relative mode), or between -2048 to 2047 (for program-counter relative mode).

If neither program-counter relative nor base relative addressing can be used (because the displacements are too large), then the 4-bytes extended instruction format (Format 4) must be used. The 4-byte format contains a 20-bit address field, which is large enough to contain the full memory address. In this case, there is no displacement to be calculated.

> 在 SIC/XE 中一共有 2<sup>20</sup> bytes 所以 format 4 可以直接定址，而 format 3 可以用相對的方法去取得位置

For example

```nasm
CLOOP    +JSUB    RDREC
```

Note that the programmer must specify the extended format by using the prefix **+**. If extended format is not specified, our assembler **first** attempts to translate the instruction **using program-counter relative addressing**. **If** this is not possible(because the required displacement is **out of range**), the assembler **then** attempts to **use base relative addressing**. **If neither form of relative addressing is applicable and extended format is not specified**, then the instruction cannot be properly assembled. In this case, the assembler must generate an **error message**.

---

We now examine the details of the displacement calculation for program-counter relative and base relative addressing modes.

```nasm
10    0000    FIRST    STL    RETADR
12    0003    ...
...
95    0030    RETADR   RESW   1
```

> 首先先以 PC relative 來做計算
>
> STL = 14 = 0001 0100
>
> RETADR  = 0030
>
> TA = 0030 - 0003 = 002D
>
> | OP      | n/i/x/b/p/e | disp |
> | ------- | ----------- | ---- |
> | 0001 01 | 11 0010     | 02D  |
>
> Obj code: **17202D**

```nasm
105    0036    BUFFER    RESB    4096
...
160    104E              STCH    BUFFER,X
165    1051    ...
```

> 首先先以 PC relative 來做計算
>
> STCH = 54 = 0101 0100
>
> BUFFER = 0036
>
> PC = 1051
>
> TA = 0036 - 1051 <0 ... (out of range)
>
> 改用 Base relative 來計算 (目前base 為 0033)
>
> TA = 0036 - 0033 = 3
>
> | OP      | n/i/x/b/p/e | disp |
> | ------- | ----------- | ---- |
> | 0101 01 | 1/1/1/1/0/0 | 003  |
>
> Obj code: 57C003

(待補)

### 2.2.2 Program relocation

As we mentioned before, it is often desirable to have more than one program at a time sharing the memory and other resources of the machine. If we knew in advance exactly which programs were to be executed concurrently in this way, <u>we could assign address when the programs were assembled </u>so that they would fit together without overlap or wasted space. Most of time, however, it is not practical to plan program execution this closely. (W e usually do not know exactly when jobs will be submitted, exactly how long they will run, etc.) Because of this, <u>it is desirable to be able to load a program into memory wherever there is room for it</u>.

Since the assembler does not know the actual location where the program will be loaded, it cannot make the necessary changes in the addresses used by the program. However, the <u>assembler can identify for the loader those parts of the object program that need modification.</u> An object program that contains the information necessary to perform this kind of modification is called a *relocatable program*.

We can solve the relocation problem in the following way:

> 1. Load programs into memory wherever there is room.
>
> 2. Not specifying a fixed address at assembly time

+ No instruction modification is needed for <u>immediate addressing</u>, <u>PC-relative</u>, <u>Base-relative addressing</u>

+ The only parts that require modification at load time are those that specify <u>direct addresses</u>

+ In SIC/XE Format 1, 2, 3 are not effect

+ 反正就是只改 format4 的，但是如果他有用 # 做立即定址，就不更動他

We can accomplish this with a Modification record having the following format:

> Col. 1     M
>
> Col. 2-7  Starting location of the address field to be modified, relative to the beginning of the program (hex)
>
> Col. 8-9  Length of the address field to be modified, **in half-bytes** (hex)

> 這裡的話舉個例子，比如
>
> ```nasm
> 0006    CLOOP    +JSUB    RDREC
> ```
>
> 在OPCODE記錄為(沒有空隔)
>
> ```nasm
> M 000007 05
> ```
>
> 這裡可以理解在 7 後面更改長度為 2.5 bytes (20 bits) 的地址
>
> ```nasm
> 4B101036
> ```
>
> 我們把這個翻譯回去
>
> ```nasm
> 0010 1011 0001 0x01036
> ↑        ↑     ↑      ↑
> 6        7     7.5    10
> ```
>
> | OP      | n/i/x/b/p/e | addr   |
> | ------- | ----------- | ------ |
> | 0010 10 | 1/1/0/0/0/1 | 0x0136 |

## 2.3 Machine-Independent Assembler Features

### 2.3.1 Literals (Skip)

### 2.3.2 Symbol-defining statements (Skip ORG)

Up to this point the only user-defined symbols we have seen in assembler language programs have appeared as labels on instructions or data areas. The value of such a label is the address assigned to the statement on which it appears. Most assemblers provide an assembler directive that allows the programmer to define symbols and specify their values.

```nasm
symbol    EQU    value
```

same as following code in C

```c
#define symbol value
```

How assembler handles it?

> **Pass1:** when the assembler encounters the EQU statement, Enters the symbol into SYMTAB.
>
> **Pass2**: Assemble the instruction with the value of the symbol

### 2.3.3 Expressions (Skip)

### 2.3.4 Program Blocks

In all of the examples we have seen so far the program being assembled was treated as a unit. The source programs logically contained subroutines, data areas, etc. However, they were handled by the assembler as one entity, resulting in a single block of object code. Within this object program the generated machine instruction and data appeared in the same order as they were written in the source program.

Two approaches to provide more flexibility:

1. Program blocks

   Segments of code that are rearranged within a single object program unit

2. Control sections

   Segments of code that are translated into independent object program units

#### Program block

![Figure2.11](src/Figure2-11.jpg)

**Block name table**

| Block name | Block number | Address | Length |
|:----------:|:------------:|:-------:|:------:|
| (default)  | 0            | 0000    | 0066   |
| CDATA      | 1            | 0066    | 000B   |
| CBLKS      | 2            | 0071    | 1000   |

> First block
>
> Line 5 \~ Line 70 + Line 123 ~ Line 180 + Line 208 ~ Line 245

> Second Block(CDATA)
>
> Line 92 \~ Line 100 + Line 183 \~ Line 185 + Line 252 \~ Line 255

> Third Block (CBLKS)
>
> Line 105 \~ Line 107

**How the assembler handles program blocks**

**<mark>Pass1.</mark>**

+ Maintaining separate location counter for each program block.

+ Each label is assigned an address that is relative to the start of the block that contains it

+ When labels are entered into SYMTAB

  + The block name or number is stored along with the relative addresses

+ At the end of Pass1, the latest value of the location counter for each block indicates the length of that block

**SYMTAB**

| Symbol Name | Address | Block Number |
|:-----------:|:-------:|:------------:|
| FIRST       | 0000    | 0            |
| CLOOP       | 0003    | 0            |
| ENDFIL      | 0015    | 0            |
| RETADR      | 0000    | 1            |
| LENGTH      | 0003    | 1            |
| ......      | ......  | .....        |

**<mark>Pass2.</mark>**

+ The address of each symbol can be computed by

+ Belonging block's starting address + relative address of the symbol

+ Then calculate the object code as before

#### Control sections

> A part of the program that maintains its identity after reassembly
>
> Each control section can be assembled, loaded and relocated independently
>
> Often used for subroutines or other logical subdivisions of program

Instruction in one control section may refer to instructions or data located in another section. Called **external reference**

However, assembler have no idea where any other control sections will be located at execution time.

The assembler has to generate information for **external references** to allow the loader to perform the required linking

**External definition and reference**

> **External definition**
>
> <u>Assembler directives: EXTDEF</u>
>
> > 在 control section 內的 symbol 若要在另一個 control section 取用時，可宣告為 EXTDEF。功能類似 public
>
> EXTDEF names symbols, called **external symbols**, that are defined in this control section and may be used by other sections
>
> Control section names do not need to be named in an EXTDEF statement (e.g. COPY, RDREC, and WRREC)
>
> + They are automatically considered to be external symbols.
>
> **External reference**
>
> <u>Assembler directives: EXTREF</u>
>
> > 需要使用其他 control section 內的 symbol 時，可宣告 EXTREF。功能類似 PHP 中 function 要取用外部變數時使用 globol
>
> EXTREF names symbols that are used in this control section and are defined elsewhere

##### Define record

gives information about external symbols names by EXTDEF

> Col.1         D
>
> Col. 2-7     Name of external symbol defined in this section
>
> Col. 8-13   Relative address within this control section (hex)
>
> Col. 14-73 Repeat information in Col. 2-17

##### Refer record

lists symbols used as external reference, i.e., symbols named by EXTREF

> Col. 1        R
>
> Col. 2-7     Name of external symbol referred to in this section
>
> Col. 8-73  Name of other external reference

##### Modification record

> Col. 1         M
>
> Col. 2-7      Starting address of the field to be modified (hex)
>
> Col. 8-9      Length of the field to be modified, in half-bytes (hex)
>
> Col. 10        Modification flag (+ or -)
>
> Col. 11-16  External symbol whose value is to be added to or subtracted from the indicated held.

Example (figure 2.17)

#### Program blocks v.s. control sections

> **Program blocks**
>
> Refer to segments of code that are rearranged with a single object program unit
>
> **Control sections**
>
> Refer to segments that are translated into <u>independent object program units</u>

## 2.4 Assembler Design Options

### 2.4.1 One-pass assemblers

+ Goal: avoid a second pass over the the source program

+ Main: Forward references to <u>data items</u> or <u>labels on instructions</u>

**Solution**

1. Data items: require all such areas be defined before they are referenced

2. Label on instructions: can not be eliminated (one-pass 重點處理)

**Two types of one-pass assemblers**

> 1. Load-and-go
>
>    Produces object code directly in memory for immediate execution
>
> 2. The other assembler
>
>    Produces usual kind of object code for later execution.
>
>    However, when definition of a symbol is encountered, the assembler must generate another
>    Text record with the correct operand address.
>
>    When the program is loaded, this address will be
>    inserted into the instruction by loader.

#### Load-and-go assembler

#### The other assembler

### 2.4.2 Multi-pass assembler
