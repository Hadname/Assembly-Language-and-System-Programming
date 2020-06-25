# CH4 Data Transfers, Addressing, and Arithmetic

## Data Transfer Instructions

**Operand Types**

> + Immediate - a constant integer
>   
>   + value is encoded within the instruction
> 
> + Register - the name of a register
>   
>   + register name is converted to a number and encoded within the instruction
> 
> + Memory - reference to a location in memory
>   
>   + memory address is encoded within the instruction, or a register holds the address of a memory location

### MOV instruction

Move from source to destination. Syntax: MOV destination, source

+ No more than one memory operand permitted

+ **CS, EIP, and IP** cannot be the destination

+ No immediate to segment moves

```nasm
.data
    count BYTE 100
    wVal  WORD 2
.code
    MOV   BL, count    ; 8-bit
    MOV   AX, wVal     ; 16-bit
    MOV   count, AL    ; 8-bit

    ; ERROR :
    MOV   AL,  wVal    ; 8-bit , 16-bit
    MOV   AX,  count   ; 16-bit, 8-bit
    MOV   EAX, count   ; 32-bit, 8-bit 
```

```nasm
.data
    bVal   BYTE  100
    bVal2  BYTE  ?
    wVal   WORD  2
    dVal   DWORD 5

.code
    ; ERROR EXAMPLE :
    MOV DS  ,45    ;immediate move to DS not permitted
    MOV ESI, wVal  ;size mismatch 32-bit, 16-bit
    MOV EIP, dVal  ;EIP cannot be the destination
    MOV 25 , bVal  ;immediate value cannot be destination
    MOV bVal2, bVal;memory-to-memory move not permitted 
```

### Zero extension

When you copy a smaller value into a larger destination the **MOVZX instruction fills (extends)** the upper half of the destination with zeros.

---

> **The destination must be a register**

---

```nasm
MOV   BL, 10001111b
MOVZX AX, BL        ; AX: 00000000 10001111

MOVZX reg32, reg/mem8
MOVZX reg32, reg/mem16
MOVZX reg16, reg/mem8
```

## Sign extension

The **MOVSX instruction fills the upper half of the destination** **with a copy of the source operad's sign bit.**

```nasm
MOV   BL, 10001111b
MOVZX AX, BL       ; AX: 11111111 10001111
```

 ---

### 以下為期末考範圍

> Date: 2020/05/11

---

## OFFSET

OFFSET returns the distance in bytes, of a label from the beginning of its enclosing segment

+ Protected mode: 32bits

+ Real mode: 16bits

> The Protected-mode programs we write use only a single segment (flat memory model)

**Example**

Let's assume that the data segment begins at 00404000h

```nasm
.data
    bVal   BYTE  ?
    wVal   WORD  ?
    dVal   DWORD ?
    dVal2  DWORD ?

.code
    MOV    ESI, OFFSET bVal  ;ESI = 00404000
    MOV    ESI, OFFSET wVal  ;ESI = 00404001
    MOV    ESI, OFFSET dVal  ;ESI = 00404003
    MOV    ESI, OFFSET dVal2 ;ESI = 00404007 
```

## PTR Operator

Overrides the default type of label. Provides the flexibility to access part of a variable.

有點類似 C 語言中用 \* 來取值，但是組語中的指標類似 C 語言的 void\* 是沒有型態的，所以在取值是要在轉型

```nasm
.data
    myDouble DWORD 12345678h

.code
    MOV  AX, myDouble             ; error
    MOV  AX, WORD PTR myDouble    ; AX = 5678
    MOV  WORD PTR myDouble, 4321h ; myDouble = 12344231    
```

| offset | byte |
|:------:|:----:|
| 0000   | 78   |
| 0001   | 56   |
| 0002   | 34   |
| 0003   | 12   |

## TYPE Operator

The TYPE operator return the size, in bytes, of a single element of a data declaration.

```nasm
.data
    var1 BYTE  ?
    var2 WORD  ?
    var3 DWORD ?
    var4 QWORD ?

.code
    MOV  EAX, TYPE var1    ; 1
    MOV  EAX, TYPE var2    ; 2
    MOV  EAX, TYPE var3    ; 4
    MOV  EAX, TYPE var4    ; 8
```

## LENGTHOF Operator

The LENGTHOF operator counts the number of elements in a single data declaration

```nasm
.data
    byte1  BTYE  10, 20, 30           ;  3
    array1 WORD  30 DUP(?), 0 , 0     ; 32
    array2 WORD  5 DUP(3 DUP(?))      ; 15
    array3 DWORD 1,2,3,4              ;  4
    digitStr BYTE "12345678", 0       ;  9

.code
    mov ecx, LENGTHOF array1          ; 32
```

## SIZEOF Operator

The SIZEOF operator return a value that is equivalent to <u>multiplying LENGTHOF by TYPE</u>

```nasm
.data
    array1 WORD 30 DUP(?), 0, 0
.code
    MOV    ECX, SIZEOF array1
```

## INDIRECT Operator

An indirect operand hold the address of a variable, usually an array or string. It can be dereferenced (just like a pointer)

```nasm
.data
    val1 BYTE 10h, 20h, 30h

.code
    MOV    ESI, OFFSET vall
    MOV    AL,  [ESI]        ; AL = 10h

    INC    ESI
    MOV    AL,  [ESI]        ; AL = 20h

    INC    ESI
    MOV    AL,  [ESI]        ; AL = 30h
```

## Exaple Array Sum

## Index scaling

You can scale an indirected or indexed operand to the offset of an array element. This is done by multiplying the index by the array's TYPE.

```nasm
.data
    arrayB BYTE  0,1,2,3,4,5
    arrayW WORD  0,1,2,3,4,5
    arrayD DWORD 0,1,2,3,4,5
.code
    MOV    AL, arrayB[1*TYPE arrayB] ; 01
    MOV    AL, arrayW[2*TYPE arrayW] ; 0002
    MOV    AL, arrayD[2*TYPE arrayD] ; 00000003
```

## Pointers

You can declare a pointer variable that contains the offset of another variable.

```nasm
.data
    arrayW WORD 1000h, 2000h, 3000h
    ptrW   DWORD arrayW
.code
    MOV    ESI, ptrW
    MOV    AX,  [ESI]    ; AX = 1000h
```

**Alternate format**

```nasm
ptrW   DWORD  OFFSET arrayW
```

## JMP Instruction

```nasm
top:
    ;
    ;
    ;
    ;
    JMP top
```

類似 C 語言中的 goto

## LOOP Instruction

```nasm
    MOV   AX,  0
    MOV   ECX, 5
L1: ADD   EAX, CX
    LOOP  L1
```

每經過一次 loop ，就把 ECX 減一 並檢查是否為 0，若為零則跳出迴圈

```nasm
add    EAX, CX    ; AX = 5
loop   L1         ; CX = 4

add    EAX, CX    ; AX = 9
loop   L1         ; CX = 3

add    EAX, CX    ; AX = 12
loop   L1         ; CX = 2

add    EAX, CX    ; AX = 14
loop   L1         ; CX = 1


add    EAX, CX    ; AX = 15
loop   L1         ; CX = 0   跳出
```
