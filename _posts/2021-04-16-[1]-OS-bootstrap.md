---
layout: single
categories: 
    - OS
author: Yue Yin

toc: true
toc_sticky: true
---

# PC Booting



## TLDR

When booting a PC, the following happens:

- BIOS ROM executes to load *bootloader* from *boot sector* on disk into memory;
- The bootloader switches from 16-bit real mode to 32-bit protected mode, and loads kernel code (in ELF format) from disk into memory;
- The kernel sets up hardware-supported virtual memory, switches to use virtual memory by setting `CRO_PG` flag in %cr0 register, prepares kernel stack, and jumps to C code at high address. 



## PC Physical Memory Layout

```
+------------------+  <- 0xFFFFFFFF (4GB)
|      32-bit      |
|  memory mapped   |
|     devices      |
/\/\/\/\/\/\/\/\/\/\

/\/\/\/\/\/\/\/\/\/\
|      Unused      |
+------------------+  <- depends on amount of RAM
|                  |
| Extended Memory  |
|                  |
+------------------+  <- 0x00100000 (1MB)
|     BIOS ROM     |
+------------------+  <- 0x000F0000 (960KB)
|  16-bit devices, |
|  expansion ROMs  |
+------------------+  <- 0x000C0000 (768KB)
|   VGA Display    |
+------------------+  <- 0x000A0000 (640KB)
|    Low Memory    |
+------------------+  <- 0x00000000
```

Intel 8088 can only address 1MB of physical memory, from 0x00000000 to 0x0000FFFF. The 640KB area marked as "Low Memory" is the only RAM that can be used. The 384KB area from 0x000A0000 through 0x000FFFFF was reserved by the hardware. BIOS is stored in read-only memory (ROM).

When Intel advanced from 8088 to 8086, the architecture is preserved for backward compatibility. So there's a "hole" in modern 8086 PC's physical memory, from 0x000A0000 to 0x00100000. Memory above 0x00100000 is called "extended memory". Also, some memory at the top of the 32-bit address space is also reserved. If the processor can support more than 4GB of physical memory, this becomes the second "hole" in the physical memory address space. JOS only uses the first 256 MB of a PC's physical memory.



## BIOS ROM

BIOS loads bootloader from disk into memory. If the disk is bootable, the first sector is called the *boot sector*. BIOS loads the boot sector into memory at physical address 0x7c00 through 0x7dff, and uses `jmp` instruction to set the `CS:IP` to `0000:7c00`, passing control to the boot loader. Theses addresses are designed arbitrarily by Intel.



## Bootloader

The bootloader loads the kernel from disk. The kernel is stored starting from the 2nd disk sector, in ELF format.

When compiling and linking a C program, the compiler transforms source file `.c` into *object file* `.o`. The object file contains assembly instructions in binary format (0's and 1's). The linker combines all object files into a single *binary image* like `obj/kern/kernel` in the ELF format. ELF: Executable and Linkable Format.

Consider an ELF executable as a header with loading information, followed by several *program sections*, each being a contiguous chunk of code or data to be loaded at a specified memory address. The boot loader loads the ELF executable into memory and starts executing.

An ELF binary starts with a fixed-length *ELF header*, followed by a variable-length *program header*, listing program sections to be loaded. We are interested in the following program sections:

- `.text`: the program's executable instructions.
- .`rodata`: read-only data, like ASCII string constants produced by the C compiler.
- `.data`: the program's initialized data, like *initialized* global variables.
- `.bss`: space for *uninitialized* global variables, the space is zeroed.

When linker computes the memory layout of the program, it reserves space for *unitialized* global variables in the `.bss` section, immediately after `.data`. C requires that unitialized global variables start with value of zero. There's no need to store contents for `.bss` in the ELF binary. Instead, the linker records the address and size of the `.bss` section. The loader arranges to zero the `.bss` section when loading the executable into the memory.



## Kernel

LMA, load memory address, is the memory address at which that section should be loaded into memory. VMA, virtual memory address, is the memory address from which the section expects to execute. For example, JOS kernel has VMA = 0xf0100000, LMA = 0x00100000. 

The kernel is loaded at high virtual address to leave the lower part of the address space to user programs. At bootstrap, PC relies on CPU's memory management hardware to map high virtual address to low physical address. For JOS, we'll map the first 4MB of physical memory, from physical address `0x00000000` through `0x00400000`, to virtual addresses `0xf0000000` through `0xf0400000`. This is done using hand-written, statically-initialized page directory and page table in `kern/entrypgdir.c`.

Up until the `CR0_PG` is set, memory references are treated as physical addresses. Once `CR0_PG` is set, memory references are virtual addresses translated by the virtual memory hardware to physical addresses. `entry_pgdir` translates virtual addresses in the range `0xf0000000` through `0xf0400000` to physical addresses `0x00000000` through `0x00400000`, as well as virtual addresses `0x00000000` through `0x00400000` to physical addresses `0x00000000` through `0x00400000`. That is, both virtual address `[0x00000000, 0x00400000]` and virtual address `[0xf0000000, 0xf0400000]` are mapped to physical address `[0x00000000, 0x00400000]`. Any virtual address that's not in one of these two ranges would cause hardware exception.



## Kernel stack

The kernel stack is manually allocated using `.data`:

```
.data
###################################################################
# boot stack
###################################################################
	.p2align	PGSHIFT		# force page alignment
	.globl		bootstack
bootstack:
	.space		KSTKSIZE
	.globl		bootstacktop   
bootstacktop:
```



## Reference

https://github.com/yinfredyue/MIT6.828/blob/master/labNotes/lab1/lab1.md

https://pdos.csail.mit.edu/6.828/2018/labs/lab1/