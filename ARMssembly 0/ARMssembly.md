# ARMssembly 0

Category: Reverse Engineering

Author: Dylan McGuire

# Solution

`cat chall.S` gives us the following ARM assembly

```
        .arch armv8-a
        .file   "chall.c"
        .text
        .align  2
        .global func1
        .type   func1, %function
func1:
        sub     sp, sp, #16
        str     w0, [sp, 12]
        str     w1, [sp, 8]
        ldr     w1, [sp, 12]
        ldr     w0, [sp, 8]
        cmp     w1, w0
        bls     .L2
        ldr     w0, [sp, 12]
        b       .L3
.L2:
        ldr     w0, [sp, 8]
.L3:
        add     sp, sp, 16
        ret
        .size   func1, .-func1
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
        str     x19, [sp, 16]
        str     w0, [x29, 44]
        str     x1, [x29, 32]
        ldr     x0, [x29, 32]
        add     x0, x0, 8
        ldr     x0, [x0]
        bl      atoi
        mov     w19, w0
        ldr     x0, [x29, 32]
        add     x0, x0, 16
        ldr     x0, [x0]
        bl      atoi
        mov     w1, w0
        mov     w0, w19
        bl      func1
        mov     w1, w0
        adrp    x0, .LC0
        add     x0, x0, :lo12:.LC0
        bl      printf
        mov     w0, 0
        ldr     x19, [sp, 16]
        ldp     x29, x30, [sp], 48
        ret
        .size   main, .-main
        .ident  "GCC: (Ubuntu/Linaro 7.5.0-3ubuntu1~18.04) 7.5.0"
        .section        .note.GNU-stack,"",@progbits
```

Although `func1` might seem a bit confusing at first sight, it actually calculates the maximum of two numbers given as parameters (`w0` and `w1`).

`main` function expects 2 numbers as parameters (reading is done with the help of `atoi`), calls `func1` with those parameters and print the result with `printf`. So, essentially the result will be the maximum of the two numbers given. In particular we have 4112417903 and 1169092511 and therefore the programm is going to print:

```
Result: 4112417903
```

and the hex number if f51e846f. Finally, the flag is *picoCTF{f51e846f}*.
# Notes

*W* registers in ARM assembly are for 32 bits and *X* registers are for 64 bits. Actually, *W1* register is the same as the last 32 bits of *X1* register.
