# CH6 Conditional Processing

| Upper |                  | Small |           |
|:-----:|:----------------:|:-----:|:---------:|
| 'A'   | 01<u>0</u>0 0001 | 'a'   | 0110 0001 |
| 'B'   | 01<u>0</u>0 0010 | 'b'   | 0110 0010 |
| 'C'   | 01<u>0</u>0 0011 | 'c'   | 0110 0011 |
| 'D'   | 01<u>0</u>0 0100 | 'd'   | 0110 0100 |
| 'E'   | 01<u>0</u>0 0101 | 'e'   | 0110 0101 |

| DEC |                  | ASCII |           |
|:---:|:----------------:|:-----:|:---------:|
| 1   | 00<u>00</u> 0001 | '1'   | 0011 0001 |
| 2   | 00<u>00</u> 0010 | '2'   | 0011 0010 |
| 3   | 00<u>00</u> 0011 | '3'   | 0011 0011 |
| 4   | 00<u>00</u> 0100 | '4'   | 0011 0100 |
| 5   | 00<u>00</u> 0101 | '5'   | 0011 0101 |

## AND

> 用途：要保留的設成 1，要 clear 的設成 0
> 
> 舉個例子要把小寫換成大寫
> 
> > 小寫 & 1101 1111
> 
> 要把字元轉成數字
> 
> > 字元 & 1100 1111

## OR

> 用途：要保留的設成 0，要 preset 的設定 1
> 
> 舉個例子要把大寫換成小寫
> 
> > 大寫 | 0010 0000
> 
> 要把數字轉成字元
> 
> > 數字 | 0011 0000

## XOR

> 用途：要保留的設成 0，要反轉設成 1

**Example**

```nasm
.data
    myArray BYTE 1,2,5,7,10,11,15

.code
    MOV   ECX, LENGTHOF myArray ;Set the number for LOOP executions
    XOR   EBX, EBX              ;Set EBX = 0
    XOR   ESI, ESI              ;Set ESI = 0

L1:
    MOVZX EAX, myArray[esi]
    AND   EAX, 1                ;If EAX is 0, myArray[esi] is an even number
                                ;If EBX is 1, myArray[esi] is an odd number
    ADD   EBX, EAX              ;EBX 會存入奇數的個數

L2:
    INC   ESI
    LOOP  L1 
```

## TEST instruction

+ Performs a **nondestructive AND** operation between each pair of matching bits in two operands

+ **No operands are modified, but the Zero flag is affected**

+ Usually used with JZ (jump if ZF=1) or JNZ (jump if ZF=0)

+ 我們不會改變暫存器的值但會改變 ZF 的值

```nasm
TEST  AL, 00000011b 
JNZ   ValueFound

TEST  AL, 00000011b
JZ    ValueNotFound
```

## CMP instruction

+ Compares the destination operand to the source operand
  
  + Nondestructive subtraction of source from destination (destination operand is not changed)

+ Syntax: CMP destination, source    (d-s)

> CMP 就是一個減法的動作

```nasm
MOV    AL, 6
CMP    AL, 5     ; ZF = 0, CF =0
```

**Unsigned Integer**

| Case  | ZF  | CF  |                           |
|:-----:|:---:|:---:|:------------------------- |
| D = S | 1   | -   | D-S=0                     |
| D < S | 0   | 1   | D-S<0 (Unsigned overflow) |
| D > S | 0   | 0   | D-S>0                     |

**Signed Integer**

> 當 SF == OF 時代表 Destination > Source
> 
> 當 SF != OF 時則代表 Destination < Source

| Case | D         | S         | SF  | OF (+7 \~ -8) |
|:----:|:---------:|:---------:|:---:|:-------------:|
| D>S  | 0101 (5)  | 1110 (-2) | 0   | 0             |
| D>S  | 0110 (6)  | 1110 (-2) | 1   | 1             |
| D<S  | 1110 (-2) | 0111 (7)  | 0   | 1             |
| D<S  | 1111 (-1) | 0101 (5)  | 1   | 0             |

## Jump based on specific flags

| Mnemonic | Description           | Flags |
|:--------:|:--------------------- |:-----:|
| JZ       | Jump if zero          | ZF=1  |
| JNZ      | Jump if not zero      | ZF=0  |
| JC       | Jump if carry         | CF=1  |
| JNC      | Jump if not carry     | CF=0  |
| JO       | Jump if overflow      | OF=1  |
| JNO      | Jump if not overflow  | OF=0  |
| JS       | Jump if signed        | SF=1  |
| JNS      | Jump if not signed    | SF=0  |
| JP       | Jump if parity (even) | PF=1  |
| JNP      | Jump if parity (odd)  | PF=0  |

## Jump based on equality

| Mnemonic | Description         |
|:--------:| ------------------- |
| JE       | Jump if  equal      |
| JNE      | Jump if not equal   |
| JCXZ     | Jump if CX is zero  |
| JECXZ    | Jump if ECX is zero |

## Jump based on unsigned compares (Above & Below)

| Mnemonic | Description                             |
|:-------- | --------------------------------------- |
| **JA**   | Jump if above (left > right)            |
| JNBE     | Jump if not below or equal (same as JA) |
| **JAE**  | Jump if above of equal (left >= right)  |
| JNB      | Jump if not below (same as JAE)         |
| **JB**   | Jump if below                           |
| JNAE     | Jump if not above or equal (same as JB) |
| **JBE**  | Jump if below or equal                  |
| JNA      | Jump if not above (same as JBE)         |

## Jump based on signed compares (Greater & Less)

| Mnemonic | Description                                 |
|:-------- | ------------------------------------------- |
| **JG**   | Jump if greater                             |
| JNLE     | Jump if not less than or equal (same as JG) |
| **JGE**  | Jump if greater or equal                    |
| JNL      | Jump if not less than (same as JGE)         |
| **JL**   | Jump if less than                           |
| JNGE     | Jump if not greater or equal (same as JL)   |
| **JLE**  | jump if less than or equal                  |
| JNG      | Jump if not greater (same as JLE)           |

### Application

Compare unsigned AX to BX, and copy the larger of the two into a variable named Large

```nasm
     MOV LARGE, BX   //假設 BX 比較大
     CMP AX, BX
     JNA NEXT       //如果 AX 沒有 BX 來的大
     MOV LARGE, AX
NEXT:  
```
