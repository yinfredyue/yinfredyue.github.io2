---
layout: single
categories: 
    - OS
author: Yue Yin

toc: true
toc_sticky: true
---



## TLDR

To set up virtual memory, the kernel does the following (the `mem_init` function):

- Allocate memory for page directory `kern_pgdir`;
- Allocates memory for data structures for physical memory (an array `pages` and a free list `page_free_list`);
- Initialize `pages` and `page_free_list` (the `page_init` function);
- Define functions related to physical memory (by reading/writing `pages` and `page_free_list`);
- Define functions related to virtual memory (by reading/writing the page tables);
- Create mappings in `kern_pgdir` for `pages`, kernel stack, and all physical memory;
- Switch from `entry_pgdir` to `kern_pgdir`.



## Part 1: Physical Page Management

A physical page allocator is needed before implementing the rest of virtual memory. Reason: the page table management code requires allocating physical memory for (1) page directory `kern_pgdir`, and (2) the `pages` array of `struct PageInfo`. 



## Part2: Virtual Memory

[Segmentation and paging mechanism](https://github.com/yyin-dev/MIT6.828/blob/master/labNotes/lab2/IntelManualReading%20-%20Chapter%205.md). In `boot/boot.S`, segmentation is effectively disabled by setting all segment base addresses to 0 and limits to 0xFFFFFFFF. JOS uses two-level page table. 

Before entering kernel (Lab1), a simple page table maps both virtual address `[0x00000000, 0x00400000]` and virtual address `[0xf0000000, 0xf0400000]` to physical address `[0x00000000, 0x00400000]`. In Lab2, JOS maps the first 256MB of physical memory (this is all physical memory that JOS will use) starting at address 0xF00000000 (this contains the 4MB simple mapping in Lab1) and some other regions. 

The bootloader switched from 16-bit mode to 32-bit protected mode. After that, there's no way to directly use a linear or physical address. *All* memory references are interpreted as virtual addresses and translated by MMU (so all C pointers are virtual addresses). It's meaningless to dereference a physical address now, as MMU translates all addresses.

The JOS kernel may need to find a physical address given a virtual address. Kernel global variables and memory allocated by `boot_alloc` is in the region where the kernel is loaded, starting at 0xF0000000. This is the same address where we mapped ALL physical memory. Thus, to turn a virtual address in this region into a physical address, the kernel can simply subtract 0xF0000000. Use `PADDR(va)` and `KADDR(pa)`.

One physical page can be mapped at multiple virtual address (or in the address spaces of multiple environments). The `PageInfo` struct maintains a `pp_ref` as the physical page reference count. The value should equal to the number of times the physical page appears *below* `UTOP` in all page tables (the mappings above `UTOP` are mostly set up at boot time by the kernel and should never be freed, so there's no need to reference count them). 



## Part 3: Kernel Address Space

JOS divides CPU's 32-bit address space into two parts. User environments (processes) controls the lower part, while the kernel maintains complete control over the upper half. The dividing line is defined arbitrarily by `ULIM`, reserving approximately 256MB of virtual address space for kernel. Refer to `inc/memlayout.h`.

As in `inc/memlayout.h`, kernel and user memory are both present in each environment's address space, we must set permission bits in page tables properly. Otherwise, bugs in user code can overwrite kernel data. 

For `[ULIM, 0xFFFFFFFF]`, the user environment has no permission, while kernel has full permission. For `[UTOP, ULIM)`, both the kernel and user environment can read but not write. This range is used to expose certain kernel data structures read-only to the user environment. For `[0, UTOP)`, the address space is for the user environment to use; the user environment will set permissions for accessing this memory.

In this part, we set up address space above `UTOP`: the kernel part of the address space. In `mem_init`, use `boot_map_region` to map `pages` at `UPAGES`, kernel stack at `KSTACKTOP-KSTKSIZE`, and 256MB of physical memory at `KERNBASE`. Because of this, JOS can use at most 256MB of physical memory!



## Summary

Part 1 allocates space for data structures (`kern_pgdir`, `pages`, and `page_free_list`) and provides functions to manage physical memory, mainly by manipulating two data structures: `pages` and `page_free_list`.  

Part 2 provides functions to manage virtual memory, by manipulating physical memory (using functions in Part 1) and page table.

Part 3 creates several mappings above `UTOP` in `kern_pgdir`: maps `pages` into `UTOP` (so that user can read some of kernel's data structure), sets up kernel stack, and maps physical memory into kernel's address space. Then kernel switches from `entry_pgdir` to `kern_pgdir`. The virtual memory system is set up now. 



## Q & A

- After paging is enabled, how does the kernel access the page directory `kern_pgdir`?

    After the bootloader switched from 16-bit mode to 32-bit protected mode, all memory references are interpreted as virtual addresses and get translated by MMU. So the `kern_pgdir` variable represents a virtual address.

    At entering kernel code, paging uses the page table `entry_pgdir`, which maps virtual address range `[KERNBASE, KERNBASE+4MB)` at physical address `[0, 4MB)`. When `kern_pgdir` is allocated by `boot_alloc` in `mem_init`, it is in the range `[KERNBASE, KERNBASE+4MB)`, covered by `entry_pgdir`. Thus, when paging uses `entry_pgdir`, it can access `kern_pgdir` as the mapping exists in `entry_pgdir`. 

    When setting up virtual memory, kernel switches from `entry_pgdir` to `kern_pgdir`. Before switching, kernel must set up mapping in `kern_pgdir` properly. In `mem_init`, the virtual range `[KERNBASE, 2^32)` is mapped at physical range `[0, 2^32 - KERNBASE)` - all physical memory is mapped above `KERNBASE`. Because `kern_pgdir` maps `[KERNBASE, KERNBASE+4MB)` (memory storing kernel code and kernel data structures like `kern_pgdir` and `pages`) the same way as in `entry_pgdir`, After switching from `entry_pgdir` to `kern_pgdir`, the program still executes correctly and `kern_pgdir` can be accessed correctly.

    The switching is done in `mem_init` by `lcr3(PADDR(kern_pgdir));`. The physical address of `kern_pgdir` is stored into %cr3, to be used by the address translation hardware (MMU).



## Reference

https://github.com/yyin-dev/MIT6.828/blob/master/labNotes/lab2/lab2.md

https://pdos.csail.mit.edu/6.828/2018/labs/lab2/