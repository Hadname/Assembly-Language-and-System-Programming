# CH5 Procedures
## Stack Operations

| Memory |
|:------:|
| Code   |
| Data   |
| Heap   |
| ↓      |
| ↑      |
| Stack  |

| Address  | Data     |
| -------- | -------- |
| 00001000 | 00000006 |
| 00000FFC | 000000A5 |
| 00000FF8 | 00000002 |
| 00000FF4 |          |

### Nested Loop

```nasm
    mov ECX, 100      ;set outer loop count
L1:
    push ECX          ; save outer loop count

    mov ECX, 20       ; set inner loop count
L2:
    ; TODO
    loop L2

    pop ECX           ; resrote outer loop count
    loop L1
```

### Using push and pop

save and restore registers when they contain important values, PUSH and POP instructions occur in the opposite order.

```nasm
push ESI
push ECX
push EBX

MOV ESI, OFFSET dowrdVal
MOV ECX, LENGTHOF dowrdVal
MOV EBX, TYPE dwordVal
call DumpMem

pop EBX
pop ECX
pop ESI
```

### Example: Reverse string

```nasm
.data
    myString BYTE "ylbmessA"

.code
main PROC
    MOV   ECX, LENGTHOF myString
    MOV   ESI, 0

L1:
    MOVZX EAX, myString[ESI]
    push  EAX
    INC   ESI
    LOOP  L1

    MOV   ECX, LENGTHOF myString
L2:
    pop   EAX
    CALL  WriteChar   ;寫入一個單獨字元到標準輸出
    LOOP  L2

    EXIT
main ENDP
END main
```

### Related instructions

+ PUSHFD and POPFD

  + push and pop the flags register

+ PUSHAD pushed the 32-bit general-purpose registers on the stack

  + order: EAX, ECX, EDX, EBX, ESP, EBP, ESI, EDI

+ POPAD pops the same registers off the stack in reverse order

  + PUSHA and POP A do the same for 16-bit registers

## Defining and Using Procedures

```nasm
sample PROC
    ; TODO
    ; TODO
    ; TODO
    ret ;回到呼叫函數原先的位址
sample ENDP
```

### Example: sumof

```nasm
SumOf PROC
;
; Calculates and returns the sum of three 32-bit integers
; Receives: EAX, EBX, ECX, the three integers. May be signed or unsigned.
; Returns: EAX = sum, and the status flags(Carry,
; Overflow, etc.) are changed
; Requires: nothing

    add eax, ebx
    add eax, ecx
    ret
SumOf ENDP
```

### Call and Ret

```nasm
main PROC
    CALL  Mysub            ;PUSH EIP
    MOV   EAX, EBX
    ; TODO
main ENDP

Mysub PROC
    ; TODO
    RET                    ;POP EIP
Mysub ENDP
```

### Nested procedure calls

### Procedure Parameters

**寫法一 (差)**

```nasm
ArraySum PROC
    MOV   ESI, 0
    MOV   EAX, 0
    MOV   ECX, LENGTHOF myarray

L1
    ADD   EAX, myArray[ESI]
    ADD   ESI, 4
    LOOP L1

    MOV   theSum, EAX
    RET
ArraySum ENDP
```

**寫法二 (佳)**

```nasm
ArraySum PROC
; Receives: ESI points to an array of doubleword
; ECX: number of array elements
; Returns: EAX = sum
    MOV   EAX, 0
L1:
    ADD   EAX, [ESI]
    ADD   ESI, 4
    LOOP L1
    RET
ArraySum ENDP
```

> 寫法一把陣列名稱寫死了，寫法二則利用 ESI

### USES

> Lists the registers that will be preserved

```nasm
ArraySum PROC USES ESI ECX
    MOV EAX, 0
```

> MASM 會轉換成：

```nasm
ArraySum PROC
    PUSH  ESI
    PUSH  ECX
    ; ...
    ; ...
    ; ...
    POP   ECX
    POP   ESI
    RET
ArraySum ENDP
```
