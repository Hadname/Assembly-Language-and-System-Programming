# Assembly Language and System Programming Midterm

## 2020 111級中興資工 組合語言與系統程式期中考

1. Please list four examples of system software. (10%)

   > + Text editor
   >
   > + Complier
   >
   > + Assembler
   >
   > + Marcro processor

2. (a) What is a compiler? (5%)

   > 將高階程式語言轉換成組合語言

   (b) What is an assembler? (5%)

   > 將低階程式語言轉換成機器語言

3. Please list the following units from fast to slow. (RAM, Register, L2 Catch, L1 Catch, Hard Disk) (6%)

   > Register → L1 Catch → L2 Catch → RAM → Hard Disk

4. What is the purpose of EIP? (4%)

   > 儲存下一個要執行的指令的位置

5. Consider the double word DWORD 34125678h. If placed in memory at offset 0000, please show the memory layout. (Hint: The computer system uses little endian order.) (10%)

   > | 0000 | 78  |
   > |:----:|:---:|
   > | 0001 | 56  |
   > | 0002 | 12  |
   > | 0003 | 34  |

6. In the following instruction sequence, show the values of the Carry, Zero, and Sign flags where indicated: (15%)

```nasm
mov ax,00FFh
add ax,1     ; a) AX= SF= ZF= CF=
sub ax,1     ; b) AX= SF= ZF= CF=
add al,1     ; c) AL= SF= ZF= CF=
mov bh,6Bh
add bh,45h   ; d) BH= SF= ZF= CF=
mov al,5
sub al,6     ; e) AL= SF= ZF= CF=
```

|     | AX   | SF  | ZF  | CF  |
|:---:|:----:|:---:|:---:|:---:|
| (a) | 0100 | 0   | 0   | 0   |
| (b) | 00FF | 0   | 0   | 0   |

|     | AL  | SF  | ZF  | CF  |
|:---:|:---:|:---:|:---:|:---:|
| (c) | 00  | 0   | 1   | 1   |

> 6B = 0110 1011
>
> 45 = 0100 0101
>
> 6B+45 = 1011 0000 = B0

|     | BH  | SF  | ZF  | CF  |
|:---:|:---:|:---:|:---:|:---:|
| (d) | B0  | 1   | 0   | 0   |

> 5 = 0101b
>
> 6 = 0110b
>
> -6 = 1010b
>
> 5 - 6 = 1111b = -1 (超過無號數表示的範圍 CF = 1)

|     | AL                  | SF  | ZF  | CF  |
|:---:|:-------------------:|:---:|:---:|:---:|
| (e) | 1111(bin) = FF(hex) | 1   | 0   | 1   |

7. What will be the values of the given flags after each operation? (4%)

```nasm
mov al,-128
neg al            ;a) CF = OF =
mov ax,8000h
add ax,2          ;b) CF = OF =
mov ax,4
sub ax,8          ;c) CF = OF =
mov al,-10
sub al,+121       ;d) CF = OF =
```

|     | CF  | OF  |
|:---:|:---:|:---:|
| (a) | 1   | 1   |
| (b) | 0   | 0   |
| (c) | 1   | 0   |
| (d) | 0   | 1   |

8. What will be the value in EAX after following lines execute? (6%)

```nasm
mov eax,1002FEFAh
neg ax ; 1) EAX=
```

1002FEFA = 0001 0000 0000 0010 | 1111 1110 1111 1010

0001 0000 0000 0010 | 0000 0001 0000 0110

EAX = 10020106

9. Write down the hexadecimal value of each destination operand: (10%)

```nasm
.data
arrayB BYTE 10h,20h,30h,40h
arrayW WORD 2000h,3000h,5000h
arrayD DWORD 5,6,3,8

myByte BYTE 0FFh, 0
.code
mov eax,0
mov al,arrayB+3        ; 1) AX  = 0040h
mov eax,[arrayD+8]     ; 2) EAX = 00000003h
mov ax,[arrayW+4]      ; 3) AX  = 5000h
mov al,myByte          ; 4) AX  = 50FFh
mov ah,[myByte+1]      ; 5) AX  = 00FFh
dec ah                 ; 6) AX  = FFFFh
inc al                 ; 7) AX  = FF00h
dec ax                 ; 8) AX  = FEFFh
mov ax,00FFh           ; AX = 00FFh
inc al                 ; 9) AX  = 0000h
add ax,0FEBFh          ;10) AX  = FEBFh
```

#### 詳解
>
> eax = 00 00 00 00
>
> __(1)__ AL = 40 EAX = 00 00 00 40
>    
>    __AX = 0040h__
>
> __(2)__ arrayD + 8 = 00 00 00 03
>    
>    __EAX = 00000003h__
>    
>    | +0  | 05     |
>    |:---:|:------:|
>    | +1  | 00     |
>    | +2  | 00     |
>    | +3  | 00     |
>    | +4  | 06     |
>    | +5  | 00     |
>    | +6  | 00     |
>    | +7  | 00     |
>    | +8  | **03** |
>    |     | **00** |
>    |     | **00** |
>    |     | **00** |
>
> __(3)__ arrayW + 4 = 5000h
>    
>    __AX = 5000h__
>
> | +0  | 00  |
> | --- | --- |
> | +1  | 20  |
> | +2  | 00  |
> | +3  | 30  |
> | +4  | 00  |
> |     | 50  |
>


10. Indicate the hexadecimal value of AL after each shift. (10%)

```nasm
mov al,0D4h     ; D4 = 1101 0100
shr al,1        ; (1) AL = 0110 1010 = 6Ah
mov al,0A8h     ; AL = 1010 1000
sar al,1        ; (2) AL = 1101 0100 = D4h
shl al,3        ; (3) AL = 1010 0000 = A0h
mov al,8Ch      ; AL = 1000 1100
sar al,3        ; (4) AL = 1111 0001 = F1h
sar al,1        ; (5) AL = 1111 1000 = F8h
```

11. Translate the following expression into assembly language. (15%)

    > Rval = 35 * Wval + Xval - (-Yval + Zval) + 8

```nasm
.code
mov eax, Wval
sal eax, 5     ;Wval*32
add eax, eax   ;Wval*33
add eax, eax   ;Wval*34
add eax, eax   ;Wval*35
add eax, Xval  ;35*Wval + Xval
add eax, Yval  ;35*Wval + Xval + Yval
sub eax, Zval  ;35*Wval + Xval + Yval - Zval
add eax, 8     ;35*Wval + Xval + Yval - Zval + 8
mov Rval, eax
```

> 第10題和第11題是我自己寫的不確定答案
