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

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/a332855ba537f0872d583cadbea23018801c4264/Images/image8.jpg)

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

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/5baa72d367e8fc7867f5467adcdee7b1ad266105/Images/image4.jpg)

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

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/9d9a8c075dc31eea662c7c5551f0f8b703bbc285/Images/image1.jpg)

## 4. Hex Dump & Disassembly

```bash
riscv32-unknown-elf-objdump -d hello.elf > hello.dump
riscv32-unknown-elf-objcopy -O ihex hello.elf hello.hex
```

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/c6af0f14ead338576c6221b1c82d050fcb592529/Images/image15.jpg)
![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/99fcf5c388519d1cb732421ba4c60732d24f744f/Images/image6.jpg)

## 5. RISC-V Register ABI

| Register | ABI Name | Role          |
|----------|----------|---------------|
| x0       | zero     | Always zero   |
| x1       | ra       | Return address|
| x2       | sp       | Stack pointer |
| x10–x17  | a0–a7    | Args/returns  |
| x5–x7    | t0–t2    | Temporaries   |


## 6. GDB Debugging

```bash
riscv32-unknown-elf-gdb hello.elf
(gdb) target sim
(gdb) break main
(gdb) run
(gdb) stepi
(gdb) info registers
```

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/199a879e3eb6d3e3d8991512933d697215d5fc36/Images/image7.jpg)
![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/33b3805b937af46ea16c35285f197fe8a41263d6/Images/image11.jpg)

## 7. Emulator Usage

```bash
spike --isa=rv32imc pk hello.elf
# or
qemu-system-riscv32 -nographic -kernel hello.elf
```

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/a8a0ac0c7ac799dff3afacfabe9ca60891858c24/Images/image12.jpg)
![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/2d00752c97879654a73629e5f12ec8250785c7f6/Images/image2.jpg)

## 8. GCC Optimization

```bash
riscv32-unknown-elf-gcc -O0 -S hello.c -o O0.s
riscv32-unknown-elf-gcc -O2 -S hello.c -o O2.s
```

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/6f7843e2d06636da896d75b2eb5833aaff92f7d1/Images/image5.jpg)
![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/c178ba8cd333465651e3708a048e46d53e749a5a/Images/image16.jpg)

## 9. Inline Assembly

```c
static inline uint32_t rdcycle(void) {
    uint32_t c;
    asm volatile ("csrr %0, cycle" : "=r"(c));
    return c;
}
```

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/e1fc039ad230506eb8b2b4408494e9f25385edd5/Images/image13.jpg)

## 10. Memory-Mapped I/O

```c
volatile uint32_t* gpio = (uint32_t*)0x10012000;
*gpio = 0x1;
```

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/b731cb5a0d922a2c21d889c9833afdd0ed524423/Images/image17.jpg)

## 11. Linker Script Basics

```ld
SECTIONS {
  .text 0x00000000 : { *(.text*) }
  .data 0x10000000 : { *(.data*) }
}
```

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/83d0f5a67e52b4d80810cda6bc8b41c5351d13af/Images/image10.jpg)

## 12. Startup Code (crt0.S)

- Set stack pointer
- Zero `.bss`
- Call `main()`
- Infinite loop

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/77db721569cc06aecc9680e5fdf3caf28a6aa68e/Images/image19.png)
![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/c7bd3bcfd2b314f42e1e97cb09828929cc598bd8/Images/image20.png)

## 13. Timer Interrupt

- Write to `mtimecmp`
- Enable `mie`, `mstatus`
- Use `__attribute__((interrupt))`

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/7a3649d6fe9b6df1ac3ee7f79a43f59badeecccf/Images/image9.jpg)

## 14. Atomic 'A' Extension

Instructions: `lr.w`, `sc.w`, `amoadd.w`, etc.

Used for: Spinlocks, mutexes, semaphores


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

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/1e1b3e04da2f0bd7e696a59b12e4be261540a8a9/Images/image3.png)
![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/a5bf615f41d7e5aafb39da486c7dba335adb88a0/Images/image14.png)

## 16. Newlib printf Without OS

```c
int _write(int fd, const char* buf, int len) {
    for (int i = 0; i < len; i++) {
        *((volatile char*)0x10000000) = buf[i];
    }
    return len;
}
```


## 17. Endianness & Struct Packing

```c
union {
    uint32_t i;
    uint8_t b[4];
} u = { .i = 0x01020304 };

printf("%x %x %x %x\n", u.b[0], u.b[1], u.b[2], u.b[3]);
```
Expected output: `04 03 02 01` (Little Endian)

![Image alt](https://github.com/Aryan2632/vsdRiscvSoc/blob/14140da8f990015ca6ea72c84e57b4f4b0e551ff/Images/image18.jpg)

