# ARMssembly 2

Category: Reverse Engineering

Author: Dylan McGuire

# Solution

`cat chall_2.S` gives us the following ARM assembly

```
        .arch armv8-a
        .file   "chall_2.c"
        .text
        .align  2
        .global func1
        .type   func1, %function
func1:
        sub     sp, sp, #32
        str     w0, [sp, 12]
        str     wzr, [sp, 24]
        str     wzr, [sp, 28]
        b       .L2
.L3:
        ldr     w0, [sp, 24]
        add     w0, w0, 3
        str     w0, [sp, 24]
        ldr     w0, [sp, 28]
        add     w0, w0, 1
        str     w0, [sp, 28]
.L2:
        ldr     w1, [sp, 28]
        ldr     w0, [sp, 12]
        cmp     w1, w0
        bcc     .L3
        ldr     w0, [sp, 24]
        add     sp, sp, 32
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

`main` functions gets an argument and passes it to `func1`. `func1` consist of a loop, whose condition is  `w1 < w0`. That comes from the part
```
cmp w1, w0
bcc .L3 # See Notes for bcc command
```
`w0` holds the input argument and `w1` gets incremented by 1 on each loop iteration. The final result is stored on `sp 24` and gets incremented by 3 on each loop iteration. Therefore, `func1` returns multiplies by 3 its argument. However, the desired flag must be of 32 bits length and the result *78304947301* is 33 bits long. Therefore, we keep the last 32 bits and we get the hex 0xd2bbde0a and the flag is *picoCTF{d2bbde0a}*.

# Notes

`bcc` commands checks if the carry flag is clear. If so, then the branch is taken, otherwise the branch is not taken. In our example, the carry flag is taken until `w1 >= w0`. The `cmp w1, w0` command essentially performs the subtraction `w1-w0` and if `w0>w1` the carry flag is set.
