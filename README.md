# vsdRiscvSoc

# RISC-V Toolchain & Bare-Metal Programming - Week 1

Welcome! This week is all about setting up your RISC-V toolchain, writing and compiling your first bare-metal programs, and understanding the basics of how things work under the hood. Each task below is practical, beginner-friendly, and relevant for VLSI and embedded enthusiasts. 

## 1. Toolchain Setup & Verification

```bash
tar -xzf riscv-toolchain-rv32imac-x86_64-ubuntu.tar.gz
echo 'export PATH="$HOME/riscv/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

riscv32-unknown-elf-gcc --version
riscv32-unknown-elf-objdump --version
riscv32-unknown-elf-gdb --version
```

![Image alt](assets/https://github.com/Aryan2632/vsdRiscvSoc/blob/a332855ba537f0872d583cadbea23018801c4264/Images/image8.jpg)

## 2. Hello, RISC-V

```c
#include <stdio.h>
int main() {
    printf("Hello, RISC-V!\n");
    return 0;
}
```

```bash
riscv32-unknown-elf-gcc -march=rv32imc -mabi=ilp32 -o hello.elf hello.c
file hello.elf
```

![Image](assets/image10.jpg)

## 3. C to Assembly

```bash
riscv32-unknown-elf-gcc -S -O0 hello.c
```

Assembly snippet:
```asm
addi sp, sp, -16
sw ra, 12(sp)
sw s0, 8(sp)
addi s0, sp, 16
```

![Image](assets/image11.jpg)

## 4. Hex Dump & Disassembly

```bash
riscv32-unknown-elf-objdump -d hello.elf > hello.dump
riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
```

![Image](assets/image12.jpg)

## 5. RISC-V Register ABI

| Register | ABI Name | Role          |
|----------|----------|---------------|
| x0       | zero     | Always zero   |
| x1       | ra       | Return address|
| x2       | sp       | Stack pointer |
| x10–x17  | a0–a7    | Args/returns  |
| x5–x7    | t0–t2    | Temporaries   |

![Image](assets/image13.jpg)

## 6. GDB Debugging

```bash
riscv32-unknown-elf-gdb hello.elf
(gdb) target sim
(gdb) break main
(gdb) run
(gdb) stepi
(gdb) info registers
```

![Image](assets/image14.png)

## 7. Emulator Usage

```bash
spike --isa=rv32imc pk hello.elf
# or
qemu-system-riscv32 -nographic -kernel hello.elf
```

![Image](assets/image15.jpg)

## 8. GCC Optimization

```bash
riscv32-unknown-elf-gcc -O0 -S hello.c -o O0.s
riscv32-unknown-elf-gcc -O2 -S hello.c -o O2.s
```

![Image](assets/image16.jpg)

## 9. Inline Assembly

```c
static inline uint32_t rdcycle(void) {
    uint32_t c;
    asm volatile ("csrr %0, cycle" : "=r"(c));
    return c;
}
```

![Image](assets/image17.jpg)

## 10. Memory-Mapped I/O

```c
volatile uint32_t* gpio = (uint32_t*)0x10012000;
*gpio = 0x1;
```

![Image](assets/image18.jpg)

## 11. Linker Script Basics

```ld
SECTIONS {
  .text 0x00000000 : { *(.text*) }
  .data 0x10000000 : { *(.data*) }
}
```

![Image](assets/image19.png)

## 12. Startup Code (crt0.S)

- Set stack pointer
- Zero `.bss`
- Call `main()`
- Infinite loop

![Image](assets/image2.jpg)

## 13. Timer Interrupt

- Write to `mtimecmp`
- Enable `mie`, `mstatus`
- Use `__attribute__((interrupt))`

![Image](assets/image20.png)

## 14. Atomic 'A' Extension

Instructions: `lr.w`, `sc.w`, `amoadd.w`, etc.

Used for: Spinlocks, mutexes, semaphores

![Image](assets/image3.png)

## 15. Atomic Test Program

```c
while (1) {
    asm volatile (
        "lr.w t0, (a0)\n"
        "sc.w t1, a1, (a0)\n"
        "bnez t1, retry\n"
    );
    *lock = 0;  // release
}
```

![Image](assets/image4.jpg)

## 16. Newlib printf Without OS

```c
int _write(int fd, const char* buf, int len) {
    for (int i = 0; i < len; i++) {
        *((volatile char*)0x10000000) = buf[i];
    }
    return len;
}
```

![Image](assets/image5.jpg)

## 17. Endianness & Struct Packing

```c
union {
    uint32_t i;
    uint8_t b[4];
} u = { .i = 0x01020304 };

printf("%x %x %x %x\n", u.b[0], u.b[1], u.b[2], u.b[3]);
```
Expected output: `04 03 02 01` (Little Endian)

![Image](assets/image6.jpg)

