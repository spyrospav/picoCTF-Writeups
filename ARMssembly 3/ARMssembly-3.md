# ARMssembly 3

Category: Reverse Engineering

Author: Dylan McGuire

# Solution

`cat chall_3.S` gives us the following ARM assembly

```
        .arch armv8-a
        .file   "chall_3.c"
        .text
        .align  2
        .global func1
        .type   func1, %function
func1:
        stp     x29, x30, [sp, -48]!
        add     x29, sp, 0
        str     w0, [x29, 28]
        str     wzr, [x29, 44]
        b       .L2
.L4:
        ldr     w0, [x29, 28]
        and     w0, w0, 1
        cmp     w0, 0
        beq     .L3
        ldr     w0, [x29, 44]
        bl      func2
        str     w0, [x29, 44]
.L3:
        ldr     w0, [x29, 28]
        lsr     w0, w0, 1
        str     w0, [x29, 28]
.L2:
        ldr     w0, [x29, 28]
        cmp     w0, 0
        bne     .L4
        ldr     w0, [x29, 44]
        ldp     x29, x30, [sp], 48
        ret
        .size   func1, .-func1
        .align  2
        .global func2
        .type   func2, %function
func2:
        sub     sp, sp, #16
        str     w0, [sp, 12]
        ldr     w0, [sp, 12]
        add     w0, w0, 3
        add     sp, sp, 16
        ret
        .size   func2, .-func2
        .section        .rodata
        .align  3
.LC0:
        .string "Result: %ld\n"
        .text
        .align  2
        .global main
        .type   main, %function
main:
        stp     x29, x30, [sp, -48]!
        add     x29, sp, 0
        str     w0, [x29, 28]
        str     x1, [x29, 16]
        ldr     x0, [x29, 16]
        add     x0, x0, 8
        ldr     x0, [x0]
        bl      atoi
        bl      func1
        str     w0, [x29, 44]
        adrp    x0, .LC0
        add     x0, x0, :lo12:.LC0
        ldr     w1, [x29, 44]
        bl      printf
        nop
        ldp     x29, x30, [sp], 48
        ret
        .size   main, .-main
        .ident  "GCC: (Ubuntu/Linaro 7.5.0-3ubuntu1~18.04) 7.5.0"
        .section        .note.GNU-stack,"",@progbits
```

`main` function gets an argument and passes it to `func1`. `func1` consists of a loop with condition `w0 != 0`. The loop starts with the following commands
```
and w0, w0, 1 # checks the last bit of w0
cmp w0, 0
beq .L3
```

Therefore, if the last bit of `w0` is 1 the branch is not taken and we have a call to `func2`. `func2` simply adds 3 to the result (at memory position `sp 44`). Then, we have a logical shift right of `w0`, which is equivalent with division by 2. What this loop does is to check the last bit of `w0`, if it is 1 then adds 3 to the result, and shifts right `w0` in order to get the second to last bit to the last position for the next iteration. So, the number of times that 3 is added to the result is equal to the number of 1 bits on the input number. The input number is 469937816 which has a binary representation of *0b11100000000101010111010011000*. The total number of 1 bits is 12 and therefore the final result is 3 multiplied by 12, which is equal to 36 or 0x24. The flag is therefore *picoCTF{00000024}*.
